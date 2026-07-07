---
name: pq-analysis
description: How to analyse parliamentary questions — browse for the documents, SPARQL for the aggregates, and the 30-second timeout traps that will otherwise cost you the query
---

# Analysing parliamentary questions

Two tools, two jobs:

- **`browse_documents`** on `written-questions` (or `oral-questions`) to list
  and read individual PQs. Rich filters: `subject`, `department` /
  `answeringDept`, `askingMember`, `tablingMember`, `answeringMember`,
  `session`, and `dateFrom`/`dateTo`. This is the default.
- **`sparql_query`** for the aggregates browse can't express — "top tablers",
  "how many by department", distributions over time.

**Coverage.** `written-questions` is indexed from 2020-01-01 (on
`date_tabled`). The triplestore holds ~432k `parl:WrittenParliamentaryQuestion`.
Members link out to MNIS via `see_also` URIs (`asking_member.see_also`,
`tabling_member.see_also`, …) — that's the join spine to MNIS for party, seat
and tenure (see the query-member-data-odata playbook).

## The SPARQL building blocks (verified)

- Class: `parl:WrittenParliamentaryQuestion`
- `parl:tablingMember` / `parl:tablingMemberPrinted` (the printed name literal)
- `parl:askingMember`, `parl:answeringMember`
- `parl:answeringDeptPrinted`, `parl:departmentPrinted`
- `dcterms:subject` (SIT-class SES term IRIs)
- `parl:dateTabled`, `parl:dateOfAnswer` (typed `xsd:dateTime`)

## The timeout traps (30s limit — each learned the hard way)

1. **Use typed date literals.** `FILTER(?d >= "2026-06-01T00:00:00"^^xsd:dateTime)`
   completes; the string form `FILTER(STR(?d) >= "2026-06-01")` forces a
   per-row cast over the whole class and times out.

2. **Bound the date window.** Scan cost scales with the window. A ~1-month
   window over the full PQ class completes for both department- *and*
   member-level grouping; an open-ended window risks the 30s cap. Add a
   `dateTabled` lower bound before you group.

3. **Watch cardinality, but don't fear member grouping.** Low-cardinality
   `GROUP BY` (department, ~20 groups) is cheap. Member-level grouping
   (thousands of tablers) is heavier but *does* complete inside a bounded
   window — it is the unbounded scan, not the group count, that kills queries.

4. **Confirm subject selectivity before scoping by it.** A subject term that
   isn't actually used on WPQs doesn't fail fast — it can make the planner scan
   the whole class and time out. In particular, **TPG-class terms are not
   subjects**: `dcterms:subject <…/terms/95736>` (the TPG "Renewable energy")
   returns 0, and layering a grouped date query on top timed out. Resolve the
   **SIT**-class subject and `COUNT` it first:
   `SELECT (COUNT(?q) AS ?n) WHERE { ?q a parl:WrittenParliamentaryQuestion ; dcterms:subject <iri> }`.
   If a subject-scoped member ranking still times out, fall back to
   `browse_documents(filters={subject:<id>, …}, dateFrom=…)` and tally
   `tablingMember` client-side, or answer the member question via MNIS.

## Worked example (verified, ran clean)

Top answering departments, questions tabled since 1 June 2026:

```sparql
PREFIX parl: <http://data.parliament.uk/schema/parl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?dept (COUNT(?q) AS ?n) WHERE {
  ?q a parl:WrittenParliamentaryQuestion ;
     parl:dateTabled ?d ;
     parl:answeringDeptPrinted ?dept .
  FILTER(?d >= "2026-06-01T00:00:00"^^xsd:dateTime)
}
GROUP BY ?dept ORDER BY DESC(?n) LIMIT 10
```

→ Department of Health and Social Care 1095, Department for Transport 838,
Ministry of Housing, Communities and Local Government 739, Ministry of Defence
727, Home Office 572, …

Swap `answeringDeptPrinted` for `tablingMemberPrinted` (same window) and it
still completes — top tablers came back McMurdock 320, Snowden 303, Wood 294, …

See scope-a-sparql-aggregation for the general IRI-confirmation discipline.

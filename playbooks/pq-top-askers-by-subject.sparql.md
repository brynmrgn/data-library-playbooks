---
name: pq-top-askers-by-subject.sparql
description: Verified worked SPARQL — top tabling members for a subject since a date; the companion query referenced by pq-analysis
---

# Top askers by subject — worked query

Companion to `pq-analysis`. A verified `sparql_query` that ranks the members
tabling the most written questions on a subject since a date. Copy it, swap the
subject IRI and the date literal.

## The query

```sparql
PREFIX parl: <http://data.parliament.uk/schema/parl#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?member (COUNT(?q) AS ?n) WHERE {
  ?q a parl:WrittenParliamentaryQuestion ;
     dcterms:subject <http://data.parliament.uk/terms/92809> ;   # Renewable energy (SIT)
     parl:dateTabled ?d ;
     parl:tablingMemberPrinted ?member .
  FILTER(?d >= "2026-01-01T00:00:00"^^xsd:dateTime)
}
GROUP BY ?member ORDER BY DESC(?n) LIMIT 10
```

## Verified output (run July 2026)

Subject 92809 carries 1,025 written questions in total; scoped to those tabled
since 1 January 2026, the top tablers are:

```
member,n
"McMurdock, James",17
"Bowie, Andrew",7
"Heylings, Pippa",6
"Ramsay, Adrian",5
"Stafford, Gregory",4
The Lord Bishop of Norwich,4
"MacDonald, Mr Angus",3
"Young, Claire",3
```

## Why it completes (and how to keep it that way)

- **Use the SIT-class subject IRI, not its TPG twin.** "Renewable energy" is
  both 92809 (SIT) and 95736 (TPG); only the SIT concept tags written
  questions. Filtering on 95736 returns 0 — and, worse, the non-selective
  triple makes the planner scan the whole ~430k class and time out. Resolve via
  `search_terms(id='ses')`, take the SIT id, and COUNT it first:
  `SELECT (COUNT(?q) AS ?n) WHERE { ?q a parl:WrittenParliamentaryQuestion ; dcterms:subject <iri> }`.
- **Typed date literal.** `"2026-01-01T00:00:00"^^xsd:dateTime`, never
  `FILTER(STR(?d) >= "2026-01-01")` — the string cast defeats the index and
  times out.
- **The subject filter is what makes the member GROUP BY safe.** A selective
  subject (here 1,025 questions) narrows the working set before the
  high-cardinality grouping, so this completes where an unscoped GROUP BY over
  members would not. See `scope-a-sparql-aggregation`.

For a name into party / seat / tenure, join out per `join-member-across-sources`.

---
name: scope-a-sparql-aggregation
description: When and how to reach for sparql_query for aggregation, ranking and cross-cutting counts the structured tools cannot express
---

# Scoping a SPARQL aggregation

`browse_documents` is the default for listing and filtering. Reach for
`sparql_query` only when the question is about **aggregation, ranking, or
cross-cutting counts** that the structured tools cannot express — for example:

- "top N tabling members for a subject"
- "how many X by Y"
- distributions over time

`sparql_query` is the specialist, not the first move. If the question is really
"show me the documents about X", stay with `browse_documents`.

## Confirm the IRIs before you query

A SPARQL query against the wrong class or predicate IRI returns an empty result
set, which is a failure, not a pass. Confirm the building blocks first:

1. **Classes** — confirm the class IRI with `sparql_list_classes`.
2. **Predicates** — confirm predicate IRIs with `sparql_list_predicates`.
3. **Subject terms** — resolve a subject label to its term IRI with
   `resolve_term` (do not guess the IRI, and do not pre-expand — pass the label
   or URI and let the thesaurus tools widen it).

## Then write the query

Keep the query tabular (`SELECT`) when the consumer wants counts or a ranking.
Group and order explicitly. If the result set comes back empty, treat it as a
signal to re-check the class/predicate/term IRIs above before assuming the data
is absent.

## Mind the 30-second timeout — it is shape-sensitive

The store enforces a 30s limit, and whether a query beats it depends less on
what you ask than on how you write it:

- **Type date filters.** `FILTER(?d >= "2026-06-01T00:00:00"^^xsd:dateTime)`
  completes; `FILTER(STR(?d) >= "2026-06-01")` casts every row and times out.
- **Bound the scan.** Cost scales with how much of a large class you touch. Add
  a date lower bound (or another selective pattern) before aggregating over a
  class with hundreds of thousands of instances.
- **Cardinality.** Low-cardinality `GROUP BY` (e.g. ~20 departments) is cheap;
  very-high-cardinality grouping is heavier and needs a bounded working set.
- **Selective patterns must actually be selective.** A subject term that isn't
  used on the target class doesn't short-circuit to empty — the planner may
  scan the whole class and time out. `COUNT` the scoping term against the class
  first. Note that **TPG-class SES terms are not document subjects** (subjects
  are SIT-class) — scoping `dcterms:subject` by a TPG twin matches nothing.

## Know where the store ends

The triplestore covers the **data-services graph only**. `sparql_list_classes`
is the authority on what's in scope: chamber, legislation, research and
government classes are present, but committee evidence, committee publications
and calendar events are **not** — so cross-cutting committee aggregations
("which organisations submitted evidence to the most inquiries") are not
expressible in SPARQL. Reach for the committee resources via `browse_documents`
instead.

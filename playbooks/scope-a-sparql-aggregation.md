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

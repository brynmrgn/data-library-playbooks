---
name: scope-a-sparql-aggregation
description: >-
  Use when a question needs AGGREGATION, RANKING, or CROSS-CUTTING COUNTS the
  structured browse/filter tools cannot express — "how many X by Y", "top N…",
  distributions over time, "which department/member/committee most…" — even if
  they don't mention SPARQL. Covers when to reach for sparql_query and how to
  scope it without timing out.
id: scope-a-sparql-aggregation
title: Scoping a SPARQL aggregation
task_types:
  - sparql_aggregation
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the SPARQL tools available. Reach for this when the question is a
count, ranking, or distribution that browse_documents cannot express.
<!-- /skill-only -->

# Scoping a SPARQL aggregation

`browse_documents` is the default for listing and filtering. Reach for
`sparql_query` only when the question is about **aggregation, ranking, or
cross-cutting counts** that the structured tools cannot express — for example:

- "top N tabling members for a subject"
- "how many X by Y"
- distributions over time

`sparql_query` is the specialist, not the first move. If the question is really
"show me the documents about X", stay with `browse_documents`.

## Know where the store ends

The triplestore covers the **data-services graph only** — legislation, written
and oral questions, proceedings, briefings, deposited papers, statements.
There are **no classes for committee evidence, committee publications, or
calendar events**, so committee-domain aggregation ("which organisations
submitted evidence to the most inquiries") is inexpressible in SPARQL, full
stop. Answer those by enumeration on the browse side (see
`exhaustive-coverage-search`) or not at all — don't burn time debugging a
query the store cannot answer. `sparql_list_classes` is the ground truth for
what's in scope.

## Confirm the IRIs before you query

A SPARQL query against the wrong class or predicate IRI returns an empty result
set, which is a failure, not a pass. Confirm the building blocks first:

1. **Classes** — confirm the class IRI with `sparql_list_classes`.
2. **Predicates** — confirm predicate IRIs with `sparql_list_predicates`.
3. **Subject terms** — resolve a subject label to its term IRI with
   `resolve_term` (do not guess the IRI, and do not pre-expand — pass the label
   or URI and let the thesaurus tools widen it).

## Then write the query — shape determines whether it survives the timeout

Queries are killed at **30 seconds**, and whether you hit the limit depends on
query *shape*, not result size. Two shapes account for every timeout observed
against this store (verified July 2026, over ~430k written questions):

- **Date filters must use typed literals.**
  `FILTER(?d >= "2026-01-01T00:00:00"^^xsd:dateTime)` completes;
  the same filter written `FILTER(STR(?d) >= "2026-01-01")` times out —
  `STR()` casts defeat the index. Never string-compare dates.
- **Scope high-cardinality groupings.** `GROUP BY` over members times out
  unscoped but completes when the pattern is first narrowed by date, session,
  or subject; the identical query grouped by *department* (low cardinality)
  is fine unscoped. Rule of thumb: hundreds-of-groups aggregations need a
  scoping filter inside the pattern; tens-of-groups don't.

Keep the query tabular (`SELECT`) when the consumer wants counts or a ranking.
Group and order explicitly.

## Reading failure correctly

- **Empty result** → re-check the class/predicate/term IRIs above, then check
  the domain boundary, before assuming the data is absent.
- **Timeout** → reshape first: typed literals for dates, scope the grouping,
  tighten the pattern. But a timeout can also be **transient load**, not a bad
  query — once the IRIs are confirmed and a cheap selective COUNT runs fast, one
  retry is worth it before assuming the shape is wrong (verified: a known-good
  ranking timed out once and ran clean on retry).

## Widening a subject without blowing up

To count/rank across a subject *and its narrower concepts*, don't drop an inline
`skos:narrower*` into the big join — the property path across a large class is
heavy and tips over under load. Two-step instead: resolve the subtree cheaply
(`expand_term` narrower, or a standalone `<term> skos:narrower* ?t` query — the
thesaurus lives in the same store), then match a bounded `VALUES` set of those
IRIs with `COUNT(DISTINCT ?q)`. The planner gets concrete, selective subjects
and stays inside the limit. Worked example in `pq-top-askers-by-subject.sparql`.

For the PQ corpus specifically — the most common aggregation target — worked
query shapes are in `pq-analysis`.

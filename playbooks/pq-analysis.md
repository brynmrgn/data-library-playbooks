---
name: pq-analysis
description: Researching written parliamentary questions — browse for the question texts, SPARQL for the aggregates, and the query shapes that avoid timeouts
---

# Analysing parliamentary questions

PQ research splits cleanly in two: **reading questions** is a
`browse_documents` job; **counting or ranking them** is a `sparql_query` job.
Picking the wrong half wastes time in both directions — paging through browse
results to count them by hand, or writing SPARQL to fetch ten question texts.

## Reading questions: browse

`browse_documents` on `written-questions` with SES filters, which compose as
AND:

- `subject` = what the question is about (SIT class terms — resolve via
  `search_terms` against the `ses` source; the old per-facet source ids like
  `subject` now return empty *silently*);
- `department` = the answering department;
- `dateFrom`/`dateTo` for the window.

Subject + department + date is the workhorse combination ("questions to DHSC
on hospital waiting lists since January"). Use the facets block on a first
broad call to pick the exact term ids, per `exhaustive-coverage-search`.
`get_document` returns question and answer text.

## Counting and ranking: SPARQL

"Top tablers on X", "volume by department per month", "how many since the
election" — aggregation the structured tools cannot express. Read
`scope-a-sparql-aggregation` first for IRI confirmation; the corpus here is
large (~430k written questions), which makes the query-shape traps live:

- **Typed date literals only.** `FILTER(?d >= "2026-01-01T00:00:00"^^xsd:dateTime)`
  completes; the same filter via `STR(?d)` times out at the 30s limit.
- **Scope high-cardinality groupings.** GROUP BY member over the whole corpus
  times out; the identical query date- or session-scoped completes. GROUP BY
  department is low-cardinality and safe unscoped.
- A worked, verified query (top tablers on a subject since a date) is in the
  project file `pq-top-askers-by-subject.sparql.md`.

PQs are in the data-services graph, so they are on the right side of the
triplestore's domain boundary (unlike committee material — see
`scope-a-sparql-aggregation`).

## Member-level analysis: join out

A ranking gives you member IRIs/names; anything *about* those members — party,
seat, tenure, whether they sit on the relevant committee — is a join to MNIS
per `query-member-data-odata` and `join-member-across-sources`. "Are the top
askers on renewable energy mostly opposition members?" is a two-source
question: SPARQL for the ranking, MNIS for the party column.

## Why

PQ questions from users are usually aggregate in disguise ("who's been asking
about X" = a ranking), and the naive route — browse and count — is slow,
paginated, and unciteable at corpus scale. The split above keeps each tool on
the job it is actually good at.

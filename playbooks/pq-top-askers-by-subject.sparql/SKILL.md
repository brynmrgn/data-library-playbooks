---
name: pq-top-askers-by-subject.sparql
description: >-
  Use when ranking the MEMBERS who table the most written questions on a
  subject since a date — the verified worked SPARQL query (subject widened to
  its narrower concepts) that pq-analysis references for its top-askers step.
  Reach for it when you need the exact query, not just the approach.
id: pq-top-askers-by-subject.sparql
title: Top askers by subject — worked SPARQL query
task_types:
  - pq_analysis
  - sparql_aggregation
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the SPARQL tools available. This is the verified query behind pq-
analysis's top-askers ranking.
<!-- /skill-only -->

# Top askers by subject — worked query

Companion to `pq-analysis`. Ranks the members tabling the most written questions
on a subject since a date.

A bare `dcterms:subject <iri>` match **undercounts** — it misses questions tagged
only with a *narrower* concept (Wind power, Solar power, Photovoltaics under
"Renewable energy"). For "Renewable energy" since 1 Jan 2026 that's 1,023
questions exact vs **3,289** across the subtree. So widen to the narrower terms —
in a way Oxigraph can handle under load.

## Step 1 — resolve the subject subtree (cheap, isolated)

Widen the SIT subject to itself + its narrower concepts. Use `expand_term`
(direction narrower), or this one-off query — it's a sub-second traversal on its
own:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT ?t WHERE { <http://data.parliament.uk/terms/92809> skos:narrower* ?t }
```

For "Renewable energy" (92809) that's ~20 terms: Wind / Solar / Tidal / Wave /
Hydroelectric / Geothermal power, Photovoltaics, Biofuels, Combined heat and
power, Renewables obligation, and so on.

## Step 2 — rank against that fixed set (bounded, selective)

Paste those IRIs into a `VALUES` block. **This is the robust shape:** the planner
gets concrete, selective subjects rather than evaluating a property path across
the ~430k-question join, so it stays well inside the 30s limit.

```sparql
PREFIX parl: <http://data.parliament.uk/schema/parl#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT ?member (COUNT(DISTINCT ?q) AS ?n) WHERE {
  VALUES ?subject {
    <http://data.parliament.uk/terms/92809> <http://data.parliament.uk/terms/11962>
    <http://data.parliament.uk/terms/90341> <http://data.parliament.uk/terms/90585>
    <http://data.parliament.uk/terms/93503> <http://data.parliament.uk/terms/93470>
    <http://data.parliament.uk/terms/12460> <http://data.parliament.uk/terms/525805>
    <http://data.parliament.uk/terms/508369> <http://data.parliament.uk/terms/518406>
    <http://data.parliament.uk/terms/93465> <http://data.parliament.uk/terms/472162>
    <http://data.parliament.uk/terms/488403> <http://data.parliament.uk/terms/12139>
    <http://data.parliament.uk/terms/91425> <http://data.parliament.uk/terms/12694>
    <http://data.parliament.uk/terms/93267> <http://data.parliament.uk/terms/91596>
    <http://data.parliament.uk/terms/93475> <http://data.parliament.uk/terms/93067>
  }
  ?q a parl:WrittenParliamentaryQuestion ;
     dcterms:subject ?subject ;
     parl:dateTabled ?d ;
     parl:tablingMemberPrinted ?member .
  FILTER(?d >= "2026-01-01T00:00:00"^^xsd:dateTime)
}
GROUP BY ?member ORDER BY DESC(?n) LIMIT 10
```

`COUNT(DISTINCT ?q)` because a question tagged with both the parent and a child
would otherwise count twice.

## Verified output (run July 2026, subtree since 1 Jan 2026)

```
member,n
"McMurdock, James",54
"Bowie, Andrew",15
"Heylings, Pippa",11
"Hollinrake, Kevin",10
"Morello, Edward",9
"Mullan, Dr Kieran",8
"Stafford, Gregory",6
"Hobhouse, Wera",6
"Niblett, Samantha",5
"Maguire, Ben",5
```

Exact-subject-only, for comparison: McMurdock 17, Bowie 7, Heylings 6 — the
narrower terms roughly treble the counts and surface members (Hollinrake,
Morello, Mullan) who tag under wind/solar, not the parent concept.

## Keeping it inside the timeout

- **SIT subject IRI, not the TPG twin.** "Renewable energy" is 92809 (SIT) and
  95736 (TPG); only the SIT concept tags questions. Filtering the TPG returns 0
  and can make the planner scan the whole class and time out. COUNT the subject
  first to confirm it's selective.
- **Typed date literal** — `"2026-01-01T00:00:00"^^xsd:dateTime`, never
  `FILTER(STR(?d) >= "2026-01-01")`; the string cast defeats the index.
- **Prefer the two-step VALUES to an inline `skos:narrower*`** in the ranking.
  The one-query form (`<subject> skos:narrower* ?subject` inside the join) *does*
  work, but it's heavier and likelier to tip over under triplestore load; the
  bounded VALUES set is the reliable shape.
- **A timeout can be transient load, not a bad query.** If a known-good query
  times out, confirm the IRI, run the cheap selective COUNT, then **retry once**
  before reshaping — verified: this ranking timed out once and ran clean on the
  retry.

For a name → party / seat / tenure, join out per `join-member-across-sources`.

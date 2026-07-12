---
name: join-member-across-sources
description: >-
  Use when a member-shaped answer needs data from MORE than one source joined
  — enriching data-library results (what a member DID) with MNIS (what they
  ARE: seat, party, dates) or Wikidata (external ids) — e.g. "the party of
  everyone who asked about X", "link these members to Wikidata". The seeAlso
  to Member_Id to Wikidata P10428 join spine.
id: join-member-across-sources
title: Joining member data across sources
task_types:
  - member_lookup
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the Parliamentary Data Library and member tools available. Reach
for this when a member question needs the data-library / MNIS / Wikidata
join.
<!-- /skill-only -->

# Joining member data across sources

No single source answers a member-shaped question end to end. The data library
knows what a member *did* (questions tabled, contributions, evidence given);
MNIS knows what a member *is* (seat, party, house, membership dates); Wikidata
knows what Parliament deliberately doesn't publish (date of birth, education,
prior career). The join spine that connects them is the **MNIS member id**.

## The spine

```
data library result ──seeAlso──▶ MNIS Member_Id ──P10428──▶ Wikidata item
```

- **Data library → MNIS.** Member references on data-library results carry the
  MNIS id in their `seeAlso` fields. Extract it rather than matching on name —
  names collide and restyle (honorifics, peerage titles, marriage).
- **MNIS.** For the common hop — party, seat, house, current status, a name→id
  lookup, or a member's registered financial interests — use the in-server
  tools **`mnis_get_member`** (by id), **`mnis_search_members`** (by name), and
  **`mnis_get_member_registered_interests`**. They return clean JSON and take
  the same `Member_Id` the data library carries in `seeAlso`. Only drop to the
  raw MNIS OData service when you need what those don't cover — the legacy
  cross-reference ids (`Dods_Id`, `Pims_Id`, `Clerks_Id`) or swearing-in order
  for precedence; its mechanics and traps are in `query-member-data-odata`.
- **MNIS → Wikidata.** Wikidata property **P10428** ("UK Parliament member ID")
  holds the MNIS id, so the SPARQL join is exact:

  ```sparql
  SELECT ?item ?itemLabel ?dob WHERE {
    ?item wdt:P10428 "185" .                # MNIS Member_Id as a string
    OPTIONAL { ?item wdt:P569 ?dob . }
    SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
  }
  ```

  against `https://query.wikidata.org/sparql`. Note the id is a **string
  literal** in Wikidata, not a number.

## When each hop earns its cost

- Stop at the data library if the question is about documents.
- One hop to MNIS (the `mnis_*` tools) for seat/party/tenure columns or
  registered interests — the usual case, e.g. adding a party column to a PQ
  ranking from `pq-analysis`.
- Two hops to Wikidata **only** for biography: MNIS's `DateOfBirth`,
  `TownOfBirth` and `BirthCountry_Id` fields exist but are null for every
  member, by policy. Don't page MNIS hoping otherwise; go to Wikidata.

## Traps

- **Wikidata is community-maintained.** Fine for DOB/education on current
  members; spot-check anything that will be quoted, and cite it as Wikidata,
  never as Parliament.
- **P10428 coverage is very good for recent members, thinner historically.**
  A missing item means "no link", not "no such member" — fall back to a
  labelled name search, then verify via the constituency.
- **Do the join per-id, not per-name, in both directions.** The only safe
  name-free path is seeAlso → Member_Id → P10428.

## Why

Each hop is individually documented nowhere and rediscovered every time a
question crosses a boundary ("which of the top askers are ex-ministers?",
"how old are the new intake?"). Fixing the spine in one place makes
cross-source member questions routine instead of bespoke.

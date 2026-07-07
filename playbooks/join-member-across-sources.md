---
name: join-member-across-sources
description: The member join spine — data library seeAlso to MNIS Member_Id to Wikidata P10428 — for enriching parliamentary results with member data Parliament does not hold
---

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
- **MNIS.** `Members(<id>)` keys the whole platform — seat, party, dates, and
  the legacy cross-reference ids (`Dods_Id`, `Pims_Id`, `Clerks_Id`) that the
  modern Members API drops. Query mechanics, format traps and worked examples
  are in `query-member-data-odata` — not repeated here.
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
- One hop to MNIS for seat/party/tenure columns — the usual case, e.g. adding
  a party column to a PQ ranking from `pq-analysis`.
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

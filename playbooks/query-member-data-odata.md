---
name: query-member-data-odata
description: How to query the MNIS OData v3 service for UK member data — finding members, their seat, party, and membership dates, ranking by tenure or precedence, and the format traps that will otherwise cost you time
---

# Querying member data with OData

The MNIS OData service is the authoritative source for UK members: who is or was
an MP or peer, their seat, party, and membership dates. Service root:
`https://data.parliament.uk/membersdataplatform/open/OData.svc/`. It is a
conformant OData **v3** service; the entity set you want is `Members`, keyed on
`Member_Id`. The full schema, covering all 76 entity sets and their fields, is at
`.../OData.svc/$metadata` (EDMX, `MaxDataServiceVersion=3.0`) — reach for it when
you need a field this playbook doesn't cover.

## Three traps to know before you start

1. **It speaks XML/Atom only.** Requesting JSON (`Accept: application/json` or
   `$format=json`) returns *Unsupported media type*. Parse the Atom XML.
2. **Responses carry a UTF-8 BOM.** Strip it before parsing, or the parser
   chokes.
3. **`$filter ... ne null` is silently ignored.** It returns an unfiltered page
   and a misleading `$inlinecount`. Never treat a `ne null` count as evidence a
   field is populated — inspect the rows for the `m:null` attribute.

## Working pattern

Filter `Members` with `$filter` and select the fields you need. Standard query
options work in native form: `$filter` (with `eq`, `and`/`or`, `substringof`),
`$select`, `$orderby`, `$top`/`$skip`, `$inlinecount`. Pagination is
server-driven — follow the `<link rel="next">` cursor for a full extraction
rather than relying on `$skip`.

- Current Commons members: `$filter=House_Id eq 1 and CurrentStatusActive eq true`.
- A named member: `$filter=Surname eq 'Cooper'`, or `substringof('Cooper',NameDisplayAs)`.
- A single record by key: `Members(420)`.

Useful fields: `Member_Id`, `NameDisplayAs`, `Party`, `House` / `House_Id`
(1 = Commons), `MembershipFrom` (current seat), `CurrentStatusActive`,
`StartDate`, `EndDate`. The cross-reference ids `Dods_Id`, `Pims_Id` and
`Clerks_Id` are here too and are dropped by the modern Members API.

## Two field-level gotchas

- **Use `StartDate`, not `CurrentStatusDate`, for when a membership began.**
  `CurrentStatusDate` can lag — a by-election winner showed a fresh `StartDate`
  but a years-old `CurrentStatusDate`. Sort on `StartDate` for tenure. But it
  can't fully rank "longest-serving": members elected at the same general
  election tie on it. Break the tie via the oaths table (below).
- **`DateOfBirth`, `TownOfBirth` and `BirthCountry_Id` are null for every
  member**, by policy. The fields exist but are never populated. For biographical
  data, join to Wikidata on `Member_Id` (the `P10428` property).

## Breaking tenure ties: the oaths table

Members elected at the same general election share a `StartDate`, so a
`StartDate` sort can't order them. Precedence — the basis for "Father/Mother of
the House" — comes from swearing-in order, held in `MemberConstituencyOaths`.

It joins on **`MemberConstituency_Id`, not `Member_Id`**: go `Members` →
`MemberConstituencies` (pick the membership for the relevant election) →
`MemberConstituencyOaths`. The field you want is `SwearInOrder` — lower means
sworn in earlier. `OathDate` is null throughout, so don't rely on it.

A flatter `MemberOaths` set is keyed directly on `Member_Id` and has an
`OathDate` but a mostly-null `SwearInOrder` — prefer `MemberConstituencyOaths`
for precedence.

Worked example: the three current MPs first elected on 1983-06-09 tie on
`StartDate`; their 1983 `SwearInOrder` values (Leigh 9, Corbyn 13, Gale 21) give
the correct order.

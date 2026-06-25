---
name: query-member-data-odata
description: How to query the MNIS OData v3 service for UK member data — finding members, their seat, party, and membership dates — and the format traps that will otherwise cost you time
---

# Querying member data with OData

The MNIS OData service is the authoritative source for UK members: who is or was
an MP or peer, their seat, party, and membership dates. Service root:
`https://data.parliament.uk/membersdataplatform/open/OData.svc/`. It is a
conformant OData **v3** service; the entity set you want is `Members`, keyed on
`Member_Id`.

## Three traps to know before you start

1. **It speaks XML/Atom only.** Requesting JSON (`Accept: application/json` or
   `$format=json`) returns *Unsupported media type*. Parse the Atom XML.
2. **Responses carry a UTF-8 BOM.** Strip it before parsing or the parser
   chokes.
3. **`$filter ... ne null` is silently ignored.** It returns an unfiltered page
   and a misleading `$inlinecount`. Never treat a `ne null` count as evidence a
   field is populated — inspect the actual rows for the `m:null` attribute.

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
  but a years-old `CurrentStatusDate`. For tenure or "longest-serving" ranking,
  sort on `StartDate`.
- **`DateOfBirth`, `TownOfBirth` and `BirthCountry_Id` are null for every
  member**, by policy. The fields exist but are never populated. For biographical
  data, join to Wikidata on `Member_Id` (the `P10428` property) rather than
  expecting it here.

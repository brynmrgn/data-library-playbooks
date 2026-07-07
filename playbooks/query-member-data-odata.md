---
name: query-member-data-odata
description: How to query the MNIS OData v3 service for UK member data — finding members, their seat, party, and membership dates, ranking by tenure or precedence, and the format traps that will otherwise cost you time
---

# Querying member data with OData

**First, check whether you need this at all.** For the common lookups — a
member's party, seat, house, current status, a name→id lookup, or their
registered financial interests — use the in-server tools `mnis_search_members`,
`mnis_get_member` and `mnis_get_member_registered_interests`. They wrap the
modern Members API, return clean JSON, and take the same `Member_Id` the data
library carries in `seeAlso` — none of the XML/BOM/`ne null` handling below.
Reach for the OData service in this playbook only for what those don't cover:
the legacy cross-reference ids (`Dods_Id`/`Pims_Id`/`Clerks_Id`) and swearing-in
order for precedence ("Father/Mother of the House").

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

## Common queries

Each links to live results (Atom XML). Visible labels are unencoded for
readability; the link targets are encoded.

- **Current MPs** (Commons, sitting):
  [`Members?$filter=House_Id eq 1 and CurrentStatusActive eq true`](https://data.parliament.uk/membersdataplatform/open/OData.svc/Members?$filter=House_Id%20eq%201%20and%20CurrentStatusActive%20eq%20true)

- **MPs who are ministers** — government posts join to `Members` via `Member_Id`,
  and `EndDate eq null` selects posts still held. Filter on the expanded
  `Member/House_Id` to keep Commons only (this nav-property filter *is* honoured —
  see the null note below):
  [`MemberGovernmentPosts?$filter=EndDate eq null and Member/House_Id eq 1&$expand=Member`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberGovernmentPosts?$filter=EndDate%20eq%20null%20and%20Member/House_Id%20eq%201&$expand=Member)

- **Peers who are ministers** — same, with `House_Id eq 2`:
  [`MemberGovernmentPosts?$filter=EndDate eq null and Member/House_Id eq 2&$expand=Member`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberGovernmentPosts?$filter=EndDate%20eq%20null%20and%20Member/House_Id%20eq%202&$expand=Member)

- **Membership of a House on a given date** — a membership was live on date *D* if
  it started on or before *D* and either has no end or ended on or after *D*. Use
  `MemberConstituencies` for the Commons (replace the date as needed):
  [`MemberConstituencies?$filter=StartDate le datetime'2000-01-01' and (EndDate eq null or EndDate ge datetime'2000-01-01')`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberConstituencies?$filter=StartDate%20le%20datetime'2000-01-01'%20and%20(EndDate%20eq%20null%20or%20EndDate%20ge%20datetime'2000-01-01'))

  Date literals take the form `datetime'YYYY-MM-DD'`. For the Lords, apply the
  same start/end logic to `MemberHouseMemberships`, which carries its own
  `House_Id` (so scope with `House_Id eq 2` directly — no expand needed):
  [`MemberHouseMemberships?$filter=House_Id eq 2 and StartDate le datetime'2010-01-01' and (EndDate eq null or EndDate ge datetime'2010-01-01')`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberHouseMemberships?$filter=House_Id%20eq%202%20and%20StartDate%20le%20datetime'2010-01-01'%20and%20(EndDate%20eq%20null%20or%20EndDate%20ge%20datetime'2010-01-01'))

  (`MemberHouseMemberships` covers both Houses, so `House_Id eq 1` gives the
  equivalent Commons view if you prefer one entity for both.)

  **Caveat on historic Lords dates.** MNIS's Lords house-membership records
  effectively begin at the House of Lords Act 1999, which removed most hereditary
  peers (the latest start dates in the pre-2000 data cluster around Nov 1999). A
  membership-on-a-date query for a point *before* the 1999 reform will badly
  undercount — it returns only the post-reform House (life peers plus the 92
  retained hereditaries), not the larger pre-reform chamber. Treat the query as
  reliable from 2000 onwards; for earlier snapshots, the data isn't there.

- **State of the parties in a House** — there's no standings entity and OData v3
  can't aggregate, so fetch the current members of the House and tally the
  `Party` field client-side. The browsable source query (Commons):
  [`Members?$filter=House_Id eq 1 and CurrentStatusActive eq true&$select=Member_Id,Party`](https://data.parliament.uk/membersdataplatform/open/OData.svc/Members?$filter=House_Id%20eq%201%20and%20CurrentStatusActive%20eq%20true&$select=Member_Id,Party)

  Use `House_Id eq 2` for the Lords. Two cautions when reading the result: the
  Speaker appears under their own "party" label (the Speaker sits apart from
  party), and the tally counts whole membership, not voting strength — non-sitting
  members (e.g. Sinn Féin) and the non-voting Speaker and deputies mean an
  effective-majority figure is a further calculation, not the raw breakdown.

A note on null filtering: the `ne null` trap above is specific to `ne null`.
`EndDate eq null` *does* filter correctly (it's how "still in post" / "still
sitting" is expressed), and filters on expanded navigation properties such as
`Member/House_Id` are honoured. Only `ne null` is silently ignored.

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

### Worked example: the 1983 cohort

The three current MPs first elected on 1983-06-09 tie on `StartDate`. No single
query ranks them — the steps below each fetch one piece of evidence, and the
final order is assembled by comparing the three `SwearInOrder` values. Each query
is shown readably, with a link to the live result (Atom XML).

1. Current Commons members, sorted on `StartDate`, surfaces the tie — Member_Ids
   87 (Gale), 185 (Corbyn) and 345 (Leigh) all show 1983-06-09:

   [`Members?$filter=House_Id eq 1 and CurrentStatusActive eq true&$select=Member_Id,NameDisplayAs,StartDate&$orderby=StartDate`](https://data.parliament.uk/membersdataplatform/open/OData.svc/Members?$filter=House_Id%20eq%201%20and%20CurrentStatusActive%20eq%20true&$select=Member_Id,NameDisplayAs,StartDate&$orderby=StartDate)

2. For each member, find the 1983 constituency membership and note its
   `MemberConstituency_Id` (here, Leigh → 1435):

   [`MemberConstituencies?$filter=Member_Id eq 345`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberConstituencies?$filter=Member_Id%20eq%20345)

3. Read that membership's oath to get `SwearInOrder`:

   [`MemberConstituencyOaths?$filter=MemberConstituency_Id eq 1435`](https://data.parliament.uk/membersdataplatform/open/OData.svc/MemberConstituencyOaths?$filter=MemberConstituency_Id%20eq%201435)

Repeating steps 2–3 for all three gives Leigh 9, Corbyn 13, Gale 21. Ordering
those `SwearInOrder` values ascending is done client-side, not by the service,
and yields the precedence Leigh → Corbyn → Gale.

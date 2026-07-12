---
name: whats-on-this-week
description: >-
  Use for FORTHCOMING parliamentary business from the forward calendar —
  "what's on in the Lords this week?", "what's coming up on X?", "is the House
  sitting Thursday?", debates and committee meetings set down in the next days
  or weeks — even if they don't say "calendar". Covers what the calendar can
  and cannot tell you (no recess or sitting-status signal).
id: whats-on-this-week
title: What's on this week
task_types:
  - forward_business
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the Parliamentary Data Library tools available. Reach for this for
forthcoming business from the forward calendar — what is set down, not what
has happened.
<!-- /skill-only -->

# What's on this week

`calendar-events` mirrors Parliament's What's On: chamber sittings, debates,
and general/grand committee meetings, forward-only, roughly six months out.
It answers "what is set down" — it does not answer "is the House sitting", and
two of its advertised behaviours don't work.

## Working pattern

1. **Browse by date; never call the show endpoint.** `browse_documents` on
   `calendar-events` filtered to the date range, sorted by date. `get_document`
   on this type 404s for every id, including ids taken straight from its own
   listing and genuinely live events (verified with 52949, 50541, 55449, 55467
   across May–July 2026). Everything you can know about an event is in the
   browse row.

   **Trap: the date filters go soft when `q=` is present.** With a keyword
   query alongside `startDateFrom`/`startDateTo`, past events leak back in — the
   date bounds behave as ranking signals, not hard constraints (verified 8 July
   2026). So for "what's scheduled from today", filter by date **without** a
   `q=`, and if you must keyword-search, discard out-of-range rows client-side
   rather than trusting the bound.

2. **Use standard fields, not `minimal`.** Under `fields='minimal'`,
   calendar-events rows can come back as nothing but `resource_type` + `date` —
   no title, no uri, nothing citeable.

3. **Scope by House/location facets** if the question is chamber-specific, and
   remember coverage is chamber + general/grand committee business. Select
   committee sessions live under committee events, not here.

## What an empty day does not mean

The calendar carries **no recess or sitting-status signal**. A nil return for
a date is ambiguous between "in recess" and "business not yet tabled" — do not
assert recess from an empty day. If sitting status matters, say the calendar
shows no business and flag the ambiguity, or confirm recess dates from an
announcement (the Leader of the House's business statement in `proceedings`).

## What a populated day does not mean

Set-downs can be **ghosts**. When a recess is announced after business was set
down (the classic case: PMB second readings on a sitting Friday that the
recess then takes), What's On does not clear or cancel the entries — they
persist with no `cancelledDate`, and the MCP mirrors source faithfully
(verified 29 May 2026 against whatson.parliament.uk directly). So:

- absence of `cancelledDate` ≠ confirmation the event will happen;
- a stale-looking entry is an upstream fidelity question, not MCP staleness —
  don't report it as an index bug.

Report future business as "set down / scheduled as of today", and for PMB
Fridays specifically, see the cross-model trap in `track-a-bill`.

## Why

Calendar questions get answered for people planning around them, so the failure
mode is over-assertion: "Parliament is in recess" from an empty day, or "the
bill will be debated Friday" from a ghost row. The calendar is a statement of
tabled intentions, and the answer should be worded as one.

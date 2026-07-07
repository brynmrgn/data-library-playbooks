---
name: whats-on-this-week
description: How to answer "what's happening in Parliament" from the calendar — its forward-only window, the missing recess signal, and why some calendar rows can't be cited
---

# What's on this week

`calendar-events` is the What's On feed: chamber business, Westminster Hall,
General Committees and Grand Committees. Filter it with `startDateFrom` /
`startDateTo` (plus `house`, `eventType`, `category`, `committeeId`,
`memberId`, `searchTerm`) and sort by `startDate` (the default, ascending).

## Four things the calendar will not tell you

1. **It is forward-only** (a rolling window, roughly six months out). There is
   no history here — you cannot ask what happened last week. For past chamber
   business, that's Hansard, not this resource.

2. **There is no recess signal.** An empty day or week does **not** mean
   recess. It could equally be a day with no scheduled business, or a date past
   the end of the window. The calendar cannot distinguish these — never report
   "Parliament is in recess" purely from an absence of events.

3. **Select and Joint Committee events are not here.** They are deliberately
   excluded (to avoid dual-modelling) and surfaced via `committee-events`
   instead. "What is the X Committee doing this week" needs `committee-events`;
   this resource only covers General/Grand Committees and the chambers.

4. **Future entries are provisional.** Business scheduled ahead can be pulled;
   a cancelled event carries a `cancelledDate`. Treat a forward entry as
   planned, not guaranteed. (This is upstream fidelity — the real order paper
   firms up closer to the day — not MCP staleness.)

## Read the list payload, don't chase get_document

The browse payload under `fields='standard'` is complete: each event carries
`description`, `startDate`/`endDate`, `location`, `house`, `committee`,
`members`, `eventActivities` (with `attendees`), and `billId`/`billName` for
bill business. Summarise straight from that — there is no need to call
`get_document` per event, and the per-event show endpoint is unreliable.

**Do not use `fields='minimal'` on the calendar.** Many calendar rows come back
under minimal as nothing but `resource_type` + `date` — no title, no uri, no
identifier, nothing citeable (verified). Use `fields='standard'` and cite from
`description` + `startDate`.

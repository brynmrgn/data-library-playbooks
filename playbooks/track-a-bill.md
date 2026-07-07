---
name: track-a-bill
description: How to trace a bill's progress — where stage data actually lives, why bill-stages is a dead end, and the PMB calendar traps
---

# Tracking a bill

The task everyone assumes is one call and isn't. Stage data is split across
three places — the bill record, proceedings, and the forward calendar — and the
resource type that *sounds* right is broken.

## Working pattern

1. **Find the bill.** `browse_documents` on `bills` with `q=` — bills are a
   catalogue-only type, so `q=` searches title/summary metadata and works well
   (`q=flood` returns the flood bills). `search_content` correctly returns 0
   for bills; that zero means "not semantically indexed", not "no such bill".

2. **Read stages from the bill record, never from `bill-stages`.**
   `browse_documents` on `bill-stages` with `filters={billId: <n>}` returns
   total 0 for *every* billId (verified against 3311, Energy Act 2023, and
   four others across May–July 2026), even though the same data is populated
   on the `bills` resource. The full stage list and `currentStage.stageSittings`
   on the bill record are the reliable source.

3. **Stage *debates* live in proceedings, not on the bill.** For what was said
   at second reading, switch to `proceedings` with a phrase-quoted `q=`
   (e.g. `q="Renters' Rights Bill"`) and sort by date. The bill record tells
   you *where* the bill is; Hansard tells you *what happened*.

4. **Forward look: the calendar.** Upcoming stages appear in `calendar-events`
   (browse by date — see `whats-on-this-week` for that type's own traps,
   including the broken show endpoint).

5. **Once enacted, leave this MCP.** Chain to **lex-uk-law** for the Act's
   text, commencement, and subsequent amendment history. `command-papers` and
   `statutory-instruments` cover the delegated-legislation tail on this side.

## The PMB Friday trap

What's On and the Bills API model a Private Member's Bill sitting Friday
differently, and each looks like it refutes the other:

- A PMB "set down" for a sitting Friday appears in `calendar-events`, but its
  second-reading `stageSittings` on the bill record **stays empty until the
  sitting is confirmed or occurs**. Empty `stageSittings` does NOT mean "not
  scheduled" — do not use the bill record to deny a calendar entry.
- Conversely, a calendar set-down persisting after a recess is announced is an
  upstream What's On fidelity issue, not proof the sitting will happen
  (verified 29 May 2026: pre-Whitsun PMB set-downs persisted with no
  `cancelledDate`, mirrored faithfully from What's On itself).

State PMB scheduling as "set down for" a date, never "will be debated on".

## Why

Bill-stage questions fail in a distinctive way: every individual call succeeds,
so nothing looks wrong, but the answer assembled from the wrong resource is
confidently incomplete — a briefing that misses a scheduled second reading, or
denies one that is set down. Knowing which of the three stores answers which
sub-question is the whole task.

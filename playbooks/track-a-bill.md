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

2. **Read the sitting schedule from a `browse_documents` on `bills`, not from
   `bill-stages` and not from `get_document`.** `bill-stages` returns total 0
   for *every* billId (verified against 3311 and others). And mind the
   projection drift: `currentStage.stageSittings` — the dated sittings for the
   current stage — is present on a **`browse_documents` on `bills`** result
   (standard fields) but is **dropped by `get_document`**, whose `currentStage`
   carries only the stage label (verified 8 July 2026, billId 4124). So browse
   the bill for the schedule; use `get_document` only for the fuller record.
   For the full historic stage list beyond the current one, the bill record on
   this side is thin — fall back to the Bills API (`/Bills/<id>/Stages`).

3. **Stage *debates* live in proceedings, not on the bill.** For what was said
   at second reading, switch to `proceedings` with a phrase-quoted `q=`
   (e.g. `q="Renters' Rights Bill"`) and sort by date. The bill record tells
   you *where* the bill is; Hansard tells you *what happened*.

4. **Forward look: the calendar.** Upcoming stages appear in `calendar-events`,
   which carry `billId`/`billName` in their payloads — but the **`billId`
   filter is broken**: `describe_resource_type` advertises it, yet the search
   endpoint 400s it as an unknown parameter (verified 8 July 2026). So you
   can't scope events to a bill structurally; browse by **date range**
   (`startDateFrom`/`startDateTo`) and match on `billName`/title, or filter the
   returned rows client-side on `billId`. Note the date filters are soft when a
   `q=` is also present (past events leak in) — see `whats-on-this-week` for
   that and the broken show endpoint.

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

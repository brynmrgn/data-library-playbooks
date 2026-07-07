---
name: track-a-bill
description: How to trace a bill's progress through Parliament — where stage data actually lives, why bill-stages is a dead end, and how scheduled bill business surfaces in the calendar
---

# Tracking a bill

The `bills` resource is the primary entity. Fetch one with
`get_document(type='bills', id=<billId>)` — the id is the numeric `billId`
(e.g. 3311 = Energy Act 2023).

**Getting the id from a browse.** Under `fields='minimal'`, bills come back
with `uri: null` — so you can't derive the id from the uri tail as you can for
other types. Use the `identifier` field instead: for bills it carries the
`billId`, which is exactly the id `get_document` wants.

## Where stage data lives — and where it doesn't

- **`bills.currentStage` is where the bill is *now***, not its full journey. It
  is a one-element array carrying the current stage only — `description`
  (e.g. "Royal Assent"), `house`, `abbreviation`, `sortOrder`. There is no
  full stage-history timeline on the bill record.
- **`isAct`** (boolean) and a `currentStage` of "Royal Assent" tell you the
  bill has passed. For the statute text itself, chain out to **lex-uk-law**.
- **Do not use the `bill-stages` resource.** It returns 0 rows for every
  `billId` (confirmed: `billId` 3311 → `total: 0`). It is effectively dead;
  nothing in its description says so. Treat it as absent.

## Forthcoming stages live in the calendar, not on the bill

Scheduled bill business — report stage, remaining Commons/Lords stages,
private members' bill (PMB) sitting Fridays — is not on the bill record. It is
in **`calendar-events`**, which carries `billId`, `billName` and
`billPageLink`. Scan or filter the calendar for the bill to see what is
*scheduled*.

Two cautions:

- The calendar is **forward-only** (see the whats-on-this-week playbook), so it
  shows upcoming sittings, not the stages a bill has already been through.
- A **PMB set down for a Friday is provisional**, not a completed stage. The
  Friday order paper is long and most set-down bills are never reached. Seeing
  a bill in the calendar for a date means it is scheduled, not that the stage
  happened — don't infer progress from a calendar entry alone.

## Why

The data library models the bill's *current* position well but not its stage
history; the What's On calendar models *scheduled* business but only forward.
Between them you can answer "where is this bill now" and "what's next for it",
but "what stages has it already cleared, and when" is not reliably answerable
from these resources — reach for the Bills API or Hansard directly for a
retrospective stage timeline.

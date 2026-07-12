---
name: analyse-written-evidence
description: >-
  Use when the user asks you to analyse or summarise the WRITTEN (or oral)
  EVIDENCE submitted to a committee inquiry — "what did people tell the X
  inquiry?", "summarise the submissions on Y", "who supported or opposed Z?" —
  even if they don't say "evidence". For a BODY of submissions, not a single
  document.
id: analyse-written-evidence
title: Analysing written evidence
task_types:
  - evidence_analysis
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the Parliamentary Data Library tools available. Reach for this when
a request is about a body of committee evidence submissions, not one
document.
<!-- /skill-only -->

# Analysing written evidence

When you are asked to analyse a body of evidence — for example, all the
written-evidence submissions to a particular committee inquiry — classify the
submitters by stakeholder camp *before* you start reading, rather than reading
in submission order. Reading in submission order over-weights whoever happened
to submit first and gives an accidental, not a representative, picture.

## Working pattern

1. **Enumerate.** Call `browse_documents` with the `committeeBusiness` filter
   set to the inquiry to list every submission. Switch to `fields='minimal'`
   and `facets='none'` once you have the inquiry's id and just want the list.

2. **Classify before reading.** Group the submitters by stakeholder type using
   the `witness_display` / `witnesses` fields — government bodies, regulators,
   industry, trade and professional bodies, unions, charities and NGOs,
   academics, and individuals.

3. **Target the high-signal voices first.** Read the umbrella organisations
   (they speak for many members), the outliers, and the most strident voices in
   each camp before anything else. These set the boundaries of the debate.

4. **Sample the rest broadly.** Read around ten individual submissions sampled
   across the camps to round out the picture, rather than reading every
   individual in order.

## Why

A committee inquiry's value is in the *range* of positions, not the volume from
any one camp. Classifying first lets you weight by stakeholder type and gives a
defensible account of who said what — "these are the positions, here is who
holds each" — instead of a first-come summary.

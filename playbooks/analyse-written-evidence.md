---
name: analyse-written-evidence
description: How to analyse a body of written-evidence submissions to a committee inquiry without over-weighting whoever submitted first
---

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

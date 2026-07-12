---
name: exhaustive-coverage-search
description: >-
  Use when the user wants EVERYTHING on a topic, not just a document about it
  — "find all…", "a complete list of…", "has Parliament looked at X at all?",
  or any claim that must be exhaustive or defensible — even when they don't
  say "everything" (a bare "what has Parliament done on X?" that expects
  completeness counts). Also when a keyword search feels like it is missing
  things.
id: exhaustive-coverage-search
title: Exhaustive coverage search
task_types:
  - topic_coverage
sources:
  - parliament
version: 1
---

<!-- skill-only -->
You have the Parliamentary Data Library tools available. Reach for this when
a request needs COVERAGE — everything on a topic — not the single best hit.
<!-- /skill-only -->

# Exhaustive coverage search

"Find the document about X" and "find *everything* on X" are different tasks,
and the tool that is right for the first silently fails the second.

## The trap: `q=` is a hard filter

BM25 keyword search (`q=` on `browse_documents`) returns only documents
matching the query terms, *then* ranks. Documents on-topic but using different
vocabulary are **excluded entirely, not ranked low**. Verified failure
(May 2026): `q=artificial intelligence medicine` against oral-evidence missed
a live 19 May session in the same inquiry because the session was about CAR-T
cell therapy and polygenic risk scores — on-topic, wrong words. Nothing in the
response signals the omission.

So: `q=` for "find the doc about X"; never trust it for "everything on X".

## Working pattern for coverage

1. **Open with a keyword browse anyway — for the facets, not the list.**
   The `facets` block on the response shows the SES term ids and structural
   ids (committee, `committeeBusiness`, publisher) that matching documents
   carry. This is discovery, not the answer.

2. **Re-query with a structural filter + date sort.** Coverage comes from
   membership, not word-matching:
   - a topic across a document type → SES `subject` term id (check
     `expand_term` narrower terms — tag granularity varies, and a document
     tagged "Photovoltaics" may not carry "Renewable energy"; note labels
     collide, e.g. two distinct "Renewable energy" concepts, 92809 and 95736);
   - a committee inquiry → `committeeBusiness=<inquiry id>`, sort
     `publicationDate desc` — the reliable enumeration path (the filter is
     flagged `hidden` in `describe_resource_type` but works; e.g. 9659 =
     "Innovation in the NHS"). Then hand off to `analyse-written-evidence`;
   - "everything from body Y" → `publisher` / `creator` / `department` ids
     (25267 = Commons Library).

3. **Add `semantic_search` for conceptual framings, at document granularity.**
   Semantic search catches the different-vocabulary documents BM25 drops. By
   default it's chunk-level (the same parent can appear several times), so for a
   coverage check pass **`granularity='document'`** — it collapses to distinct
   documents (best passage per doc, with a `passage_matches` count), giving you a
   document set you can **diff directly against the step-2 enumeration**: the
   documents semantic surfaced that your structural filter didn't are your recall
   gap. Each result carries `resource_type` + `resource_id` for a `get_document`
   follow-up. It covers only the full-text-indexed types (research-briefings,
   written-questions, written-evidence, oral-evidence, deposited-papers,
   written-statements); on catalogue-only types it returns a 0 indistinguishable
   from "no matches". It also has **no date filter** — recency scoping must come
   from the browse side.

4. **Retrieve lean.** Once ids are known: `fields='minimal'`, `facets='none'`
   — but note `minimal` currently drops `id` (needed for `get_document`);
   derive it from the uri tail or fall back to standard fields. Committee
   evidence rows are enormous — `per_page` 5–8 or the response spills.
   Escalate to `get_document` only for hits you actually need to read.

5. **State the method.** An exhaustive claim is only as defensible as its
   enumeration: "all N submissions tagged/filed under Y, date-sorted" survives
   scrutiny; "a keyword search found…" does not.

## Why

Coverage failures are silent: the response looks complete, ranks sensibly, and
is missing the most recent relevant thing. Structural filters make the
completeness claim checkable; keyword search never can.

---
name: exhaustive-coverage-search
description: How to find *everything* on a topic — why q= (keyword) search silently drops documents, and what to use instead when the goal is exhaustive coverage
---

# Exhaustive coverage of a topic

There are two different jobs hiding behind "find material on X":

- **"Find the document about X"** — precision. Keyword search is right.
- **"Find *everything* on X"** — recall. Keyword search is wrong.

`browse_documents` with `q=` is BM25 keyword matching, and **it is a hard
filter**: a document that is about your topic but uses different words —
synonyms, related terminology, an official name you didn't think of — does not
match and is **silently dropped**. You get a clean-looking result set that is
quietly incomplete. That's fine for pulling up a specific known item; it is the
wrong tool for bounding a corpus.

## For exhaustive coverage, don't lead with q=

Use one or both of these instead:

1. **Structured subject filter (the backbone).** Resolve the concept to a term
   id with `search_terms(id='ses')` — take the **SIT-class** subject id, not a
   same-label TPG/ORG twin — then
   `browse_documents(type=…, filters={subject: <id>}, sort='date')`, paging
   with `dateFrom`/`dateTo`. This returns every document *tagged* with the
   concept regardless of the words in its title or body. Discover the right
   term id from the **facets block** of a first exploratory `q=` browse: it
   lists the SES terms that cluster your keyword hits, ready to pass back as a
   filter.

2. **Semantic search (`search_content`).** Surfaces conceptually-related
   passages that don't share the query's surface vocabulary. Use it to catch
   the untagged / differently-worded tail that even a subject filter can miss.

Combine them: the subject filter gives curated-tag recall, semantic search
widens to the conceptual neighbourhood, and you reserve `q=` for locating a
specific document by name.

## Caveats

- Curated subject tags are applied by humans, so coverage tracks tagging
  completeness — a genuinely untagged document won't appear under a subject
  filter (which is why semantic search earns its place alongside).
- Some types also carry `pseudo_terms` (classifier-inferred), which widen recall
  further but are a recall aid, not ground truth — don't cite them as
  "Parliament tagged this as X".
- **Pick the right SES class for the handle.** Document subjects are SIT-class
  terms. The one exception is **research-briefing topics**, which use TPG-class
  terms — so for a briefing topic sweep a TPG id is the *right* handle, whereas
  everywhere else (written questions and the rest) a TPG term matches nothing.
  Same label can exist in both classes; take the class that matches the field
  you're filtering.

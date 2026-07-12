# Authoring a playbook

A playbook is a short procedural guide for a **recurring** UK Parliament
research task — the debugged trace of how to use the data-library MCP's tools
to a given end, so an agent inherits the experience instead of thrashing. If a
task is one-off, or is a simple lookup, it doesn't need a playbook.

## Where it goes live

The MCP server serves these files at runtime from this repo (`PLAYBOOKS_REPO` /
`PLAYBOOKS_REF`, default `main`). Merge to `main` and the change is live within
the cache TTL — **~5 min** for the `list_playbooks` listing, **~60 s** for a
playbook's content. No redeploy. To trial a draft before merging, point the
server's `PLAYBOOKS_REF` at your branch.

Keep the **bundled snapshot** in the MCP repo (`data-library-mcp/playbooks/`)
in sync — it's the fallback served when this repo is unreachable, so a new or
changed playbook should be mirrored there in the same change set.

## File and frontmatter

Create `playbooks/<name>/SKILL.md` (one folder per playbook):

```markdown
---
name: <name>
description: >-
  Trigger-shaped — when should an agent reach for this? Give concrete phrasings
  and an explicit "even if the user doesn't say X". e.g. "Use when the user asks
  about a bill's progress, current stage, or amendments — even if they don't say
  'bill' (e.g. 'where has the Renters' Rights Bill got to?')."
id: <name>
title: Human-readable title
task_types:
  - <one or more controlled task-type slugs>
sources:
  - parliament
version: 1
---

<!-- skill-only -->
Optional framing shown only to the skill-loading path — stripped from the
get_playbook body, kept for a fresh skill load.
<!-- /skill-only -->

# Title
...
```

- **`name` must equal the folder name.** Each playbook is its own directory,
  `playbooks/<name>/SKILL.md`; `get_playbook` keys on the slug.
- **`description` is the routing signal.** `list_playbooks` returns the full
  frontmatter (name, id, title, description, task_types, sources, version), so
  the description is what an agent — or the host classifier — matches a task
  against. Write it trigger-shaped. The parser is full YAML (YamlDotNet):
  folded scalars (`>-`) and lists are supported, and unknown keys are ignored.
- **`task_types`** are the controlled-vocabulary intent tags the host classifier
  binds to (e.g. `bill_tracking`, `pq_analysis`). The host owns the canonical
  registry — add new types through it, don't coin them here.
- **`sources`** declares which corpora the playbook touches — `[parliament]` for
  a single-source playbook. A cross-source composite lists all its sources and
  is served host-side only; this MCP serves a playbook iff its `sources` are a
  subset of what it owns. Absent/empty = served everywhere.
- **`<!-- skill-only -->` … `<!-- /skill-only -->`** blocks are stripped from the
  `get_playbook` body but kept for the skill-loading path — use for up-front
  framing that only helps a fresh skill load.

## The quality bar

- **Verify first.** Ground every claim in live MCP behaviour before you write
  it — reproduce the task, capture the real field names, query output and
  failure modes. A playbook that parrots an assumption is worse than none. Most
  of the value is in the traps, and traps have to be observed.
- **Include a worked example with real output** where the task has one (a query
  and its actual result, a real id, a real field shape).
- **Lead with the traps.** State the thing that will otherwise cost the reader
  the query — the dead endpoint, the silent filter, the timeout shape.
- **Name the neighbour at the point of need.** When a task naturally chains to
  another tool or MCP (statute text → lex-uk-law, member data → MNIS), say so
  inline where it's relevant. A pointer degrades gracefully if that neighbour
  isn't connected. Don't build a separate cross-tool index — that's what these
  inline pointers and `list_playbooks` are for.
- **Cross-reference siblings by name** ("see scope-a-sparql-aggregation").
- **Match the house style** of the existing playbooks: dense, specific,
  opinionated, second person, short sections.

## Checklist before you open a PR

- [ ] `name` matches the folder name (`playbooks/<name>/SKILL.md`)
- [ ] `description` reads as a trigger/routing entry
- [ ] `task_types` use the host's controlled vocabulary; `sources` is set
- [ ] Every factual claim was checked against the live MCP
- [ ] A worked example is included where applicable
- [ ] Neighbouring tools/MCPs are named inline where the task chains out
- [ ] The bundled snapshot in `data-library-mcp/playbooks/` is updated to match

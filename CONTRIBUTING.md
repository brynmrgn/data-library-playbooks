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

Create `playbooks/<name>.md`:

```markdown
---
name: <name>
description: One line — task-shaped, e.g. "How to trace a bill through Parliament — where stage data lives and why bill-stages is a dead end"
---

# Title
...
```

- **`name` must equal the filename** (minus `.md`). `get_playbook` keys on the
  slug.
- **`description` must be a single physical line.** `list_playbooks` returns
  *only* name + description, so it is the routing index — write it task-shaped
  so an agent can match it to the shape of a task without opening the file.
  Note the server's frontmatter parser is naive (`key: value` per line): it does
  **not** support YAML folded scalars (`>-`) or multi-line values. One line.

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

- [ ] `name` matches the filename
- [ ] `description` is one line and reads as a routing entry
- [ ] Every factual claim was checked against the live MCP
- [ ] A worked example is included where applicable
- [ ] Neighbouring tools/MCPs are named inline where the task chains out
- [ ] The bundled snapshot in `data-library-mcp/playbooks/` is updated to match

# data-library-playbooks

Reference playbooks for the [data-library MCP server](https://github.com/brynmrgn/data-library-mcp)
— short procedural guides for recurring UK Parliament research tasks.

Each playbook lives at `playbooks/<name>/SKILL.md`: a markdown document with YAML
frontmatter (`name`, `description`, `id`, `title`, `task_types`, `sources`,
`version`) in the shared **SKILL.md** format, so the files double as agent skills
and are authored once but rendered per consumer (Claude skill loader, host
injection, and the MCP's `get_playbook`). The MCP server fetches them at runtime:
edit a file here and the change goes live within ~60 seconds, no redeploy. The
server keeps a bundled snapshot as a fallback for when this repo is unreachable.

Add a playbook by creating a new `playbooks/<name>/SKILL.md` — no code change
needed. See [CONTRIBUTING.md](CONTRIBUTING.md) for the frontmatter contract.

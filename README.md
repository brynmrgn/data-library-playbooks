# data-library-playbooks

Reference playbooks for the [data-library MCP server](https://github.com/brynmrgn/data-library-mcp)
— short procedural guides for recurring UK Parliament research tasks.

Each file under `playbooks/` is a markdown document with `name` and
`description` frontmatter (the Claude Code skill format, so the files double as
skills). The MCP server fetches them at runtime: edit a file here and the change
goes live within ~60 seconds, no redeploy. The server keeps a bundled snapshot
as a fallback for when this repo is unreachable.

Add a playbook by dropping a new `playbooks/<name>.md` file in — no code change
needed.

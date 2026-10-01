# Contributing

## Adding a new skill

1. `skills/<name>/SKILL.md` with frontmatter: `name`, `description` (specific — this is what the agent matches against, not a category label), `metadata.version`.
2. Keep it client-agnostic — never hardcode a `scriptId`, `spreadsheet_id`, or client name. Client-specific values belong in that client's own project, not here.
3. If the skill wraps a CLI or library with commands that change between versions (like `clasp` did), verify command names against current docs before writing them down — don't write from memory.
4. Register it in `.claude-plugin/marketplace.json` under `skills` and list it in `README.md`.
5. Supporting files (templates, detailed references) go under `skills/<name>/references/`, not loose in the skill root.

## Updating an existing skill

If the underlying tool changes its CLI/API (clasp majors, MCP server tool renames), update the skill in the same PR as the discovery — don't let it drift silently until a dev hits the stale command.

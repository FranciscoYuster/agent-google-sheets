# AGENTS.md

Instructions for any coding agent (Claude Code, Cursor, Codex, Copilot, etc.) working inside this repo.

## What this repo is

A source of Agent Skills for Google Apps Script + Google Sheets client projects. It does not run anything itself — it ships `SKILL.md` files that other projects install via `npx skills add` or `claude plugin install`. Consumers are internal devs building separate Apps Script client projects (one repo per client, each with its own `scriptId` and spreadsheet).

## Layout

```
skills/<name>/SKILL.md          required manifest, YAML frontmatter: name, description, metadata
skills/<name>/references/       optional: templates, detailed docs the skill points to
.claude-plugin/marketplace.json install manifest for `claude plugin install`
.github/workflows/lint-skills.yml  CI: validates every SKILL.md has the required frontmatter fields
README.md                       human-facing install instructions + skill catalog
CONTRIBUTING.md                 PR/review process for humans
```

## Rules for editing or adding a skill

1. **Client-agnostic, always.** No skill may hardcode a `scriptId`, `spreadsheet_id`, client name, or any other client-specific value. Those belong in that client's own project (its `CLAUDE.md`, its `.clasp.json`), never here. If you're tempted to hardcode something, it means the skill needs a "Required Input" section instead that tells the agent how to resolve it from user input.
2. **Verify CLI/API commands against current docs before writing them down — never from memory or assumption.** `clasp` majors have silently renamed commands before (`clone` → `clone-script`, `deployments` → `list-deployments`, `login --status` never existed, the real command is `show-authorized-user`). Use Context7 or the tool's official docs/README to confirm a command exists and takes the flags you're about to document.
3. **Supporting files go in `skills/<name>/references/`.** Never loose in the skill root, never duplicated inline in `SKILL.md` if they're more than a few lines.
4. **Register every new or renamed skill in two places:** the `skills` array in `.claude-plugin/marketplace.json`, and the `## Available Skills` section of `README.md`. A skill not listed in both is invisible to half the install paths.
5. **Required frontmatter:** `name`, `description` (specific — this is what an agent matches against when deciding to auto-invoke the skill, not a category label), `metadata.version`. CI (`lint-skills.yml`) rejects a `SKILL.md` missing `name:` or `description:` or the `---` frontmatter delimiter.
6. **No AI-attribution footers, no emoji** in commit messages or in generated file content (code, docs, `SKILL.md`). This is a standing repo convention, not just a one-off request.

## Before committing

- Re-read the skill you touched end-to-end — does it still read as client-agnostic, does every command in it actually exist in the current version of the tool it wraps.
- If you changed a skill that other skills reference (e.g. `clasp` is referenced by `appscript-scaffold`), check the cross-reference still makes sense.
- Scan for emoji and attribution lines before pushing (`grep -rn "Co-Authored-By\|Generated with" .` and a unicode-emoji grep) — nothing in this repo should carry them.

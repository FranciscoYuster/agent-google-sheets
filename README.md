# Agent Google Sheets

Agent Skills for building and operating Google Apps Script projects bound to Google Sheets — reading/writing spreadsheet data, pushing code with clasp, and scaffolding new client projects. Built for AI coding agents (Claude Code and compatible tools).

Each client project (their own repo, their own Sheet, their own Apps Script project) installs these skills instead of duplicating this knowledge per repo.

## Installation

### Install all skills
```bash
npx skills add FranciscoYuster/agent-google-sheets
```

### Install a specific skill
```bash
npx skills add FranciscoYuster/agent-google-sheets --skill mcp-google-sheets
npx skills add FranciscoYuster/agent-google-sheets --skill clasp
npx skills add FranciscoYuster/agent-google-sheets --skill appscript-scaffold
```

### Claude Code plugin (alternative)
```bash
claude plugin marketplace add FranciscoYuster/agent-google-sheets
claude plugin install agent-google-sheets@agent-google-sheets
```

## Available Skills

<details>
<summary><strong>mcp-google-sheets</strong></summary>

Google Sheets read/write operations via the `mcp-google-sheets` MCP server (xing5/mcp-google-sheets). Resolves `spreadsheet_id` from a pasted URL or by name search — never hardcodes one.

**Use when:** reading or writing spreadsheet data, creating/organizing tabs, batch updates, searching spreadsheets in Drive, sharing, adding charts.

</details>

<details>
<summary><strong>clasp</strong></summary>

Push/pull Google Apps Script projects with the `clasp` CLI. Validates login (`clasp show-authorized-user`) before any command, resolves `scriptId` from a spreadsheet's bound script link, covers both setup paths (existing client script vs. brand-new), and documents real clasp 3.x command names (`clone-script`, `list-deployments`, `create-deployment`/`deploy` alias) plus common GAS pitfalls (single `doGet()`, pinned deployments, manifest force-push risk).

**Use when:** setting up clasp for a client, pushing/pulling code, deploying or redeploying a web app, diagnosing a push that silently did nothing or a deployment serving stale code.

</details>

<details>
<summary><strong>appscript-scaffold</strong></summary>

Scaffolds a brand-new Apps Script client project from self-contained templates (`package.json` with clasp npm scripts, `appsscript.json`, `.gitignore`, a minimal `onOpen()` skeleton) — no dependency on an external boilerplate repo.

**Use when:** starting a client with no existing Apps Script code yet.

</details>

## Design notes

- Skills are **client-agnostic** — none hardcode a `scriptId`, `spreadsheet_id`, or client name. Client-specific IDs and quirks belong in that client project's own `CLAUDE.md`.
- `clasp` skill commands were verified against the official clasp docs (via Context7) as of this repo's last update — clasp has broken command names between major versions before (`clone` → `clone-script`, `deployments` → `list-deployments`), re-verify if clasp majors again.
- `appscript-scaffold` keeps its templates in `references/` rather than pointing at a separate template repo, to avoid two sources of truth drifting apart.

## Skill Structure

Each skill follows the [Agent Skills Open Standard](https://agentskills.io/):

- `SKILL.md` — required skill manifest with frontmatter (name, description, metadata)
- `references/` — optional supporting files (templates, detailed docs)

---
name: appscript-scaffold
description: "Scaffold a new Google Apps Script client project from scratch — package.json with clasp npm scripts, appsscript.json manifest, .gitignore, and a minimal onOpen() skeleton. Client-agnostic, self-contained (no external boilerplate repo dependency)."
metadata:
  author: altavia
  version: "1.0"
---

## When to Use

Starting a **brand-new** client project with no existing Apps Script code — greenfield, no `.clasp.json`, no `codigo.js` yet. Not for pushing/deploying existing code (that's the `clasp` skill) and not for reading/writing spreadsheet data (that's `mcp-google-sheets`).

## Required Input

Ask (or infer from context) before generating files:
- **Client/business name** — used for the npm package name and the `onOpen()` menu label.
- **Timezone** — default `America/Santiago` for this agency's typical client base; confirm if unsure, don't guess for an unfamiliar client.

## Setup Flow

1. Create the project directory, `cd` into it.
2. Copy the four templates from `references/` into the project root, substituting placeholders:
   - `package.json.tmpl` → `package.json` (`{{PACKAGE_NAME}}` → kebab-case client name)
   - `appsscript.json.tmpl` → `appsscript.json` (`{{TIMEZONE}}`)
   - `gitignore.tmpl` → `.gitignore` (no placeholders)
   - `codigo.js.tmpl` → `codigo.js` (`{{MENU_LABEL}}` → e.g. `"<Client>"`)
3. `npm install` — pulls `@types/google-apps-script` for editor autocomplete.
4. Hand off to the **`clasp` skill** to create or bind the actual Apps Script project (`clasp create --type sheets --title "<Client>"` for new, or `clasp clone-script <url>` if the client already has a script — see that skill's two setup paths).
5. `npm run push` (or `clasp push`) to confirm the skeleton deploys cleanly before writing real logic.
6. `git init && git add -A && git commit -m "init: proyecto <cliente>"` — each client project is its own repo, never a branch of this skill or of another client's repo.

## Notes

- Self-contained by design: templates live in `references/` so this works offline and never drifts from a separate template repo. If you've previously used a standalone `boilerplate-google-sheet`-style repo, treat that as a manual fallback for devs without Claude — keep it in sync with these templates if both are maintained, or retire it in favor of this skill.
- Pairs with [[clasp]] (push/pull/deploy, login validation) and [[mcp-google-sheets]] (verifying data once the sheet is live).
- Global skill (`~/.claude/skills/`) — reused across every client project.

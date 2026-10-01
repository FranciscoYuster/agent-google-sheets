---
name: clasp
description: "Push/pull Google Apps Script projects with clasp CLI — setup from a spreadsheet link, login validation, push/deploy workflow, and common GAS pitfalls. Client-agnostic."
metadata:
  author: altavia
  version: "1.0"
  cli: clasp
  source: https://github.com/google/clasp
---

## When to Use

Use this skill when:
- Setting up clasp for a new client's Apps Script project
- Pushing local code changes to a bound/standalone GAS script
- Pulling remote script state to inspect drift before a force-push
- Deploying or redeploying a web app (`/exec` link)
- Diagnosing a push that's silently skipped or a web app serving stale/wrong output

This skill is client-agnostic — it never hardcodes a `scriptId`. Each project supplies its own via its local `.clasp.json`.

---

## Prerequisites (validate before any clasp command)

**1. clasp installed**
```
clasp --version
```
Missing → `npm install -g @google/clasp`.

**2. Logged in** — never assume. Check first:
```
clasp show-authorized-user
```
(`clasp login --status` does NOT exist in clasp 3.x — this is the real command; `--json` for parseable output). If it errors or shows no user, run `clasp login` (opens browser OAuth) and wait for the user to complete it — do not attempt pushes against an unauthenticated session, they fail opaquely or push to the wrong account.

**3. Right account.** `clasp show-authorized-user` shows which OAuth client/account is active. If the client's script belongs to a different Google account than the one logged in, `clasp login -u <name> --creds <file>` for a named profile (clasp 3.x syntax; `login --creds <file>` alone is the deprecated 2.x form) — never push blind across accounts.

## Required Input: spreadsheet link, not scriptId

Users hand you a **spreadsheet URL**, not a script ID — don't ask them to go dig it up manually. Resolve it:

1. Get the spreadsheet URL (`https://docs.google.com/spreadsheets/d/<SHEET_ID>/edit...`).
2. The bound Apps Script project's `scriptId` is **not** derivable from the sheet URL by pattern — the Sheets/Drive API doesn't expose it, and `mcp-google-sheets` has no tool for it either. Get it from the user: **Extensions → Apps Script** opens the bound script, whose own URL is `https://script.google.com/.../projects/<SCRIPT_ID>/edit` — ask them to paste that URL (or just the ID from it).
3. `clasp clone-script` accepts either the raw ID or the full script editor URL directly — no manual extraction needed:
   ```
   clasp clone-script "https://script.google.com/d/<SCRIPT_ID>/edit" --rootDir ./src
   ```
4. Once resolved, verify: `.clasp.json` must contain that exact `scriptId`. If a `.clasp.json` already exists in the repo, diff its `scriptId` against what the user gave you before trusting it — stale clones from a copied spreadsheet are a common source of pushing to the wrong project.

Never fabricate or reuse a `scriptId` from another client's project out of convenience.

---

## Setup Flow — two cases, don't conflate them

**A. Cliente ya tiene planilla con script (caso Altavia)** — resolver el link del script (sección anterior), luego:
```
clasp show-authorized-user
clasp clone-script "https://script.google.com/d/<SCRIPT_ID>/edit" --rootDir ./src
cat .clasp.json                            # sanity-check scriptId matches what the client gave you
```

**B. Cliente nuevo, sin planilla aún** — crear desde cero, no hay link que resolver:
```
clasp show-authorized-user
clasp create --type sheets --title "Nombre del Cliente"   # crea Sheet + script vinculado, genera .clasp.json
```
Si el repo parte de un boilerplate de carpetas (p. ej. `boilerplate-google-sheet`: `package.json` con scripts `push/pull/open/logs/watch/deploy`, `appsscript.json` default, `.gitignore`), copiar esa carpeta ANTES de `clasp create` — no clonar el boilerplate repo en sí, es template, nunca se trabaja directo ahí. Cada cliente es un repo git separado (`git init` propio, no hereda historial del boilerplate).

## Push Flow (existing project)

Si hay `package.json` con script `push` (patrón boilerplate), preferir `npm run push` / `npm run deploy` sobre clasp directo — ya encapsula el comando y mantiene un solo punto de verdad si el equipo cambia flags.

```
clasp show-authorized-user                 # always confirm first — sessions expire
clasp pull --rootDir ./tmp-check           # optional: pull to a scratch dir, diff manifest before forcing
clasp push --force                         # clasp 3.x asks for TTY confirmation on manifest changes; non-interactive sessions need --force
# o, si existe: npm run push
```

**Before `--force`:** compare local `appsscript.json` against the remote manifest (via the scratch pull above). A force-push that silently changes `webapp.access` (e.g. `MYSELF` vs `ANYONE_ANONYMOUS`) can lock out or expose a shared web app link without warning.

## Deploy Flow (web app / API executable)

```
clasp list-deployments                     # list existing deployments + which version each @N points to (NOT `clasp deployments` — doesn't exist)
clasp deploy --deploymentId <id> -V <version>   # alias of `create-deployment`; redeploy a pinned deployment to pick up latest push
```

`clasp push` only updates `HEAD` — a deployment pinned to `@2`, `@7`, etc. keeps serving the old code until explicitly redeployed. If the user reports "I pushed but the web app still shows old behavior," this is almost always why — check `clasp list-deployments` before debugging code.

---

## Common Pitfalls

- **"Skipping push." with no error** — clasp ≥3.x prompts for manifest-change confirmation; no TTY means it defaults to no. Fix: `--force`, but only after verifying the manifest diff (see Push Flow).
- **Multiple `doGet()` across files** — Apps Script silently picks one; the web app can start returning the wrong handler's output with no error anywhere. `grep -rn "function doGet" --include="*.js" --include="*.gs"` before touching any web-app-serving file.
- **Deployments pinned to old versions** — see Deploy Flow above.
- **Wrong account logged in** — pushes succeed but land on the wrong script/project, invisible until the client says "I don't see the change."

---

## Notes

- Global skill (`~/.claude/skills/`) — reused across every client project. Client-specific `scriptId`, deployment IDs, and quirks belong in that project's `CLAUDE.md` or its memory, never edited into this file.
- Pairs with the `mcp-google-sheets` skill: resolving a spreadsheet's bound script often starts from the same URL the sheets skill uses to resolve `spreadsheet_id`.

---
name: mcp-google-sheets
description: "Google Sheets operations via xing5/mcp-google-sheets MCP — read, write, batch update, create, search, share, and chart spreadsheets. Works across any client spreadsheet, not project-specific."
metadata:
  author: altavia
  version: "1.1"
  mcp_server: mcp-google-sheets
  source: https://github.com/xing5/mcp-google-sheets
---

## When to Use

Use this skill when working with Google Sheets:
- Reading or writing spreadsheet data
- Creating or organizing sheets/tabs
- Batch updating multiple ranges
- Searching across spreadsheets in Drive
- Sharing spreadsheets with users
- Adding charts to sheets

This skill is client-agnostic — it never hardcodes a `spreadsheet_id`. Each project supplies its own via its `CLAUDE.md` or the user's message.

---

## Prerequisites (validate before any tool call)

1. **Server enabled.** If `mcp__mcp-google-sheets__*` tools aren't in the active tool list, the MCP server isn't registered for this project — see Setup below. Check root `.mcp.json` (not `.claude/.mcp.json` — common misplacement) for the `mcp-google-sheets` entry before assuming it's broken.
2. **Auth present.** Depends which auth method is configured (see Setup) — a credential-file setup needs `.claude/mcp-google-sheets-token.json` (or `TOKEN_PATH`) to exist; an ADC setup needs `gcloud auth application-default login` already run on this machine. If the first tool call returns an auth/401/403 error, tell the user re-auth is needed — don't silently retry in a loop.
3. **Never assume credentials exist.** This skill ships with no credentials of any kind — installing it via `npx skills add` or `claude plugin install` gets you the `SKILL.md` only. A dev who just installed this on a new machine, or an external dev with no access to this org's GCP project, has nothing set up yet. Don't reference "the" `gcp-oauth.keys.json` as if it's guaranteed to exist — check first, and if missing, walk them through Setup below (ADC path first — it's the one that needs nothing from anyone else).

## Required Input

Never guess a `spreadsheet_id`. Resolve it one of these ways, in order:
1. User pastes a Sheets URL (`https://docs.google.com/spreadsheets/d/<ID>/edit...`) → extract `<ID>` between `/d/` and the next `/`.
2. User names the spreadsheet → `search_spreadsheets(query="<name>")`, confirm the single match with the user before writing (ambiguous matches: list and ask).
3. Project already documents the ID (e.g. in `CLAUDE.md`) → use it, but confirm it still resolves (`list_sheets`) before a write — IDs can be stale if the client copied/recreated the sheet.

---

## MCP Server

**Name:** `mcp-google-sheets` (xing5/mcp-google-sheets)
**Requires:** Google Sheets API + Google Drive API enabled on whichever GCP project backs the chosen auth method.

### Setup — pick one method, in this priority order

**Method 1: Application Default Credentials (ADC) — default recommendation, especially for a dev with no prior access to this org's GCP project.**

Zero credential files, no OAuth client to create. One-time per dev machine:
```bash
gcloud auth application-default login \
  --scopes=https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/spreadsheets,https://www.googleapis.com/auth/drive
gcloud auth application-default set-quota-project <ANY_GCP_PROJECT_ID>   # free tier works; just needs Sheets+Drive API enabled
```
Then register the server with no auth env vars at all — it falls through to ADC automatically:
```bash
claude mcp add mcp-google-sheets -s project -- uvx --with "mcp<2" mcp-google-sheets@latest
```
Requires `gcloud` CLI installed (`curl -LsSf https://sdk.cloud.google.com | bash` or the platform installer). This is the only method where "install the skill" and "get access" don't require anyone to hand you a file — the dev logs in with their own Google account, and real access to a given Sheet still comes from that Sheet being shared with them, same as always.

**Method 2: Shared OAuth client credential file — when the org already maintains one.**
```bash
claude mcp add mcp-google-sheets -s project \
  -e CREDENTIALS_PATH=.claude/gcp-oauth.keys.json \
  -e TOKEN_PATH=.claude/mcp-google-sheets-token.json \
  -- uvx --with "mcp<2" mcp-google-sheets@latest
```
```
.claude/gcp-oauth.keys.json       ← OAuth client credentials (from GCP Console, Credentials → OAuth client ID → Desktop app) — git-ignored, per developer, NOT something `npx skills add` ships or that a new/external dev has by default
.claude/mcp-google-sheets-token.json  ← generated on first browser login, git-ignored
```
This is the pre-existing path in Altavia-style client projects. It still requires an interactive browser login on first use — `CREDENTIALS_PATH` identifies the app, it is not a bypass of login.

**Method 3: Service Account — for headless/CI use only**, not appropriate for an interactive dev workflow. See the upstream README's Method A if this comes up.

---

## Tool Reference

### Read

| Tool | Description | Key params |
|------|-------------|------------|
| `list_spreadsheets` | List spreadsheets in configured Drive folder | `folder_id?` |
| `list_sheets` | List all tabs in a spreadsheet | `spreadsheet_id` |
| `get_sheet_data` | Read data from a range | `spreadsheet_id`, `range`, `sheet_name?` |
| `get_sheet_formulas` | Read formulas (not computed values) from a range | `spreadsheet_id`, `range` |
| `get_multiple_sheet_data` | Fetch data from multiple ranges/spreadsheets in one call | `requests[]` |
| `get_multiple_spreadsheet_summary` | Titles, tab names, headers, and first rows of multiple sheets | `spreadsheet_ids[]` |
| `find_in_spreadsheet` | Search for a value within a spreadsheet | `spreadsheet_id`, `query` |
| `search_spreadsheets` | Search across multiple spreadsheets in Drive | `query` |
| `list_folders` | List Drive folders | — |

### Write

| Tool | Description | Key params |
|------|-------------|------------|
| `update_cells` | Write data to a range (overwrites) | `spreadsheet_id`, `range`, `values[][]` |
| `batch_update_cells` | Update multiple ranges in one API call | `spreadsheet_id`, `data[]` |
| `batch_update` | General batch operations (formatting, merges, etc.) | `spreadsheet_id`, `requests[]` |
| `add_rows` | Insert empty rows at index | `spreadsheet_id`, `sheet_name`, `row_index`, `num_rows` |
| `add_columns` | Insert empty columns at index | `spreadsheet_id`, `sheet_name`, `col_index`, `num_cols` |

### Structure

| Tool | Description | Key params |
|------|-------------|------------|
| `create_spreadsheet` | Create new spreadsheet | `title` |
| `create_sheet` | Add new tab to existing spreadsheet | `spreadsheet_id`, `title` |
| `rename_sheet` | Rename existing tab | `spreadsheet_id`, `sheet_id`, `new_title` |
| `copy_sheet` | Duplicate tab between spreadsheets | `source_spreadsheet_id`, `sheet_id`, `dest_spreadsheet_id` |
| `share_spreadsheet` | Share with users and set role | `spreadsheet_id`, `email`, `role` |
| `add_chart` | Create chart from data range | `spreadsheet_id`, `sheet_name`, `chart_type`, `data_range` |

---

## Usage Patterns

### Read a sheet
```
get_sheet_data(spreadsheet_id="<id>", range="Sheet1!A1:Z100")
```

### Batch write (prefer over single update_cells when >1 range)
```
batch_update_cells(spreadsheet_id="<id>", data=[
  {"range": "Sheet1!A1:B2", "values": [["h1","h2"],["v1","v2"]]},
  {"range": "Sheet2!A1", "values": [["summary"]]}
])
```

### Find existing spreadsheet by name
```
search_spreadsheets(query="Rendiciones AVSS")
```

---

## Notes

- `update_cells` overwrites — use `get_sheet_data` first if preserving existing data matters
- `batch_update_cells` is a single API call — always prefer it over multiple `update_cells`
- `get_sheet_formulas` returns raw formulas; `get_sheet_data` returns computed values
- Tokens stored at `.claude/mcp-google-sheets-token.json` (project-local); re-auth triggers browser OAuth
- Global skill (`~/.claude/skills/`) — reused across every client project. Client-specific IDs and quirks belong in that project's `CLAUDE.md`, never edited into this file.

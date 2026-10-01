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

1. **Auth present.** Check `.claude/mcp-google-sheets-token.json` (or the path set in `TOKEN_PATH`) exists. If missing or the first call returns an auth/401 error, tell the user re-auth is needed (browser OAuth triggers automatically on next call) — don't silently retry in a loop.
2. **Server enabled.** If `mcp__mcp-google-sheets__*` tools aren't in the active tool list, the MCP server isn't registered for this project. Check root `.mcp.json` (not `.claude/.mcp.json` — common misplacement) for the `mcp-google-sheets` entry before assuming it's broken.
3. **Credentials scoped per dev.** `.claude/gcp-oauth.keys.json` and `.claude/mcp-google-sheets-token.json` are git-ignored and per-developer — never commit, never copy between machines as a shortcut.

## Required Input

Never guess a `spreadsheet_id`. Resolve it one of these ways, in order:
1. User pastes a Sheets URL (`https://docs.google.com/spreadsheets/d/<ID>/edit...`) → extract `<ID>` between `/d/` and the next `/`.
2. User names the spreadsheet → `search_spreadsheets(query="<name>")`, confirm the single match with the user before writing (ambiguous matches: list and ask).
3. Project already documents the ID (e.g. in `CLAUDE.md`) → use it, but confirm it still resolves (`list_sheets`) before a write — IDs can be stale if the client copied/recreated the sheet.

---

## MCP Server

**Name:** `mcp-google-sheets`
**Auth:** OAuth 2.0 via `CREDENTIALS_PATH` / `TOKEN_PATH`
**Requires:** Google Sheets API + Google Drive API enabled in GCP

**Setup por dev** (archivos git-ignored, cada dev pone los suyos):
```
.claude/gcp-oauth.keys.json       ← OAuth client credentials (de GCP Console)
.claude/mcp-google-sheets-token.json  ← generado automático al autenticar
```

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

## Why

Two gaps in the current TUI:

1. **Invisible documentation** — the CSV input file contains inline comments like `# Indicated airspeed (knots * 128)` that describe each offset. These are stripped at parse time and never shown. Users browsing offsets in the TUI have no idea what each offset means.

2. **JSON-only snapshots** — the only snapshot format is verbose JSON. For quick data analysis, scripting, or diffing, a lean CSV (`$Offset,$Value`) is much more convenient.

Additionally, after editing offsets in the TUI (Add/Edit/Delete), saving via `W` discards all inline comments. These should round-trip faithfully.

## What Changes

- Add `Comment` field to `OffsetDefinition`, captured during CSV parsing
- Add a 6th "Comment" column to the TUI table (dynamic width)
- Add comment prompts to Add and Edit flows
- `WriteFile` appends comments on save
- Replace `S` key with `J` (JSON snapshot) and `C` (CSV snapshot) keys
- CSV snapshot writes `Offset,Value` format with proper escaping

## Capabilities

### New Capabilities

- `tui-inline-comments`: Inline comment display in table, comment round-trip on write, comment prompts in Add/Edit
- `csv-snapshot`: CSV snapshot output format with `J`/`C` keybinding pair

### Modified Capabilities

- `tui-table-viewport`: Comment column added to existing layout

## Impact

- `Models.cs`: `OffsetDefinition` gets `string? Comment` field
- `OffsetParser.cs`: Parsing captures comments; `WriteFile` writes them back
- `TuiMode.cs`: New table column, comment prompts, `J`/`C` keybinding replaces `S`
- `BatchMode.cs`: (unchanged — JSON snapshot still uses `FormatSnapshot`)
- `TuiMode.cs` `TuiState`: (unchanged — dirty tracking already exists)
- `ParserTests.cs`: New tests for comment parsing and round-trip

## 1. Add Comment to OffsetDefinition

- [ ] 1.1 Add `string? Comment` parameter to `OffsetDefinition` record (default `null`)
- [ ] 1.2 Update `OffsetParser.Parse()` to capture inline comment text after `#` stripping
- [ ] 1.3 Update `OffsetParser.WriteFile()` to append `"  # {Comment}"` when non-empty
- [ ] 1.4 Verify build passes, format clean

## 2. Update parser tests

- [ ] 2.1 Update `ParseWithInlineComment` test to assert `Comment` is captured
- [ ] 2.2 Update `ParseMultipleWithInlineComments` test to assert comments per offset
- [ ] 2.3 Add test: `ParseOffsetWithoutComment` — no `#` → `Comment` is null
- [ ] 2.4 Add test: `WriteFilePreservesComment` — parse + write, verify comment in output
- [ ] 2.5 Add test: `WriteFileOmitsNullComment` — offset with null comment → no `#` in output
- [ ] 2.6 Add test: `CommentOnlyHash` — `0x02BC,i32  #` → `Comment` is empty string or null
- [ ] 2.7 Verify all tests pass

## 3. Add comment column to TUI table

- [ ] 3.1 Add `"Comment"` column to table in `BuildLayout()` — no fixed width, auto-sizing
- [ ] 3.2 In `AddRow()`, render `def.Comment` as a 6th cell, truncation handled by Spectre
- [ ] 3.3 `EscapeMarkup()` comment text before rendering
- [ ] 3.4 Apply `[reverse]` markup on comment cell when selected
- [ ] 3.5 Verify build, format, and run

## 4. Add comment prompts to Add and Edit flows

- [ ] 4.1 In `PromptAdd()`, add `TextPrompt<string?>` for comment after size prompt, with `AllowEmpty()` → sets `Comment` to null if empty
- [ ] 4.2 In `PromptEdit()`, add comment prompt with default value = existing comment; empty clears it to null
- [ ] 4.3 Verify comment changes set `IsDirty = true` (covered by existing dirty-on-edit logic)
- [ ] 4.4 Verify build, format, and run

## 5. Implement CSV snapshot output

- [ ] 5.1 Add `BatchMode.FormatCsvSnapshot(IReadOnlyList<RegisteredOffset>)` → `string`
- [ ] 5.2 Implement CSV escaping: null→empty, bytes→hex, string with commas→quoted
- [ ] 5.3 Write header row `Offset,Value`
- [ ] 5.4 Verify build, format

## 6. Implement J / C keybinding pair, remove S

- [ ] 6.1 Replace `S` entry in `KeyAction` enum with `JsonSnapshot` and `CsvSnapshot`
- [ ] 6.2 Add `ConsoleKey.J` → `KeyAction.JsonSnapshot` and `ConsoleKey.C` → `KeyAction.CsvSnapshot` in `HandleKey()`
- [ ] 6.3 Remove `ConsoleKey.S` mapping from `HandleKey()`
- [ ] 6.4 In the main loop, split the old `KeyAction.Save` case into two:
  - `JsonSnapshot`: same as current `Save` (uses `BatchMode.FormatSnapshot()`)
  - `CsvSnapshot`: calls new `FormatCsvSnapshot()`, saves `.csv`
- [ ] 6.5 Verify filenames: `{prefix}-{timestamp}-{hostname}.json` and `.csv`
- [ ] 6.6 Update help panel: replace `s` with `[j]    Save JSON snapshot` and `[c]    Save CSV snapshot`
- [ ] 6.7 Verify build, format

## 7. Final verification

- [ ] 7.1 Run `dotnet build`
- [ ] 7.2 Run `dotnet format` on both projects
- [ ] 7.3 Run `dotnet test`

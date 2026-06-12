## Context

Three tightly related features sharing the comment data path through the pipeline:

```
CSV file ──▶ Parse ──▶ OffsetDefinition ──▶ Table render
                              │  (Comment)
                              ▼
                        WriteFile ──▶ CSV file (with comments)
```

The CSV snapshot feature is independent but shares the keybinding change.

## Goals / Non-Goals

**Goals:**
- Inline comments from CSV displayed in TUI table as a 6th dynamic-width column
- Comments round-trip through `WriteFile` (W key)
- Comment prompts in Add and Edit flows
- `J` key saves JSON snapshot, `C` key saves CSV snapshot
- CSV snapshot: `Offset,Value` header + data rows with proper escaping

**Non-Goals:**
- Full-line comment preservation (blank lines, `#`-only lines)
- Comment editing as a standalone action
- Snapshot comparison or history
- Binary snapshot format

## Decisions

### Comment storage

`OffsetDefinition` gets a `string? Comment` property. This is the minimal change — no line-tracking, no new types. Full-line comments and blank lines in the source file are still discarded, but inline comments round-trip.

```
record OffsetDefinition(int Address, OffsetType Type, int Size, string? Comment = null)
```

### Comment parsing

After stripping the comment for field parsing, save the remainder:

```python
commentIdx = line.IndexOf('#')
if commentIdx >= 0:
    comment = line[(commentIdx+1)..].Trim()
    line = line[..commentIdx].TrimEnd()
else:
    comment = null
```

This preserves the original comment exactly (no re-formatting).

### Comment display: dynamic-width column

Spectre.Console `TableColumn` supports `Width` or no width. For the comment column:
- Set `Width = null` (auto) on the Comment `TableColumn`
- The first 5 columns have fixed widths (2 + 8 + 8 + 4 + 30 = 52)
- The Comment column fills remaining terminal width
- Long comments are auto-truncated by Spectre's cell overflow behavior
- Text is `EscapeMarkup()`'d before rendering

```csharp
.AddColumn(new TableColumn("Comment"));  // no Width → auto
```

Risk: at very narrow terminals, the comment column may get 0 width. This is acceptable — comments simply don't appear, no crash.

### Add/Edit comment prompt

A simple `TextPrompt<string?>` with `AllowEmpty()`:
- Add: default is empty → `null`
- Edit: default is existing comment, use `.DefaultValue()`; clearing → `null`

### WriteFile comment format

```csharp
if (!string.IsNullOrEmpty(def.Comment))
    writer.Write($"  # {def.Comment}");
```

This produces: `0x290C,u32  # number of hot joystick button slots`

### Keybinding: J / C pair, remove S

| Key | Action | File pattern |
|---|---|---|
| `J` | Save JSON snapshot | `fsuipc-snapshot-{yyyyMMdd-HHmmss}-{hostname}.json` |
| `C` | Save CSV snapshot | `fsuipc-snapshot-{yyyyMMdd-HHmmss}-{hostname}.csv` |

The `Save` case in the main loop becomes two cases sharing identical structure except format.

### CSV escaping rules

```
if value is null:
    write empty cell
else if value is byte[]:
    write Convert.ToHexString(value)
else if value is string s:
    if s contains ',' or '"':
        write $"\"{s.Replace("\"", "\"\"")}\""
    else:
        write s
else:
    write value.ToString()
```

### Help panel

Replace `s` entry with two entries:
```
[j]    Save JSON snapshot
[c]    Save CSV snapshot
```

## Risks / Trade-offs

- **[Narrow terminals]** At widths below ~60 chars, the comment column disappears. The Value column is also affected. Acceptable — the tool is designed for a reasonable terminal width.
- **[Comment character in values]** The parser uses first `#` on the line. If a future type or value format contained `#`, it would be misinterpreted as a comment start. Not currently a concern since types and values come from FSUIPC, not user input.
- **[Edit flow verbosity]** Adding a 4th prompt to the Edit flow (addr → type → size → comment) makes it longer. Tradeoff: completeness over brevity.

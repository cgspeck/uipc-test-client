## 1. Refactor row generation

- [x] 1.1 Extract per-row table cell logic from `CreateTable` loop into `AddRow(Table, RegisteredOffset, int index, int selectedIndex)` — preserves current isSelected/reverse behavior exactly
- [x] 1.2 Wire `AddRow` into the existing `CreateTable` loop, verify build passes and output is identical

## 2. Implement viewport computation

- [x] 2.1 Implement `GetAvailableHeight(int totalRows, bool hasError)` → `int` — computes `Console.WindowHeight - overhead` where overhead accounts for: column header (1), table borders (2), status line (1), error line (0/1), scroll indicator lines (0/1/2)
- [x] 2.2 Implement `ComputeWindow(int total, int selected, int viewportHeight)` → `(int start, int end)` — centered-with-clamp algorithm from design.md
- [x] 2.3 Implement `BuildScrollIndicator(bool isUp, int startOrEnd, int total)` → `IRenderable` — returns `"[dim]▲  (rows 1-19)"` style Markup, or `Text.Empty` if not needed

## 3. Rewrite CreateTable to BuildLayout

- [x] 3.1 Rename `CreateTable` → `BuildLayout`, change return type to `IRenderable`
- [x] 3.2 Build the table without `.Caption()` — plain windowed table
- [x] 3.3 Compose `Rows(scrollUp?, table, scrollDown?, statusLine, errorLine?)` using the new helpers
- [x] 3.4 Remove `.Overflow(VerticalOverflow.Ellipsis)` from the `Live` context in `RunLive`
- [x] 3.5 Verify build, format, and run

## 4. Implement immediate UpdateTarget on navigation

- [x] 4.1 In `RunLive`, after `HandleKey` returns `None` but the key was ↑/↓, call `ctx.UpdateTarget(BuildLayout(client, state))` immediately before the next tick
- [x] 4.2 Verify build passes, format clean, tests pass

## 5. Final verification

- [x] 5.1 Run `dotnet build` and `dotnet format`
- [x] 5.2 Run `dotnet test`
- [x] 5.3 Smoke-test with a large offset file: verify viewport centers, indicators appear, resize works, error line shows

## Why

When the offset table has more rows than fit in the terminal, Spectre.Console crops from the top — hiding the selected row and making navigation disorienting. The TUI needs a viewport that follows selection, with visual indicators for off-screen content.

## What Changes

- Extract per-row table cell generation into `AddRow()`
- Compute a window of visible rows centered on the selected index
- Add scroll-up / scroll-down indicator lines wrapped around the table via `Rows`
- Pull the status line out of the table caption into a standalone element below indicators
- Rename `CreateTable` to `BuildLayout`
- Trigger an immediate `UpdateTarget` on ↑/↓ for instant viewport response

## Capabilities

### New Capabilities

- `tui-table-viewport`: Scrolling viewport for the TUI offset table — windowed row rendering centered on selection, scroll indicators, and immediate update on navigation.

### Modified Capabilities

*(none)*

## Impact

- `TuiMode.cs`: Rewrites `CreateTable` → `BuildLayout`, splits row generation and viewport logic into helper functions, removes `.Overflow(VerticalOverflow.Ellipsis)` from the live context.
- No external dependencies or API changes.

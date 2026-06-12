## Context

`CreateTable` in `TuiMode.cs:369` renders all offset rows into a `Table`, set on `AnsiConsole.Live` with `VerticalOverflow.Ellipsis`. When the row count exceeds terminal height, Spectre.Console clips from the top — hiding the selected row and providing no visual feedback about overflow.

The existing layout uses the table's `.Caption()` for status/error text, which is rendered inside the table border. Scroll indicators need to live outside the border to be visually distinct.

## Goals / Non-Goals

**Goals:**
- Viewport follows the selected row (centered) so the active row is always visible
- Visual indicators for rows above/below the viewport when overflow exists
- Status line (and optional error line) rendered below all other content
- Immediate viewport re-center on ↑/↓ keypress
- Responsive to terminal resize on each render tick

**Non-Goals:**
- Mouse scroll support
- Custom horizontal scrolling
- Changing the Spectre.Console rendering pipeline

## Decisions

### Layout: Rows wrapper with standalone elements

The screen layout uses `Rows` to compose three or four independent renderables:

```
Rows(
    scrollUp?,       ← Markup, dim, only if start > 0
    Table,           ← windowed table (no caption)
    scrollDown?,     ← Markup, dim, only if end < total
    statusLine,      ← Markup, always
    errorLine?       ← Markup, only if state.LastError != null
)
```

**Rationale**: Pulling status/error out of the table caption lets scroll indicators sit between the table and the status line without clashing with the table border. `Rows` stacking is the simplest Spectre.Console-native approach.

### Window algorithm: centered with clamp

```
halfViewport = Math.Max(1, viewportRows / 2)
start = Math.Max(0, selectedIndex - halfViewport)
end = Math.Min(total, start + viewportRows)
start = Math.Max(0, end - viewportRows)     // re-clamp when hitting bottom boundary
```

The window is re-computed every render tick. `viewportRows` derives from `Console.WindowHeight` minus a fixed overhead budget (column header, borders) plus variable overhead (indicator lines, error line).

**Rationale**: Centering feels natural for keyboard navigation. Re-clamping at the end boundary prevents the window from showing fewer rows than `viewportRows` at the bottom.

### Immediate UpdateTarget on navigation

In `HandleKey`, when the action is `None` but the key was ↑/↓ (which only changes `SelectedIndex`), return a signal that causes `RunLive` to call `ctx.UpdateTarget()` immediately instead of waiting for the next tick.

**Rationale**: Hides the ~100ms latency between keypress and viewport re-center, making navigation feel instant.

### Dynamic viewport height

`viewportRows` = `Console.WindowHeight - overhead`
where `overhead` includes column header (1), top/bottom borders (2), status line (1), error line (0 or 1), and scroll indicator lines (0, 1, or 2).

Since indicator count varies as the selection approaches the list boundaries, the total visible data rows changes dynamically. This is acceptable — the tradeoff is simpler math with no wasted reserved space.

## Risks / Trade-offs

- **[Terminal resize]** Console.WindowHeight is read per-tick, so resizes are handled naturally. Risk: if `Live` uses an alternate buffer, `WindowHeight` may not track. → Mitigation: test on Windows Terminal / Conhost.
- **[Centering jitter]** At the top/bottom boundaries the viewport clamps, so the selection isn't perfectly centered. This is expected terminal-scroll behavior.
- **[Performance]** Building a smaller table each tick instead of a full table is actually *faster* — the concern is negligible.

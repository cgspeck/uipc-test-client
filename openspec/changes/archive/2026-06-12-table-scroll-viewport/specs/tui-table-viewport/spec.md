## ADDED Requirements

### Requirement: Viewport follows selected row

The TUI SHALL display a window of offset rows centered on the currently selected row. When the total row count fits within the viewport, all rows SHALL be shown without scrolling. When overflow occurs, the viewport SHALL re-center on selection changes.

#### Scenario: Viewport centers on selection
- **WHEN** the user navigates down through the offset list
- **THEN** the selected row SHALL remain visible and approximately centered in the viewport

#### Scenario: Viewport clamps at top boundary
- **WHEN** the selected row is near the top of the list (index < half viewport height)
- **THEN** the viewport SHALL start at row 0

#### Scenario: Viewport clamps at bottom boundary
- **WHEN** the selected row is near the bottom of the list
- **THEN** the viewport SHALL end at the last row

#### Scenario: Window fits all rows
- **WHEN** the total number of offset rows does not exceed the viewport capacity
- **THEN** all rows SHALL be visible without indicators

### Requirement: Scroll overflow indicators

When the viewport excludes rows above or below, the system SHALL display a visual indicator outside the table border.

#### Scenario: Indicator shown when rows above are hidden
- **WHEN** the viewport start index > 0
- **THEN** a scroll-up indicator SHALL be rendered above the table

#### Scenario: Indicator shown when rows below are hidden
- **WHEN** the viewport end index < total row count
- **THEN** a scroll-down indicator SHALL be rendered below the table

#### Scenario: Both indicators shown
- **WHEN** rows are hidden both above and below the viewport
- **THEN** both scroll-up and scroll-down indicators SHALL be rendered

#### Scenario: No indicators when all rows visible
- **WHEN** all offset rows fit in the viewport
- **THEN** no scroll indicators SHALL be rendered

### Requirement: Status line below scroll indicators

The status line SHALL be rendered as a standalone element below the table and any scroll-down indicator, separated from the table border.

#### Scenario: Status line always visible
- **WHEN** the layout is rendered
- **THEN** the status line SHALL appear as the last element, below the table and any scroll indicators

#### Scenario: Error line shown when present
- **WHEN** `state.LastError` is not null
- **THEN** an additional error line SHALL be rendered below the status line

### Requirement: Immediate viewport update on navigation

When the user presses ↑ or ↓, the viewport SHALL re-center on the next render cycle without waiting for the periodic refresh interval.

#### Scenario: Immediate re-center on keypress
- **WHEN** the user presses ↑ or ↓
- **THEN** `ctx.UpdateTarget` SHALL be called immediately to re-render the centered viewport

### Requirement: Terminal resize responsiveness

The viewport SHALL recalculate its capacity on each render tick based on the current terminal height.

#### Scenario: Viewport expands on terminal maximize
- **WHEN** the terminal window is made taller
- **THEN** additional offset rows SHALL become visible on the next render tick

#### Scenario: Viewport shrinks on terminal minimize
- **WHEN** the terminal window is made shorter
- **THEN** fewer offset rows SHALL be visible, with scroll indicators appearing as needed

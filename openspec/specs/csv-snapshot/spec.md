## Requirements

### Requirement: CSV snapshot keybinding

The TUI SHALL support saving a snapshot in CSV format via the `C` key, and JSON format via the `J` key. The existing `S` key SHALL be removed.

#### Scenario: C key saves CSV snapshot
- **GIVEN** the TUI is running
- **WHEN** the user presses `C`
- **THEN** a CSV snapshot SHALL be saved to disk

#### Scenario: J key saves JSON snapshot
- **GIVEN** the TUI is running
- **WHEN** the user presses `J`
- **THEN** a JSON snapshot SHALL be saved to disk (same format as previous `S` key)

### Requirement: CSV snapshot format

The CSV snapshot file SHALL contain a header row and data rows in `Offset,Value` format.

#### Scenario: Header row present
- **GIVEN** a CSV snapshot is saved
- **THEN** the first line SHALL be `Offset,Value`

#### Scenario: Numeric value
- **GIVEN** an offset with value `250`
- **WHEN** written to CSV
- **THEN** the line SHALL be `0x02BC,250`

#### Scenario: Null value
- **GIVEN** an offset with null value
- **WHEN** written to CSV
- **THEN** the value cell SHALL be empty: `0x02BC,`

#### Scenario: Byte array value
- **GIVEN** an offset with byte array value `[0xFF, 0xEE, 0xDD]`
- **WHEN** written to CSV
- **THEN** the value SHALL be hex: `0x0238,FFEEDD`

#### Scenario: String value with commas
- **GIVEN** a string offset with value `"N12345,ABC"`
- **WHEN** written to CSV
- **THEN** the value SHALL be quoted: `0x313C,"N12345,ABC"`

#### Scenario: String value without commas
- **GIVEN** a string offset with value `"B737"`
- **WHEN** written to CSV
- **THEN** the value SHALL be unquoted: `0x3160,B737`

### Requirement: CSV filename convention

The CSV snapshot filename SHALL follow the same pattern as JSON snapshots.

#### Scenario: CSV filename
- **GIVEN** the current date is 2026-06-12, hostname is "MYPC"
- **WHEN** a CSV snapshot is saved
- **THEN** the filename SHALL be `fsuipc-snapshot-20260612-143022-MYPC.csv`

### Requirement: Help panel includes j/c keybindings

The help panel SHALL reflect the new keybindings and remove the old `s` entry.

#### Scenario: Help lists J and C
- **GIVEN** the help panel is displayed
- **THEN** it SHALL show `j` for JSON snapshot and `c` for CSV snapshot
- **AND** it SHALL NOT show `s` for save

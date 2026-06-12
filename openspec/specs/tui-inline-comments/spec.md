## Requirements

### Requirement: Comment field on OffsetDefinition

`OffsetDefinition` SHALL include an optional `Comment` string field.

#### Scenario: Parsed from CSV inline comment
- **GIVEN** a CSV line `0x290C,u32  # number of hot joystick button slots`
- **WHEN** parsed by `OffsetParser.Parse()`
- **THEN** the resulting `OffsetDefinition` SHALL have `Comment = "number of hot joystick button slots"`

#### Scenario: No comment when absent
- **GIVEN** a CSV line `0x02BC,i32`
- **WHEN** parsed
- **THEN** `Comment` SHALL be `null`

#### Scenario: Comment preserved on WriteFile
- **GIVEN** an `OffsetDefinition` with `Comment = "airspeed"`
- **WHEN** written via `OffsetParser.WriteFile()`
- **THEN** the output line SHALL end with `  # airspeed`

### Requirement: Table displays comment column

The TUI offset table SHALL include a 6th column "Comment" rendered to the right of "Value".

#### Scenario: Comment column shown
- **GIVEN** a parsed offset with a comment
- **WHEN** the table is rendered
- **THEN** the comment text SHALL appear in the Comment column

#### Scenario: Comment column width is dynamic
- **GIVEN** the terminal width
- **WHEN** the table is rendered
- **THEN** the Comment column SHALL take the remaining width after the other 5 columns

#### Scenario: Comment truncated by column width
- **GIVEN** a comment longer than the column width
- **WHEN** rendered
- **THEN** the comment SHALL be truncated with `...`

#### Scenario: Empty comment cell
- **GIVEN** a parsed offset with no comment
- **WHEN** rendered
- **THEN** the Comment cell SHALL be empty

#### Scenario: Selected row shows all columns in reverse
- **GIVEN** a selected row with a comment
- **WHEN** rendered
- **THEN** the comment SHALL use `[reverse]` markup like other cells

#### Scenario: Comment text is markup-escaped
- **GIVEN** a comment containing Spectre markup characters like `[` or `]`
- **WHEN** rendered
- **THEN** the text SHALL be `EscapeMarkup()`'d to prevent rendering errors

### Requirement: Add flow includes comment prompt

The Add Offset prompt SHALL include a comment text entry after the size prompt.

#### Scenario: Comment can be blank
- **GIVEN** the user is in the Add flow
- **WHEN** prompted for a comment
- **THEN** an empty response SHALL set `Comment = null`

#### Scenario: Comment is preserved on offset
- **GIVEN** the user enters "airspeed indicator" as comment
- **WHEN** the offset is created
- **THEN** the `OffsetDefinition.Comment` SHALL contain "airspeed indicator"

### Requirement: Edit flow preserves and can change comment

The Edit Offset prompt SHALL include the current comment as a default value, and the user can change or clear it.

#### Scenario: Comment pre-filled in edit
- **GIVEN** an offset with comment "airspeed"
- **WHEN** editing
- **THEN** the comment prompt SHALL show "airspeed" as the default

#### Scenario: Comment cleared on edit
- **GIVEN** an offset with comment "airspeed"
- **WHEN** the user clears the comment in the edit prompt
- **THEN** `Comment` SHALL be `null`

### Requirement: Comment change marks dirty

Changing an offset's comment SHALL set `TuiState.IsDirty = true`.

#### Scenario: Editing comment dirties state
- **GIVEN** a clean state
- **WHEN** an offset's comment is changed via Edit
- **THEN** `IsDirty` SHALL be `true`

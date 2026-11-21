# hashline-read-annotation Specification

## Purpose
Annotate file reads with one content-derived tag so later edits can prove they target the version the agent read, without altering the content lines.

## Requirements

### Requirement: File-anchored tag
After a file read, the plugin SHALL add exactly one tag line `[<relPath>#<TAG>]` where TAG is the hash of the file's raw content, computed from the file and never from the host's rendered output. No per-line hash SHALL be added.

#### Scenario: Plain read
- **WHEN** the agent reads a file
- **THEN** the output carries one tag line, TAG equals the hash of the raw file bytes, and the added overhead is under 64 characters regardless of file length

### Requirement: Line numbers stay file-relative
The plugin SHALL leave content lines untouched so OpenCode's native `N: ` numbering, at any offset, is the addressing the patch language consumes.

#### Scenario: Offset read
- **WHEN** a 500-line file is read at offset 200
- **THEN** the displayed line numbers are the true file lines

### Requirement: Fail-safe annotation
An unrecognised Read output shape SHALL pass through untouched rather than be corrupted.

#### Scenario: Unknown shape
- **WHEN** the read output does not match a known shape
- **THEN** it is returned unchanged

### Requirement: Configuration
The plugin SHALL read `@mrmm-labs/opencode-omp-hashline.jsonc` from the project root or `~/.config/opencode/`, the project file winning, with `enabled`, `exclude`, `maxFileSize`, `injectSystemPrompt` and `debug`.

#### Scenario: Excluded file
- **WHEN** a read targets a file matching `exclude` or larger than `maxFileSize`
- **THEN** no tag is added

### Requirement: System prompt grammar
When `injectSystemPrompt` is true, the plugin SHALL append a compact grammar reference to the system prompt through `experimental.chat.system.transform`.

#### Scenario: Brief style
- **WHEN** the default prompt style applies
- **THEN** the brief grammar is appended

## Gaps
- The system prompt style option (`promptStyle`) is described in the README only; its exact values are not captured here.
- The tag is a 4-hex hash, so collisions are possible; staleness detection is probabilistic.
- Coexistence with the unrelated `opencode-hashline` package is a documented "remove it", not detected.

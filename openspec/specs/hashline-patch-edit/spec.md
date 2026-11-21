# hashline-patch-edit Specification

## Purpose
Apply multi-file line-addressed patches safely: verify every anchor first, write nothing on any mismatch, and roll back on write failure.

## Requirements

### Requirement: Patch tool
The plugin SHALL register a `hashline_patch` tool taking a `patch` string whose sections are anchored on the `[PATH#TAG]` from the most recent read.

#### Scenario: Fresh tag
- **WHEN** a section's tag matches the file's current hash and the patch is `SWAP 2.=2:`
- **THEN** line 2 is replaced and the result returns the resolved ranges and the fresh tag

### Requirement: Operations
The tool SHALL support `SWAP A.=B:`, `DEL A.=B`, `INS.PRE A:`, `INS.POST A:`, `INS.HEAD:` and `INS.TAIL:`, with body rows `+TEXT` added verbatim. Line numbers SHALL refer to the original file and never shift as hunks apply.

#### Scenario: Several hunks
- **WHEN** one patch swaps, inserts and deletes in a file
- **THEN** all line numbers refer to the original file

### Requirement: Stale anchors are rejected
A section whose tag does not match the file's current hash SHALL be rejected with a stale-anchor error and the file SHALL stay byte-identical.

#### Scenario: File changed since read
- **WHEN** the file was modified after the read that produced the tag
- **THEN** the edit is rejected and the file is unchanged

### Requirement: Atomic multi-section patches
Every section SHALL be verified before any file is written; one stale tag SHALL abort the whole batch naming the offending file. A write failure mid-batch SHALL restore files already written.

#### Scenario: One stale of two
- **WHEN** file A's tag is fresh and file B's is stale
- **THEN** neither file is modified and the error names B

### Requirement: Duplicate lines
A line SHALL be addressable by number even when its content occurs many times in the file.

#### Scenario: Repeated closing brace
- **WHEN** the target line's content occurs 15 times
- **THEN** only the targeted line changes

### Requirement: Line endings preserved
Unmodified lines SHALL keep their line endings; a CRLF file SHALL NOT convert to LF.

#### Scenario: CRLF file
- **WHEN** any operation is applied to a CRLF file
- **THEN** unmodified lines keep CRLF

### Requirement: Sandboxed paths
Paths resolving outside the project SHALL be refused.

#### Scenario: Path traversal
- **WHEN** a section's path resolves outside the project
- **THEN** the patch is refused before anything is written

## Gaps
- Block operations (`SWAP.BLK`, `DEL.BLK`, `INS.BLK.POST`) and file operations (`REM`, `MV`) are out of scope: they need the native addon.
- No three-way-merge recovery on stale anchors; rejection is the only behaviour.
- Latency and install-size targets in .specs/hashline/spec.md are not asserted by tests.

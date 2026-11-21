# release-hygiene Specification

## Purpose
Derive independent per-package versions from commit history and refuse malformed versions, changelogs or tags before they land.

## Requirements

### Requirement: Independent package versions
Each package SHALL version independently, every release SHALL target exactly one package, and tags SHALL be `<package>@<version>`. Bare `vX.Y.Z` tags SHALL be rejected by CI and the pre-push hook.

#### Scenario: Bare tag
- **WHEN** a tag `v1.2.3` is pushed
- **THEN** the tag-format job fails

### Requirement: Automatic release
Every push to main SHALL be evaluated for a release: `feat` bumps minor, `fix`, `perf` and `revert` bump patch, a breaking change bumps major (minor while 0.x), and `docs`, `style`, `refactor`, `test`, `build`, `ci` and `chore` bump nothing. The package that moves SHALL be decided by the files a commit touched, not its scope.

#### Scenario: Scope lies
- **WHEN** a `fix(hashline):` commit edits only snapcompact files
- **THEN** snapcompact is released

#### Scenario: Chore
- **WHEN** a `chore:` commit lands
- **THEN** no package is released

### Requirement: Commit message gate
The commit-msg hook SHALL reject messages that are not Conventional Commits.

#### Scenario: Free text
- **WHEN** a commit message is not `<type>(<scope>): <subject>`
- **THEN** the commit is refused

### Requirement: Version hygiene gate
`bun run check:versions` SHALL enforce valid semver, a changelog entry for each package's current version, an `[Unreleased]` section, strictly descending changelog versions, existing workspace-internal dependency versions, and, with `--tags`, a matching tag per released version.

#### Scenario: Missing changelog entry
- **WHEN** a package version has no changelog entry
- **THEN** the gate exits non-zero

## Gaps
- The workflow also publishes to npmjs and GitHub Packages; publishing is not specified here.

# telemetry Specification

## Purpose
Local-first instrumentation with an optional OpenTelemetry bridge, answering whether a plugin is worth its standing cost, and never breaking its caller.

## Requirements

### Requirement: Sinks
The package SHALL support sinks `file` (default, JSONL under `$XDG_STATE_HOME/opencode-omp/`), `otel` (through `@opentelemetry/api`, an optional peer imported lazily) and `none`, selectable as a string or array.

#### Scenario: Default
- **WHEN** no sink is configured
- **THEN** records are appended as JSONL to a local file

#### Scenario: OTel without an SDK
- **WHEN** `otel` is selected and the host registers no MeterProvider
- **THEN** nothing is sent and nothing fails

### Requirement: No network by default
The default path SHALL contain no network primitives, and no package SHALL declare an HTTP client dependency. Tests enforce both.

#### Scenario: Source scan
- **WHEN** the tests grep every shipped source file
- **THEN** no `fetch`, `node:http` or socket use is found

### Requirement: Instruments
The package SHALL provide `count`, `histogram`, `timer` and `event`. Event payloads SHALL reach the file sink only.

#### Scenario: Event with otel sink
- **WHEN** an event is recorded with only the otel sink
- **THEN** the payload is not emitted

### Requirement: Failure isolation
Every public method SHALL swallow its own errors so an unwritable sink, a circular payload or a missing SDK degrades to a no-op.

#### Scenario: Unwritable file
- **WHEN** the sink file cannot be written
- **THEN** the caller is unaffected

### Requirement: Config tolerance
Unknown keys and out-of-range values SHALL be dropped rather than applied. `"telemetry": false` SHALL disable it.

#### Scenario: Bad value
- **WHEN** `maxBytes` is out of range
- **THEN** the default applies

### Requirement: Rotation and privacy
The file SHALL rotate past `maxBytes` (default 5000000). With `redactPaths`, path-like attributes SHALL be replaced by a stable 8-character hash.

#### Scenario: Redaction
- **WHEN** `redactPaths` is true
- **THEN** the same path always yields the same 8-character hash

### Requirement: Report CLI
The `omp-telemetry` CLI SHALL offer counters, percentiles and gauges, a `verdict` subcommand weighing standing cost against realised gain with a confidence level that tracks sample size, and `watch`; every subcommand SHALL accept `--json`.

#### Scenario: Thin data
- **WHEN** the sample is small
- **THEN** the verdict says confidence is none and recommends no decision

## Gaps
- Token counts are measured only when the optional `js-tiktoken` peer is present; otherwise estimated at 3.6 characters per token.
- The verdict's exact formula lives in packages/telemetry/src/analyse.ts.

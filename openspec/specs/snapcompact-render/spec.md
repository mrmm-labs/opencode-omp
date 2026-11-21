# snapcompact-render Specification

## Purpose
Render dense text into bitmap frames a vision model can read back, only when that costs fewer tokens than sending the text.

## Requirements

### Requirement: Off by default
The package SHALL be disabled unless `enabled` is true, because its savings are conditional on content density.

#### Scenario: Not configured
- **WHEN** no config enables it
- **THEN** no render happens

### Requirement: Density gate
Rendering SHALL proceed only when the text's density (characters per token) is below the target model's image rate times one minus `densityMargin`; otherwise the text SHALL be returned unchanged with reason `not-dense-enough`.

#### Scenario: Prose
- **WHEN** the text is token-sparse prose
- **THEN** it is declined with `not-dense-enough`

#### Scenario: JSON
- **WHEN** the text is dense JSON on an Anthropic model
- **THEN** it is rendered

### Requirement: Real tokenizer
Density SHALL be computed with a BPE tokenizer, never a characters-per-four heuristic.

#### Scenario: Density
- **WHEN** density is computed for any input
- **THEN** the token count comes from the tokenizer

### Requirement: Savings are reported
A completed render SHALL state `textTokens`, `imageTokens` and `savingPct`, computed from the actual frame count and shape.

#### Scenario: Render result
- **WHEN** a render completes
- **THEN** the three figures are present

### Requirement: Provider shapes
The shape SHALL be resolved from the model id, not only the wire API, so a Claude model through a gateway keeps the anthropic shape. Shape geometry SHALL NOT be altered.

#### Scenario: Gateway
- **WHEN** a Claude model is routed through Vertex or OpenRouter
- **THEN** the anthropic shape is used

### Requirement: Valid attachable image
Each frame SHALL be a valid PNG exposed as `{type:"file", mime:"image/png", url:"data:image/png;base64,..."}`.

#### Scenario: Frame
- **WHEN** a frame is produced
- **THEN** it starts with the PNG magic bytes

### Requirement: Frame budget
Frames SHALL be capped at the provider budget (or `maxFrames`) and the overflow reported.

#### Scenario: Too much text
- **WHEN** the text needs more frames than the budget
- **THEN** frames are capped and the overflow is reported

### Requirement: Non-vision models refuse
A model without vision capability SHALL be declined with `model-not-vision-capable` and no frames produced.

#### Scenario: Text-only model
- **WHEN** compaction targets a non-vision model
- **THEN** it declines

### Requirement: Tools
The plugin SHALL register `snapcompact_estimate` (dry run reporting density and whether framing pays) and `snapcompact_render` (args `text`, `paths`, `force`; `force` bypasses the gate but still reports the loss).

#### Scenario: Force
- **WHEN** `force` is true on sparse text
- **THEN** frames are produced and the loss is reported

## Gaps
- The `auto-compact` mode (hook `experimental.chat.messages.transform`) is unverified at runtime and defaults off; not specified further here.
- `minChars` and `shapeOverride` exist in config but have no scenarios yet.

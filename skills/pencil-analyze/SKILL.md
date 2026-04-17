---
name: pencil-analyze
description: Extract a semantic design brief from an existing Pencil (.pen) file and persist it as `design.md` at the project root. Documents the visual language (atmosphere, palette roles, geometry, depth, typography) plus token vocabulary and scale parity - so every subsequent /pencil-design and /pencil-to-code pass shares the same stable brief instead of re-deriving one per session.
when_to_use: Before starting a new feature in an existing Pencil project, when onboarding a collaborator, when refreshing design.md after a significant design refactor, or when /pencil-design passes drift in vocabulary between sessions.
argument-hint: "[path-to.pen]"
disable-model-invocation: true
allowed-tools: mcp__pencil__get_editor_state mcp__pencil__get_guidelines mcp__pencil__get_variables mcp__pencil__batch_get mcp__pencil__search_all_unique_properties mcp__pencil__get_screenshot
model: sonnet
effort: high
---

# /pencil-analyze

Manual command that reads an existing Pencil (`.pen`) file, characterizes its visual language, and writes a `design.md` to the project root. The persisted brief then seeds every future `/pencil-design` and `/pencil-to-code` invocation with a stable vocabulary.

## Why this exists

`/pencil-design` captures a semantic brief in-context at the start of each authoring pass. That works for a single session, but the brief evaporates between sessions - two features designed a week apart can end up with subtly divergent atmospheres, mismatched radii slices, or two different tokens for the same role.

`/pencil-analyze` closes that loop. Run once after initial design work stabilizes. Re-run after significant refactors. The resulting `design.md` is the project's *shared* brief, consulted by every later pass.

## Relationship to other plugin skills

- **`/pencil-design`**: reads `design.md` on Step A (if present) and uses it as the canonical semantic brief instead of capturing a new one.
- **`/pencil-audit`**: orthogonal - audits numeric/variable divergences, not semantic drift. `/pencil-analyze` detects semantic drift (two "Cards" with different geometry slices) that audit cannot catch.
- **`/pencil-to-code`**: reads `design.md` to honor semantic role names (`brand`, `surface`, `muted`) in emitted code, rather than inventing generic class/variable names.

## Workflow

### Step 1 - Dispatch the analyzer subagent

Spawn `pencil-analyzer` via the `Task` tool. Pass the target `.pen` path (or allow the subagent to resolve it from the active editor state) and, if relevant, the project root where `design.md` will live.

```
subagent_type: pencil-analyzer
prompt: "Analyze the active Pencil file and return the structured brief. Project root: <cwd>. If the file is unopened or ambiguous, resolve via get_editor_state. Do not write design.md - I will."
```

The subagent returns a single structured block: `PROJECT / ATMOSPHERE / PALETTE / GEOMETRY / DEPTH / TYPOGRAPHY / SCALE_PARITY / TOKENS / COMPONENTS / VIEWPORTS / CONSISTENCY_SIGNALS`. Shape documented in `agents/pencil-analyzer.md`.

### Step 2 - Translate the structured block into `design.md`

Compose `design.md` at the project root following the canonical structure below. Use the subagent's block as source; do not paraphrase aggressively - preserve tokens, hex codes, pixel values, and component names verbatim.

Write with a design-lead voice: descriptive but not flowery. Role-named colors with hex in parentheses. Pixel values named by intent ("card interior padding 24 px" rather than "24").

### Step 3 - Handle consistency warnings

If `CONSISTENCY_SIGNALS.coherent: false`, append a `## Known Divergences` section at the end of `design.md` listing each warning, and recommend the user run `/pencil-audit` before acting on the brief.

Do **not** refuse to write `design.md` when there are warnings - a partially-coherent brief is still more useful than none. But make the divergences visible.

### Step 4 - Overwrite or merge

If `design.md` already exists:

- Read it first.
- If the existing file has user-authored prose sections not derivable from the analyzer block (e.g., a `## Product Principles` section with narrative content), preserve those sections verbatim and update only the auto-derived ones (`## Visual Theme`, `## Color Palette & Roles`, `## Typography`, `## Component Stylings`, `## Layout & Scale`, `## Token Vocabulary`).
- If the existing file is fully auto-derived, overwrite.
- Never silently delete human-authored content. If in doubt, diff in-place and ask.

### Step 5 - Report to the user

One line: file path, whether created or updated, count of roles/tokens/components/viewports captured, and whether consistency warnings were emitted.

```
Wrote design.md (12 roles, 27 tokens, 6 components, 4 viewports). 1 consistency warning - see bottom of file.
```

## Canonical `design.md` structure

Write exactly these sections, in this order. Example output in `examples/design.md`.

```markdown
# Design System: <Project title>

<Optional one-paragraph project description, written by a human or left blank on first pass>

## 1. Visual Theme & Atmosphere
<Narrative paragraph from ATMOSPHERE.adjectives + ATMOSPHERE.evidence>

**Key Characteristics:**
- <bulleted adjectives and their concrete consequences in the design>

## 2. Color Palette & Roles
<For each role from PALETTE: descriptive name + hex + functional description + used_in count>

### Unclear or Missing Tokens
<Only if PALETTE.unclear or PALETTE.missing are non-empty - list for action>

## 3. Typography
Primary family: <TYPOGRAPHY.primary_family>
Character: <TYPOGRAPHY.character>
Weight range: <TYPOGRAPHY.weights_in_use>
Font sizes in use: <list from TYPOGRAPHY.font_sizes_in_use>

### Hierarchy
<Narrative: which weight/size combinations serve which role - inferred from COMPONENTS>

## 4. Geometry & Depth
**Geometry character:** <GEOMETRY.character> - radii in use: <GEOMETRY.radii_in_use>
**Depth character:** <DEPTH.character> - shadows: <DEPTH.shadows_in_use>, borders: <DEPTH.borders_in_use>

## 5. Component Stylings
<For each COMPONENTS entry: name, role inferred from child_structure, reuse count>

## 6. Layout & Scale
**Spacing values in use:** <from SCALE_PARITY.spacing.values>
**Breakpoints designed:** <from VIEWPORTS - list device + breakpoint + width>

### Scale Parity
<SCALE_PARITY summary line per category; call out off-scale values if any>

## 7. Token Vocabulary
<TOKENS table: name | role | value | used_in count>

## 8. Known Divergences
<Only if CONSISTENCY_SIGNALS.coherent: false - itemized warnings + /pencil-audit recommendation>
```

## Honesty over aesthetics

`design.md` describes what the `.pen` *is*, not what it *should be*. A thin `.pen` produces a thin brief. A messy `.pen` produces a brief with divergences section. Do not inflate adjectives to make the design sound more intentional than it is - downstream skills will trust the brief and propagate the inflation.

If the `.pen` is clearly in early exploration and not ready for a persisted brief, say so in Step 5 output and skip writing `design.md`: `Analyzer reports sparse evidence (N frames); design.md not written. Run again after more design work.`

## Tone

Descriptive but grounded. Named colors with hex codes. Semantic role names over literal class names. No flowery filler. The output should read as a shared contract between designer, developer, and agent - not as marketing copy.

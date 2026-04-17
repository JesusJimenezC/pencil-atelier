---
name: pencil-design
description: Author Pencil (.pen) designs that are compliant by construction with the plugin's four house rules (variable-first tokens, component extraction at the second repetition, universal 4px-grid scale, grep-able viewport frame naming). Captures a semantic design brief before composition so on-scale choices match intent. Preventive counterpart to /pencil-audit.
when_to_use: The user asks to "design a component in Pencil", "create a frame", "lay out a page in .pen", "add a card to the library", or anything that produces or modifies `.pen` content.
argument-hint: "[feature-name-or-frame-description]"
disable-model-invocation: true
allowed-tools: mcp__pencil__get_editor_state mcp__pencil__get_guidelines mcp__pencil__get_variables mcp__pencil__set_variables mcp__pencil__batch_get mcp__pencil__batch_design mcp__pencil__search_all_unique_properties mcp__pencil__find_empty_space_on_canvas mcp__pencil__get_screenshot mcp__pencil__snapshot_layout mcp__pencil__open_document
model: opus
effort: xhigh
---

# /pencil-design

Author `.pen` designs that are already correct by construction. Every value emitted into a frame, every repeated structure, and every visual token follows the plugin's four house rules from the first stroke onward.

## Relationship to other plugin skills

- **`pencil-analyze` skill**: extracts a persisted semantic brief (`design.md`) from an existing `.pen`. If `design.md` exists at the project root, this skill reads it first (see Step A below) and uses it as the canonical brief - no in-session re-derivation.
- **`pencil-audit` skill**: reactive validation of an existing design. Run after editing if there is any doubt that the rules held.
- **`pencil-to-code` skill**: transpile a finished frame to code (Tailwind-aware if installed, raw pixels otherwise).

This skill is the **preventive** layer. It prevents divergences from entering the `.pen` file in the first place.

This skill focuses exclusively on _design-quality rules_. It relies on the Pencil MCP server (`mcp__pencil__*` tools - `batch_design`, `batch_get`, `get_variables`, `set_variables`, `get_editor_state`, `search_all_unique_properties`, `find_empty_space_on_canvas`, `get_screenshot`, etc.) being available in the session. It does not teach how those tools work; consult the server's own tool descriptions for argument shapes and the Pencil documentation for concepts such as components, refs, slots, variables, themes, and imports.

## MCP retrieval pipeline

Every `/pencil-design` invocation follows this fixed tool-call order. Skip a step only when the caller has already supplied the equivalent information in the prompt.

1. **`get_editor_state({ include_schema: false })`** - establish the active `.pen` file path and the current user selection. If no file is open, call `open_document('new')` or the explicit path the user named.
2. **`get_guidelines(category?, name?)`** - load any project-ships design guideline relevant to the brief (typography, color system, motion). Consult before inventing new conventions.
3. **`get_variables(filePath)`** - cache the declared token map (`$color-*`, `$spacing-*`, `$radius-*`, `$font-*`). This feeds both the Semantic brief (step below) and R1.
4. **`search_all_unique_properties(parents, properties)`** - on existing top-level frames, enumerate the values already in use for `fillColor`, `textColor`, `cornerRadius`, `padding`, `gap`, `fontSize`. Surface R1/R3 gaps **before** composing.
5. **Dispatch `pencil-navigator`** - when the brief references an existing feature by name use shape `find-by-feature`; when it references a token use `find-by-token`; before extracting a component use `find-repeated-structures` to confirm 2+ appearances across the file. Navigator returns IDs + breadcrumbs without flooding context.
6. **`find_empty_space_on_canvas({ width, height })`** (or navigator `find-empty-canvas`) - obtain non-overlapping coordinates for the new frame. Never silently overlap existing content.
7. **`set_variables(...)`** - if Step 3 revealed gaps that R1 will force you to fill (new brand token, new palette role), declare them **now** before any frame is inserted.
8. **`batch_design(operations)`** - compose the frame tree. One call per logical chunk, max 25 ops per call. Every numeric value must come from an R3 scale entry or a `$token`; every viewport name must follow R4.
9. **`get_screenshot(nodeId)`** - visually verify the root frame. Never declare the work done on the basis of the JSON mutation succeeding.

All further information (atmosphere capture, rule enforcement, anti-patterns) assumes these steps have been executed in order.

## Semantic brief (pre-composition)

Before choosing on-scale values, capture the brief's visual intent across five axes. This step exists because the scale in R3 is large (37 spacing values, 13 font sizes, 10 radii) and the right choice at each slot depends on the *character* of the design, not just on "is this value on the scale?". Without an explicit brief, the agent picks defensibly-correct but mood-incoherent values - small, technically-valid inconsistencies compound into visual disharmony.

Capture the brief either from the user's prompt or by asking one concise clarifying question. Do not skip this step even for small tasks - for a single card component, the brief is a one-line paragraph; for a full viewport, it is a short block.

### The five axes

1. **Atmosphere** - the overall mood. Use evocative adjectives: `airy`, `dense`, `editorial`, `utilitarian`, `playful`, `clinical`, `opulent`, `restrained`. Two or three adjectives is enough. Drives whitespace strategy (large section padding vs. tight gaps) and typographic scale (generous H1 72 vs. restrained H1 36).
2. **Palette roles** - list every color the design needs *by role*, not by hex. Roles: `brand`, `brand-bg`, `bg`, `surface`, `fg`, `muted`, `border`, `accent`, `success`, `warning`, `danger`. For each role, check `get_variables` output first; declare (`set_variables`) only what is missing. Emit hex only as the *value* of a token, never as a literal inside a frame.
3. **Geometry character** - how round are the edges? Choose one: `sharp` (0-2 px radii), `subtle` (4-6 px), `soft` (8-12 px), `generous` (16-24 px), `pill` (9999). This choice maps 1:1 to a slice of R3 allowed radii; all components in the same design should draw from the same slice unless the brief explicitly says otherwise.
4. **Depth character** - how does the UI stack layers? `flat` (no shadows, borders only), `whisper-soft` (shadow-sm, inner), `pronounced` (shadow-md, lg), `dramatic` (shadow-xl, 2xl). Use the shadow ramp declared in `references/tailwind-scale.md`.
5. **Typography character** - the voice of the text. `geometric` (Manrope, Inter, Outfit), `humanist` (Lora, Crimson), `industrial` (JetBrains Mono, IBM Plex Mono), `editorial` (mix: serif H1 + geometric body). Drives font-family selection and weight usage (body 400/500, heading 600/700).

### Output format (record inside the workflow, do not emit to the user)

```
Atmosphere: <2-3 adjectives>
Palette roles: brand=$brand(#HEX) bg=$bg(#HEX) fg=$fg(#HEX) muted=$muted(#HEX) [missing: border -> declare $border=#HEX]
Geometry: <one-of sharp/subtle/soft/generous/pill> -> radii from {allowed subset}
Depth: <one-of flat/whisper-soft/pronounced/dramatic> -> shadows from {allowed subset}
Typography: <one-of geometric/humanist/industrial/editorial> -> family=<name>, body=<weight>, heading=<weight>
```

Keep the record in your working context; it feeds every decision in R1-R4.

### How the brief binds the rules

- **R1 (variables)**: palette roles from axis 2 *become* tokens. Every color in every frame must cite a role-named token.
- **R2 (components)**: atmosphere influences naming of reusable components (editorial tone → `ArticleMeta`, `SectionHeading`; utilitarian → `Row`, `Item`).
- **R3 (scale)**: geometry/depth/typography axes pre-filter the scale to a coherent subset. Example: `soft` geometry + `whisper-soft` depth means every Card in the design uses radius 8 or 12 and shadow-sm; picking radius 16 or shadow-xl for one Card is now a brief violation even if the raw value is on-scale.
- **R4 (viewport names)**: no semantic brief input; the name pattern is deterministic from the viewport target.

## The four house rules

### R1 - Variable-first tokens

Every color, spacing value, radius, and font-size that appears more than once - or that represents a brand-level decision (brand colors, heading sizes, primary radii) - **must** be declared as a Pencil variable and referenced as `$token` inside frames. Literal hex colors and raw pixel values are only acceptable for truly one-off, local measurements.

Before inserting a new value, check:

1. `get_variables` - does a token with this exact value already exist? If yes, reference it.
2. If no and the value is a brand/system-level decision (likely to reuse), `set_variables` first, then reference the new token.
3. Only emit a raw literal if the value is genuinely one-off.

Typical token naming (adapt to the project's existing vocabulary - do not impose this scheme if the project already has one):

- Colors: `$brand`, `$brand-light`, `$brand-bg`, `$bg`, `$fg`, `$muted`, `$border`, `$success`, `$warning`, `$danger`, plus `$gray-100` ... `$gray-900`.
- Spacing: `$spacing-xs` (4), `$spacing-sm` (8), `$spacing-md` (16), `$spacing-lg` (24), `$spacing-xl` (32), `$spacing-2xl` (48), plus numeric aliases (`$spacing-2`=8, `$spacing-3`=12, etc.) when it helps match the scale below directly.
- Radii: `$radius-sm` (4), `$radius-md` (8), `$radius-lg` (12), `$radius-xl` (16), `$radius-2xl` (24), `$radius-full` (9999).
- Font sizes: `$font-xs` (12), `$font-sm` (14), `$font-base` (16), `$font-lg` (18), `$font-xl` (20), `$font-2xl` (24), `$font-3xl` (30), `$font-4xl` (36), `$font-5xl` (48), `$font-6xl` (60), `$font-7xl` (72).

Token values themselves still must come from the scale in R3 below.

### R2 - Component at the second repetition

If a structure is used **more than twice** (i.e. >= 2 repetitions - the user requested "two or more uses -> component"), it must be extracted into a reusable component:

- On first placement: build inline.
- On the **second** placement that is structurally identical (same child layout, same token vocabulary): stop, extract the first instance into a component origin (`reusable: true`) - or into a `.lib.pen` if one exists in the project - and replace both call sites with `ref` instances.
- On any further placement: insert a `ref` instance and override via `descendants` as needed.

Extraction checklist:

1. Identify the repeated subtree.
2. Create the component origin with a stable name (singular, noun: `Card`, `Pill`, `ArticleMeta`, `SectionHeading`).
3. Mark slots (empty frames with `slot: [...]`) for genuinely variable content.
4. Replace each existing placement with a `ref` pointing at the origin; use `descendants: { "<child-id>": { ... } }` only for per-instance overrides.
5. If the project has a `.lib.pen`, move the component origin there so other `.pen` files can consume it via `imports`.

Do **not** extract on first sight of a pattern - premature components generate churn. Wait for the second repetition to confirm.

Before extracting, dispatch `pencil-navigator` with shape `find-repeated-structures` to confirm the pattern appears in 2+ frames across the whole `.pen`. This guards against premature extraction driven by local-view bias and avoids missing a pre-existing component that already solves the same pattern.

### R3 - Universal 4px-grid sizing scale

Every numeric property on every frame must belong to the allowed scale. This rule is framework-agnostic; it is the same scale enforced by the `pencil-auditor` and consumed by `pencil-to-code`.

#### Canonical reference

**Authoritative spec**: `references/tailwind-scale.md` (sibling of this SKILL file).

Consult that file at the start of any design task to load the full scale (spacing, font sizes, font weights, line heights, letter spacing, border radius, border widths, opacity, box shadow, z-index, breakpoints, plus device labels and off-scale snap targets for every category). If the inline recap below diverges from the reference file, **the reference wins**.

#### Inline recap (fast-access summary)

- **Spacing (px)**: `0, 2, 4, 6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60, 64, 72, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 384`. Any multiple of 4 up to 384, plus fractional-derived 2, 6, 10, 14.
- **Font sizes (px)**: `12, 14, 16, 18, 20, 24, 30, 36, 48, 60, 72, 96, 128`. Legibility floor: 12.
- **Font weights**: `100, 200, 300, 400, 500, 600, 700, 800, 900`.
- **Line heights**: `1, 1.25, 1.375, 1.5, 1.625, 2`.
- **Letter spacing (em)**: `-0.05, -0.025, 0, 0.025, 0.05, 0.1`.
- **Border radius (px)**: `0, 2, 4, 6, 8, 12, 16, 24, 32, 9999`. Asymmetric arrays valid for single-side rounding.
- **Border widths (px)**: `0, 1, 2, 4, 8`.
- **Opacity**: multiples of 5 from 0 to 100 (percent).
- **Shadow levels**: `none, sm, default, md, lg, xl, 2xl, inner`.
- **Z-index**: `0, 10, 20, 30, 40, 50, auto`.
- **Breakpoint widths (px)**: `375, 640, 768, 1024, 1280, 1536`.

#### Picking on-scale frame widths

When the frame represents a full viewport (a screen), its width must be exactly one of the six allowed values above. Each value has a canonical Tailwind breakpoint token **and** a canonical semantic device label - both are used in the frame name (see R4 below):

| Device label | Breakpoint token | Width (px) | Typical target                      |
| ------------ | ---------------- | ---------- | ----------------------------------- |
| `mobile`     | `base`           | 375        | Phone portrait (iPhone baseline)    |
| `mobile-lg`  | `sm`             | 640        | Large phone / small tablet portrait |
| `tablet`     | `md`             | 768        | Tablet portrait (iPad baseline)     |
| `laptop`     | `lg`             | 1024       | Tablet landscape / small laptop     |
| `desktop`    | `xl`             | 1280       | Standard desktop / laptop           |
| `desktop-lg` | `2xl`            | 1536       | Large desktop / wide monitor        |

These three descriptors (device, breakpoint, width) form the viewport frame name - see R4 below.

#### Picking on-scale values for new designs

When choosing a value, pick the one from the scale that best expresses the intent:

- Tight inline gap: 4, 6, 8.
- Card interior padding: 16, 20, 24.
- Section padding: 32, 48, 64.
- Page margins: 16 (mobile), 32 (tablet), 48 - 176 (desktop).
- Body text: 14 or 16. Secondary: 12 or 14. H3: 20 or 24. H2: 30 or 36. H1: 48 - 72.
- Card radius: 12 or 16. Pill / avatar: 9999. Input: 6 or 8.
- Line-height: 1.25 for headings, 1.5 for body, 1.625 for comfortable reading.

If a candidate value falls between two allowed ones, prefer the smaller (denser compositions are easier to loosen later than to tighten).

### R4 - Descriptive, grep-able frame names

Every frame that represents a **full viewport** (a screen / page layout) must carry a name that encodes three things: the feature it implements, the semantic device class it targets, and the exact Tailwind breakpoint it corresponds to. The pattern is trivial to grep both inside the `.pen` JSON and across any tooling that indexes the file.

#### Pattern

```
<PascalFeatureName>__<device>-<bp>-<width>
```

- `<PascalFeatureName>` - the feature or page in PascalCase. One noun phrase: `Home`, `BlogIndex`, `ArticleDetail`, `Pricing`, `Settings`, `CheckoutReview`. Avoid generics like `Screen1`, `Layout`, `Page`.
- `<device>` - semantic device class from the table in R3. Allowed set: `mobile`, `mobile-lg`, `tablet`, `laptop`, `desktop`, `desktop-lg`. Gives humans (and greps for "tablet" or "desktop") an immediate answer to _"which form factor is this?"_ without having to translate breakpoint tokens.
- `<bp>` - Tailwind breakpoint token bound to `<device>`. Allowed set: `base`, `sm`, `md`, `lg`, `xl`, `2xl`. `base` is the mobile default (375 px) for which Tailwind emits no prefix.
- `<width>` - the literal frame width in pixels. Must match the device/breakpoint's canonical value from the R3 table.

Double underscore (`__`) separates the feature from the descriptor block; single hyphen (`-`) separates the three descriptors inside that block. These choices are deliberate: `__` rarely appears in prose or tokens, and anchoring numeric greps on `-<width>` at the end of the name remains unambiguous.

#### The six allowed descriptor blocks

Each pairing is fixed - a viewport frame **must** use one of these exact suffixes, never a partial or reordered version:

- `__mobile-base-375`
- `__mobile-lg-sm-640`
- `__tablet-md-768`
- `__laptop-lg-1024`
- `__desktop-xl-1280`
- `__desktop-lg-2xl-1536`

#### Examples

- `Home__mobile-base-375`
- `Home__tablet-md-768`
- `Home__desktop-xl-1280`
- `BlogIndex__mobile-base-375`
- `ArticleDetail__laptop-lg-1024`
- `Pricing__desktop-lg-2xl-1536`
- `CheckoutReview__mobile-lg-sm-640`

#### Grep cookbook

Once frame names follow R4, Claude Code (or any engineer) can locate the exact design to edit in a single command. Queries can target the feature, the device class, the breakpoint token, or the exact width - each descriptor is independently searchable:

- All viewports of a feature: `grep -n "Home__" design/*.pen`
- All tablet viewports (semantic): `grep -n "__tablet-" design/*.pen`
- All desktop-class viewports (both `desktop` and `desktop-lg`): `grep -nE "__desktop(-lg)?-" design/*.pen`
- All mobile-class viewports (both `mobile` and `mobile-lg`): `grep -nE "__mobile(-lg)?-" design/*.pen`
- Exact device + breakpoint: `grep -n "__laptop-lg-1024" design/*.pen`
- By Tailwind breakpoint token only (anchored by width): `grep -nE "-sm-640$"` or `grep -nE "-xl-1280$"` etc.
- By exact width: `grep -n "-1280$" design/*.pen`
- All viewport frames (vs. components): `grep -nE "__[a-z-]+-(base|sm|md|lg|xl|2xl)-\d+$" design/*.pen`

Note the `$` anchor in breakpoint-token greps: without it, the `-lg-` inside `mobile-lg` or `desktop-lg` could false-positive against the `lg` breakpoint token. Always anchor breakpoint greps on `-<width>` at the end of the name.

#### What R4 does NOT apply to

Non-viewport frames use their role name directly and **never** carry the `__bp-width` suffix:

- Components (origins and refs): `Card`, `Pill`, `NavItem`, `ArticleMeta`, `SectionHeading`, `PricingTier`.
- Variants of a component: `Button__primary`, `Button__secondary`, `Button__ghost` - double underscore here separates role from variant, which is fine since the suffix is not a breakpoint token.
- Loose utilities / icons / spacers: their semantic name.

This separation means a grep for `__(base|sm|md|lg|xl|2xl)-` cleanly isolates viewport entry-points from every other node in the file.

#### Layout discipline

Keep every viewport variant of a single feature as sibling frames at the same canvas level, sharing the PascalCase prefix. A feature that targets every breakpoint should appear as up to six siblings:

- `Home__mobile-base-375`
- `Home__mobile-lg-sm-640`
- `Home__tablet-md-768`
- `Home__laptop-lg-1024`
- `Home__desktop-xl-1280`
- `Home__desktop-lg-2xl-1536`

Not every feature needs all six - design only the breakpoints the project actually targets. If only three are designed (mobile + tablet + desktop, say), use exactly those three device/breakpoint pairings from the R3 table; do not invent custom intermediate widths.

## Workflow

The workflow is the choreography that binds the MCP pipeline, the Semantic brief, and the four house rules into a single authoring pass. Each step maps to specific pipeline calls (numbered as in the "MCP retrieval pipeline" section above).

### Step A - Understand the brief (pipeline: 1, 2)

Clarify: what is being designed (component? page? variant?), what theme modes apply, what breakpoints matter, whether a `.lib.pen` exists. Use `get_editor_state` for the active file, `get_guidelines` for project conventions.

**Check for `design.md` at the project root.** If present, read it. It is the project's persisted semantic brief (produced by `/pencil-analyze`). Treat its atmosphere, palette roles, geometry, depth, and typography sections as *binding* inputs to Step B - you are not authoring a new brief, you are extending an existing one. If the user's prompt explicitly overrides an axis ("use a more playful atmosphere for this campaign page"), honor the override and note it in your internal record; otherwise the `design.md` values stand.

### Step B - Capture (or extend) the semantic brief (no tool calls)

Walk through the five axes in the "Semantic brief" section above (atmosphere, palette roles, geometry, depth, typography). If `design.md` existed in Step A, start from its values and only record axes that are new or user-overridden for this task. If `design.md` did not exist, produce the full compact record from scratch. This is the single most important step for design coherence - do not skip it.

### Step C - Pre-flight tokens and existing values (pipeline: 3, 4, 7)

Call `get_variables` to cache the token map, then `search_all_unique_properties` on existing top-level frames to discover which values (colors, spacings, radii) are already in use and whether they match the brief's palette-roles axis. Declare missing tokens with `set_variables` **before** inserting any frame.

If the brief references an existing feature, viewport, or token, dispatch `pencil-navigator` (pipeline step 5) with the appropriate shape instead of running `batch_get` yourself.

### Step D - Locate canvas space (pipeline: 6)

Dispatch `pencil-navigator` with shape `find-empty-canvas`, or call `find_empty_space_on_canvas` directly, supplying the required width and height. Never overlap existing content silently.

### Step E - Compose with `batch_design` (pipeline: 8)

Author the frame tree. While doing so:

- Every color, radius, font-size, spacing value is either a `$token` (R1) or a raw value from the R3 scale, pre-filtered by the brief's geometry/depth/typography axes. If you catch yourself typing `padding: 17`, stop - pick 16 or 20.
- If you find yourself duplicating the same subtree (cards, pills, meta rows, list items) twice inside a single batch, collapse them immediately into a component origin + refs (R2).
- Every top-level viewport frame is named per R4 (`<PascalFeatureName>__<device>-<bp>-<width>`). Name as you insert - renaming later is cheap, but grep-ability matters from the first commit.
- Prefer flexbox (`layout: "flex"`, `gap`, `padding`) over absolute positioning. Flex gaps and paddings come from the spacing scale.

### Step F - Verify visually (pipeline: 9)

Call `get_screenshot` on the root frame. Confirm the result matches the *brief* visually, not only the JSON structurally. If atmosphere or geometry does not match the brief, iterate inside `batch_design` before declaring done.

### Step G - Self-check against the four rules

Before handing off, walk through:

- [ ] R1 - every repeated color / spacing / radius / font-size references a token; raw literals only for one-off values.
- [ ] R2 - no structure appears inline 2+ times; everything repeated is a component + refs.
- [ ] R3 - every numeric property belongs to the scale, and values within one design draw from the geometry/depth/typography subsets chosen in the brief.
- [ ] R4 - every viewport frame is named `<PascalFeatureName>__<device>-<bp>-<width>` using one of the six canonical descriptor blocks (`mobile-base-375`, `mobile-lg-sm-640`, `tablet-md-768`, `laptop-lg-1024`, `desktop-xl-1280`, `desktop-lg-2xl-1536`). Device label and breakpoint token must match the pairing from the R3 table; width must be the canonical pixel value.
- [ ] Brief - every visual choice traces back to one of the five axes. If a choice has no brief justification, re-examine it.

If any box is unchecked, fix the `.pen` before returning control to the user. If uncertain, run `/pencil-audit` - it performs R1/R2/R3 checks independently and will surface anything missed.

## Safe-apply discipline (mandatory for `$`-values)

Every write path that stores a variable reference (R1 promotion, token fix, corruption repair, ad-hoc refactor, bulk rename) MUST go through `batch_design` with explicit `U(nodeId, { prop: "$token" })` operations. This is not a recommendation — it prevents a silent data-corruption class that previously ruined an entire production `.pen` file.

### Rule 1 — Never use `replace_all_matching_properties` for `$`-values

`mcp__pencil__replace_all_matching_properties` escapes `$` into `\$` internally when the `to:` value is a variable reference. The stored string becomes `\$brand` (literal backslash + `$` + `brand`), which does NOT resolve to the declared `$brand` token. Renderers treat it as an unknown value and fall back to defaults (usually black). The tool reports success and no error surfaces until a human opens the design and sees a black screen.

This is the exact corruption that R5 in `/pencil-audit` detects. Do not use the tool at all for `$`-values. It is a footgun without a safe mode for this input class.

Allowed uses of `replace_all_matching_properties`: raw-value to raw-value rewrites (e.g., `#EA580C` → `#DC2626`). Forbidden: anything where the target starts with `$`.

### Rule 2 — All `$`-writes go through `batch_design` U() ops

`batch_design` preserves `$` literally — the token is stored as-is and resolves at render time. Use it for every write that touches a variable reference, whether you are promoting, repairing, or renaming.

```
batch_design([
  U("frameA", { fill: "$brand", cornerRadius: "$radius-md" }),
  U("frameB", { padding: ["$spacing-md", "$spacing-lg", "$spacing-md", "$spacing-lg"] }),
])
```

For bulk promotion (the legitimate use-case that historically invited `replace_all_matching_properties`), enumerate the target nodes via `search_all_unique_properties` + `batch_get`, then emit one `U()` per node in batches of ≤25 operations per `batch_design` call. The extra verbosity is the price of correctness.

### Rule 3 — Nested properties take full objects, not dot-paths

Pencil's nested shapes (`stroke`, `effects[*].shadow`, multi-layer fills) require the full object in the update. Do not use dot-path keys:

```
❌ U("frameA", { "stroke.fill": "$gray-200", "stroke.thickness": 1 })   // silently no-op on some shapes
✅ U("frameA", { stroke: { fill: "$gray-200", thickness: 1 } })          // correct
```

When repairing a nested property that was previously set, read the existing nested object first (via `batch_get`) and pass it back with the corrected field — overwriting with a partial object can drop sibling fields.

### Rule 4 — Post-apply validation is non-negotiable

A `batch_design` call that returns `success: true` proves only that the API accepted the operation, not that the design renders correctly. After every batch that touches `$`-values:

1. Re-run `batch_get` on the written nodes. Scan the returned JSON for any value starting with `\$`. If any remain, they leaked past the write — stop and investigate before writing more.
2. Call `get_screenshot` on the root frame(s) touched. Compare against the brief. A clean JSON that renders black is a failure, not a success.

Silent failure is the default mode of this class of bug. Treat unvalidated writes as untested writes.

## Anti-patterns to refuse

- Emitting `#EA580C` directly when `$brand` exists with that value. Reference the token.
- Choosing `padding: 15` because the brief said "about 15 pixels". Snap to 16 (or 14 if compact).
- Inserting a third `Card` subtree inline because "extraction can happen later". Extract on the second appearance.
- Creating a component named `ComponentCard01` or similarly generic. Names are nouns describing role (`ArticleCard`, `PricingTier`, `NavItem`).
- Introducing a radius of 20 because "it looks nicer". The allowed set is `{0, 2, 4, 6, 8, 12, 16, 24, 32, 9999}` - pick 16 or 24 and move on.
- Emitting `font-size: 17`. Pick 16 or 18.
- Treating theme modes (light/dark) as optional. If the project has a `mode: ["light","dark"]` axis, every `$color-*` token must supply both values.
- Naming a viewport frame `Home Mobile`, `home-mobile`, `Screen1`, `Landing Desktop 1440`, or any other ad-hoc scheme. Use `Home__mobile-base-375`, `Home__desktop-xl-1280`, etc., exactly. Ad-hoc names defeat the R4 grep cookbook and force every handoff to start with "where is the file I'm supposed to edit".
- Mixing device label and breakpoint token from different rows of the R3 table (e.g., `Home__tablet-lg-1024` - `tablet` belongs to `md-768`, not `lg-1024`; the correct name is `Home__laptop-lg-1024` or `Home__tablet-md-768`). Device <-> breakpoint pairings are fixed by the R3 table; do not recombine.
- Omitting either the device label or the breakpoint token ("just one or the other is enough"). Both are required - the device label answers the human question ("which form factor?"), the breakpoint token answers the engineering question ("which Tailwind prefix?"). Dropping either cripples the grep cookbook.
- Using a width that does not belong to the six canonical breakpoint values (e.g., a `lg` frame at 1100 px instead of 1024). If you need a width between two breakpoints, pick the lower one and design to it - breakpoints are thresholds, not absolutes.
- Calling `replace_all_matching_properties` to promote a raw value to a variable reference ("this is faster than writing 200 U() ops"). It escapes `$` into `\$` and ruins every node it touches; the damage is silent until a human sees the render. There is no fast path for this — enumerate, batch, verify.
- Writing nested shape updates as dot-paths (`{ "stroke.fill": "$X" }`) instead of full objects (`{ stroke: { fill: "$X", thickness: 1 } }`). The dot-path form silently no-ops on some property shapes, leaving the old value in place and wasting the rest of the batch's context.
- Declaring a batch "done" because `batch_design` returned `success: true`. Success at the API layer ≠ correct render. Re-read the written nodes for residual `\$` and screenshot the root frame before handing off.
- Overwriting a nested object (`stroke`, `effects[*]`) with a partial object when repairing one field. Read the existing nested value first, mutate the target field, write back the complete object.

## Tips for Success

Positive complement to the Anti-patterns list. Do these, not only avoid the inverse.

- **Brief first, rules second, taste third**. R1-R4 enforce correctness; the semantic brief enforces coherence. Taste operates inside the space the first two leave open - never the reverse.
- **Name colors by role before you look at a hex**. "brand", "surface", "muted" drive composition; "#EA580C" is only the *value* of brand. If you start from the hex, you will end up with three near-duplicates and no semantic story.
- **When the scale offers two adjacent values, pick by intent, not by habit**. Between radius 12 and 16, `soft` geometry says 12, `generous` says 16. Between font-size 16 and 18, dense editorial says 16, airy brochure says 18. The brief breaks the tie.
- **Pre-flight `get_variables` and `get_guidelines` even on small tasks**. A 30-second token check prevents 30-minute refactors when R1 catches a duplicate hex.
- **Extract on the second appearance, not the third**. Two cards inline is the last moment you can cheaply refactor; three inline means the fourth is already being conceived as "just one more inline one".
- **Name viewport frames before you populate them**. `Home__mobile-base-375` as an empty frame already unlocks the R4 grep cookbook; renaming a populated frame requires updating every ref and override that points into it.
- **Screenshot every root before declaring done**. A valid JSON batch with invalid visual output is the most common way to miss a brief mismatch.
- **Document a new token the moment you need it twice**. If `#F5F5F5` shows up twice in a single batch, the second occurrence is proof it should be `$surface` - stop the batch, `set_variables`, restart.

## Additional resources

- **[references/worked-example.md](references/worked-example.md)** - end-to-end walkthrough: brief → tokens → `batch_design` call → verification. Read when you need a concrete reference for what an "on-brief, on-rule" pass looks like.
- **[references/tailwind-scale.md](references/tailwind-scale.md)** - the canonical scale for R3. Consult at the start of every task to load the full tables (spacing, font sizes, radii, line heights, breakpoints, snap targets).

## Tone

Be decisive. Brief first, rules second, taste third. Pencil designs authored under this skill should be indistinguishable from those audited clean by `/pencil-audit` - the point of the skill is to make that audit a no-op.

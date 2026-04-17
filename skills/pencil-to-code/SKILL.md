---
name: pencil-to-code
description: Translate a Pencil (.pen) design into source code for any target stack. Auto-detects Tailwind CSS in the target project; if present, maps design pixel values to Tailwind class names; if absent, emits raw pixel values against the same 4px-grid scale without adding Tailwind as a dependency.
when_to_use: Transferring a finalized Pencil frame or component into a codebase, or when asked to "implement this design" from a `.pen` source.
argument-hint: "[frame-id-or-component-name]"
disable-model-invocation: true
allowed-tools: mcp__pencil__get_editor_state mcp__pencil__get_variables mcp__pencil__batch_get mcp__pencil__search_all_unique_properties
model: sonnet
effort: high
---

# /pencil-to-code

Manual command that turns a Pencil design into code for the host project's styling technology, preserving the plugin's universal sizing scale at every step.

## Philosophy

This plugin enforces a 4px-grid sizing scale on the **design side** regardless of what the target codebase uses. That scale happens to coincide with Tailwind CSS v4's default scale, which makes Tailwind a convenient **code-side** target when a project already uses it - but Tailwind is never required. If the target project does not use Tailwind, the plugin still emits values from the same scale using whatever styling technology the project prefers (vanilla CSS, CSS Modules, styled-components, inline styles, etc.).

## Detection - decide the output style first

Before writing any code, detect the target project's styling technology:

1. **Tailwind CSS**: present if any of the following holds:
   - `tailwindcss` listed in `dependencies` / `devDependencies` in `package.json`
   - `@tailwindcss/vite`, `@tailwindcss/postcss`, or `@tailwindcss/cli` present
   - A CSS file contains `@import "tailwindcss";` (v4) or `@tailwind base;` / `@tailwind components;` / `@tailwind utilities;` (v3)
   - A `tailwind.config.{js,ts,cjs,mjs}` file exists (v3) or an `@theme { ... }` block is defined in CSS (v4)
2. **CSS Modules**: `*.module.css` files exist or a build config references CSS Modules.
3. **Styled-components / Emotion / vanilla-extract / UnoCSS / Panda / Stitches**: detected by the corresponding dependency in `package.json`.
4. **Plain CSS / SCSS / LESS**: fallback when none of the above apply.

If the invocation prompt explicitly requests a specific output style, honor that instead of auto-detection.

## Output rules

### When Tailwind is detected

- Translate every pixel value from the `.pen` design into its Tailwind class equivalent using the mapping in `references/tailwind-scale.md`.
- Prefer named utilities (`p-4`, `text-lg`, `rounded-xl`) over arbitrary values (`p-[16px]`, `text-[18px]`).
- Arbitrary values are **only** permitted for legitimately off-scale requirements (1px hairlines, exact icon sizes, brand-mandated measurements). Any arbitrary value in the emitted code must be justified inline.
- Tokens declared in the `.pen` variables map to the project's Tailwind `@theme` block when one exists. If a matching token is missing in the project's theme, flag this - do **not** silently hardcode the raw value.
- Breakpoints: use `sm:`, `md:`, `lg:`, `xl:`, `2xl:` exactly.
- Line heights: use `leading-none`, `leading-tight`, `leading-snug`, `leading-normal`, `leading-relaxed`, `leading-loose` for the six allowed multipliers.

### When Tailwind is NOT detected

- Emit raw pixel values taken directly from the `.pen` design (e.g., `padding: 16px;`, `font-size: 18px;`, `border-radius: 12px;`, `line-height: 1.5;`).
- Preserve the declared Pencil tokens by translating them to the project's native token system when one exists:
  - CSS custom properties: `var(--spacing-md)`, `var(--color-brand)` if the target defines them
  - Design token file (e.g., `tokens.json`, `theme.ts`): reference the equivalent token
  - Otherwise: literal pixel values
- Do **not** introduce Tailwind as a new dependency to the project. The scale is enforced by the **numeric values** alone, which are already present in the design.
- Use media queries at the detected breakpoints: `@media (min-width: 640px)`, etc.

### Universal rules (both modes)

- Every value written must originate from the scale (pixels: multiples of 4 up to 384, plus fractional derivatives 2/6/10/14; font sizes from the allowed set; radii from the allowed set; line heights from the allowed multipliers). If the `.pen` design contains an off-scale value, **do not emit it** - stop and run `/pencil-audit` instead to surface and fix the design-side divergence.
- Component structure in code should mirror the Pencil frame hierarchy unless the host codebase has an existing component to reuse.
- Honor existing naming and file conventions in the target project.

## Workflow

### Step 0 - Load the project's design brief if present

Check for `design.md` at the project root. If present, read it. It holds role-named colors, geometry/depth commitments, and the token vocabulary map. Use it to drive naming in the emitted code:

- Role-named tokens (e.g., `$brand`, `$surface`, `$muted`) should translate to equivalently role-named CSS variables (`--color-brand`), Tailwind `@theme` entries (`--color-brand`), or the target stack's token primitive - never hardcoded hex.
- If Tailwind is detected and the project's `@theme` block does not yet declare a token for a role listed in `design.md`, flag this in Step 5 output rather than silently inlining the hex.

If `design.md` is absent, proceed without it - but note in the final report that emitting role-named code required extra inference.

### Step 1 - Read the design source

Dispatch the `pencil-navigator` subagent with shape `extract-props-from-ids` listing the target frame ID(s). If the caller gave a feature name instead of IDs, dispatch `find-by-feature` first to obtain them. Navigator returns a compact record per frame: layout, spacing, typography, colors, radii, and referenced tokens - without flattening subtrees into your context. Do **not** call `batch_get` directly for this step.

### Step 2 - Detect target styling technology

Follow the Detection section above. Record whether Tailwind is present. Record any existing token/theme system.

### Step 3 - Validate the design is on-scale

Before writing code, confirm every extracted value belongs to the scale. If any value is off-scale, halt and recommend `/pencil-audit` first. Emitting off-scale values in code perpetuates the divergence.

### Step 4 - Emit code

Write the component file(s) using the style rules above. One component per file. Follow the host project's conventions for file naming, extension, imports, and structural patterns.

### Step 5 - Report

Summarize briefly:

- Styling technology used
- Files created / modified
- Tokens referenced
- Any design-side tokens missing from the target theme (if Tailwind mode) that the user should add

## Canonical reference

**Authoritative spec**: `references/tailwind-scale.md` (sibling of this SKILL file).

Read that file at the start of every transpile task. It holds the full canonical scale (spacing, font sizes, font weights, line heights, letter spacing, border radius, border widths, opacity, box shadow, z-index, breakpoints, plus device labels and Tailwind class mappings for every category). When Tailwind is detected in the target project, map every design-side value to the class listed there; when Tailwind is absent, the px values still come from the same tables.

The three `references/tailwind-scale.md` copies across the plugin (`agents/references/`, `skills/pencil-design/references/`, `skills/pencil-to-code/references/`) are kept byte-identical. Divergence is a bug.

## Related skills and agents

- `../pencil-analyze/SKILL.md` - persists a semantic brief (`design.md`) from the `.pen`; emitted code honors its role-named tokens
- `../pencil-design/SKILL.md` - the companion authoring skill; use it when composing the design, so the `.pen` never leaves the scale in the first place
- `../pencil-audit/SKILL.md` - the companion audit skill; run first if any extracted value is off-scale

## Tone

Be practical. One component at a time. No speculative refactors. Don't reshape the host project's conventions just because Pencil uses a different convention - adapt to the target. If the target stack is unusual (e.g., Elm, SwiftUI, Compose, Flutter), translate the scale's numeric values into the stack's native units, preserving the design's proportions exactly.

---
name: pencil-audit
description: Run a complete design-quality audit over any repository that contains Pencil (.pen) files. Enforces the plugin's universal 4px-grid scale, variable-first tokens, and component-reuse heuristics across both design and companion code - framework-agnostic, stack-agnostic. Delegates to the `pencil-auditor` subagent, which runs read-only in an isolated context.
when_to_use: Before commits, before opening PRs, after Pencil design edits, or any time you want confidence that design and code share one consistent vocabulary.
argument-hint: "[path-to.pen-or-scope]"
disable-model-invocation: true
---

# /pencil-audit

Manual command to verify design quality across a Pencil-based project. Runs a read-only audit and returns a structured divergence report.

## When to run

- Before committing UI/component changes
- Before opening a PR that involves design
- After editing frames in a `.pen` file
- After extending design tokens (new colors, spacing values, etc.)
- Any time you want confidence that the design and code share one consistent vocabulary

## What it checks

The audit delegates to the `pencil-auditor` subagent, which applies four rule families. **Rule priority: R5 > R1 > R3 > R2.** R5 takes precedence because it describes a broken render, not a style choice — any R5 finding is a release blocker.

### R5 - Variable-reference integrity (release blocker)

String-valued properties whose value begins with `\$` (literal backslash + `$`) instead of the canonical `$`. These arise when a writer tool escapes `$` — notably `replace_all_matching_properties` when promoting hex/numeric values to variable references. The resulting string does NOT resolve to the declared variable; renderers fall back to defaults (usually black), silently ruining the design.

Example: `fill="\$white"` → should be `fill="$white"` (restore via `batch_design` U() op).

**Any R5 finding is a release blocker.** Fix R5 BEFORE acting on R1/R3 — promoting tokens over a corrupted property compounds the damage. The safe fix path is `batch_design` with explicit `U(nodeId, { prop: "$token" })` ops, which preserve `$` literally. Never use `replace_all_matching_properties` for values starting with `$` — it is the origin of R5 corruption.

### R1 - Variable-first design tokens

Every color, spacing, radius, and font-size in the `.pen` file that has a declared token should reference that token, not be hardcoded. Example: if `$brand = #EA580C` exists, a node with `fillColor: "#EA580C"` should use `fillColor: "$brand"`.

### R2 - Component-reuse candidates (best-effort)

Repeated frame structures that could be extracted to reusable components. Conservative - only flags obvious candidates.

### R3 - Scale parity

Every `fontSize`, `cornerRadius`, `gap`, `padding`, and `lineHeight` in Pencil and every sizing value in source code must match the plugin's universal sizing scale. This rule applies **regardless of the styling technology in use** - the scale is a pure numeric standard.

- Spacing (px): `0, 2, 4, 6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60, 64, 72, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 384`
- Font sizes (px): `12, 14, 16, 18, 20, 24, 30, 36, 48, 60, 72, 96, 128`
- Radii (px): `0, 2, 4, 6, 8, 12, 16, 24, 32, 9999`
- Line heights: `1, 1.25, 1.375, 1.5, 1.625, 2`
- Breakpoints (px): `640, 768, 1024, 1280, 1536` (mobile base `375`)

All audit diagnostics are expressed in **pure pixel values** (or multipliers for line-height). Framework-specific class translation (e.g., Tailwind class names) is handled separately by the `/pencil-to-code` skill when a styling library is detected.

## How to invoke from Claude Code

When the user types `/pencil-audit` (or asks for a design-quality audit), spawn the subagent via the `Task` tool:

```
subagent_type: pencil-auditor
prompt: "Run a full design-quality audit over this repository. Detect all .pen files and code files under standard paths, apply rules R5 (variable-ref integrity), R1 (variables), R2 (reuse), and R3 (scale parity), and produce the structured report. R5 scan must run first on every .pen file. Do not modify any files."
```

Return the subagent's report **verbatim** to the user. Do not summarize, edit, or add your own commentary to the findings.

If the user provides specific paths or globs in their invocation (e.g., `/pencil-audit --pen design/custom.pen --code 'src/ui/**/*.tsx'`), pass those constraints to the subagent in the prompt.

## Expected output shape

The subagent returns a structured report. Example PASS:

```
# Pencil Lint Audit

100% design-quality parity verified.
- R1: all properties use declared tokens where tokens exist
- R3: all sizing values match the scale
- Overall: clean

Design and code are in sync with the standard.
```

Example FAIL with corruption blocker:

```
# Pencil Lint Audit

## Summary
- Pencil files scanned: 1
- Code files scanned: 127
- Styling technology detected: tailwind
- R5 (corruption) blockers: 247
- R1 violations: 4
- R2 candidates: 1
- R3 violations (Pencil): 12
- R3 violations (code): 3
- Overall: FAIL - 267 total issues — RELEASE BLOCKER: corruption detected

## R5 - Corrupted variable references (release blocker)
design/ui.pen#heroTitle - fill="\$white" -> restore to "$white"
design/ui.pen#card1 - cornerRadius="\$radius-md" -> restore to "$radius-md"
design/ui.pen#btn - stroke.fill="\$gray-200" -> restore to "$gray-200"

## R1 - Variable-first violations
design/ui.pen#card2 - fillColor=#EA580C -> should reference $brand

## R3 - Scale violations (Pencil)
design/ui.pen#heroTitle - fontSize=38 -> snap to 36

## Recommendations
1. Restore every `\$X` → `$X` via `batch_design` U() ops FIRST (no `replace_all_matching_properties`). After the batch, `batch_get` the same nodes and confirm zero `\$` remain; screenshot the root frame to rule out black fallbacks.
2. Re-run `/pencil-audit` after corruption is cleared.
3. Only then apply R1/R3 snaps — promoting tokens over corrupted values compounds damage.
4. DO NOT use `replace_all_matching_properties` for variable promotion. It escapes `$` into `\$` and is the origin of R5 corruption.
```

## If divergences are found

**R5 corruption first, always.** Restore every `\$X` → `$X` via `batch_design` with explicit `U(nodeId, { prop: "$token" })` ops — never `replace_all_matching_properties`, which is the origin of the corruption. Re-run `/pencil-audit` to confirm zero blockers before touching R1/R3.

Once R5 is clean, apply R1/R3 snaps via `batch_design` U() ops to bring the design back onto the scale. Bulk token promotion MUST also go through `batch_design` U() (not `replace_all_matching_properties`) to avoid reintroducing R5 corruption. If the companion project uses a styling library with an equivalent scale (e.g., Tailwind CSS), follow up with `/pencil-to-code` to translate raw pixel values into the library's class names.

Do **not** commit with active divergences — especially not with R5 blockers. The plugin's philosophy is zero tolerance for scale drift and zero tolerance for corrupted variable references.

## Related files

- `agents/pencil-auditor.md` - the subagent this skill dispatches
- `skills/pencil-design/SKILL.md` - preventive companion; authors new `.pen` designs already compliant with the rules this skill audits, and bans `replace_all_matching_properties` for `$`-values at the source
- `skills/pencil-to-code/SKILL.md` - translates a compliant design into code (Tailwind-aware when installed)
- `README.md` - plugin installation, configuration, and philosophy

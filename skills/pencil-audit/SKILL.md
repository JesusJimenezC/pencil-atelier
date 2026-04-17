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

The audit delegates to the `pencil-auditor` subagent, which applies three rule families:

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
prompt: "Run a full design-quality audit over this repository. Detect all .pen files and code files under standard paths, apply rules R1 (variables), R2 (reuse), and R3 (scale parity), and produce the structured report. Do not modify any files."
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

Example FAIL:

```
# Pencil Lint Audit

## Summary
- Pencil files scanned: 1
- Code files scanned: 127
- Styling technology detected: tailwind
- R1 violations: 4
- R2 candidates: 1
- R3 violations (Pencil): 12
- R3 violations (code): 3
- Overall: FAIL - 20 total issues

## R1 - Variable-first violations
design/ui.pen#card1 - fillColor=#EA580C -> should reference $brand

## R3 - Scale violations (Pencil)
design/ui.pen#heroTitle - fontSize=38 -> snap to 36
design/ui.pen#pill - cornerRadius=20 -> snap to 24

## R3 - Scale violations (code)
src/components/Card.astro:42 - text-[13px] -> snap to 14

## Recommendations
1. Apply snaps listed above before commit
2. Consider extracting repeated "Card" pattern to reusable component
```

## If divergences are found

Apply the suggested snaps to bring the design/code back onto the scale. If the companion project uses a styling library with an equivalent scale (e.g., Tailwind CSS), follow up with `/pencil-to-code` to translate the raw pixel values into the library's class names.

Do **not** commit with active divergences. The plugin's philosophy is zero tolerance for scale drift - small inconsistencies compound into noticeable visual disharmony across a project.

## Related files

- `agents/pencil-auditor.md` - the subagent this skill dispatches
- `skills/pencil-design/SKILL.md` - preventive companion; authors new `.pen` designs already compliant with the rules this skill audits
- `skills/pencil-to-code/SKILL.md` - translates a compliant design into code (Tailwind-aware when installed)
- `README.md` - plugin installation, configuration, and philosophy

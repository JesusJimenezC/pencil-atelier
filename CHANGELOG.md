# Changelog

All notable changes to `pencil-atelier` are documented here. This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2026-04-16

### Context

In production use of v0.1.0, a bulk R1 token promotion via `mcp__pencil__replace_all_matching_properties` silently corrupted **every** variable reference in a real `.pen` file. The tool escapes `$` into `\$` when the `to:` value is a variable reference, storing literal strings like `\$white` that do NOT resolve to the declared variable. Renderers fell back to default colors (black) and the entire design visually broke.

v0.1.1 closes that loop with a new rule (R5), detection in the auditor, and explicit plugin-wide prohibition on `replace_all_matching_properties` for any value beginning with `$`. The prevention is **rule-based**, not a separate skill — a clear instruction in the existing authoring surface (`/pencil-design`) prevents the misuse at the source, and the auditor's Recommendations section embeds an inline `batch_design` U() recipe for the rare recovery case.

### Added

- **R5 — Variable-reference integrity.** The `pencil-auditor` subagent now scans every `.pen` file for corrupted variable references (strings beginning with `\$` instead of `$`). Any R5 finding is a release blocker and is reported before R1/R3 in the audit output.
- **Safe-apply discipline** (documented in `CLAUDE.md` and enforced by `/pencil-design`). Explicit plugin-wide prohibition on `replace_all_matching_properties` for any `to:` value beginning with `$`. All token-promotion writes must go through `batch_design` with explicit `U(nodeId, {prop: "$token"})` ops.
- **Nested-property write guidance.** When updating nested shapes like `stroke` or `effects[*].shadow`, pass the full object (`U(id, { stroke: { fill: "$X", thickness: 1 } })`) rather than dot-path keys (`"stroke.fill"`), which silently no-op on some property shapes.
- **Post-apply validation requirement.** After any `batch_design` pass that writes `$`-values, re-run `batch_get` on the written nodes, confirm zero `\$` strings remain, and screenshot the root frame. Silent success in the API does not prove the design renders.

### Changed

- **Rule priority** across `pencil-auditor.md`, `pencil-audit/SKILL.md`, and `CLAUDE.md`: now **R5 > R1 > R3 > R2**. R5 takes precedence because it describes a broken render, not a style choice.
- **Auditor workflow** (`agents/pencil-auditor.md`). Step 2 is now R5 corruption scan (runs first); Steps 3–6 renumbered accordingly. When R5 detects any finding, the summary appends `RELEASE BLOCKER: corruption detected`. The R5 Recommendations section embeds a `batch_design` U() recipe inline (including the nested-property form) so recovery never requires leaving the audit report.
- **Audit report format**. New `## R5 - Corrupted variable references (release blocker)` section appears before R1 when populated. Recommendations warn explicitly against `replace_all_matching_properties` and provide the safe recipe.
- **`/pencil-audit` skill**. Documents R5 alongside R1/R2/R3 and updates the FAIL example to show a corruption blocker.
- **Plugin description** in `plugin.json` and `marketplace.json`. Now mentions the safe-apply discipline that prevents `replace_all_matching_properties` variable-escape corruption.

### Security / Correctness

- The safe-apply discipline is a correctness fix for a silent data-corruption class. Previously, a single misuse of `replace_all_matching_properties` could render an entire design invisible with no error raised. v0.1.1 detects the corruption (R5), prevents the source via an explicit rule in `/pencil-design`, and provides the only safe recovery path (`batch_design` U() ops) in the audit report's Recommendations section.

### Migration notes

For users upgrading from v0.1.0:

1. Run `/pencil-audit` on every `.pen` file. If the report lists any `R5 (corruption) blockers`, your file has silent corruption from prior `replace_all_matching_properties` calls.
2. Apply the inline `batch_design` U() recipe from the audit's Recommendations section to restore all `\$X` → `$X` references. For nested properties (`stroke`, etc.), pass the full object, not dot-path keys.
3. Re-run `batch_get` on the written nodes and confirm zero `\$` strings remain. Screenshot the root frame.
4. Re-run `/pencil-audit` to confirm zero R5 blockers. Only then proceed with any remaining R1/R3 fixes.

For future work:

- Never call `replace_all_matching_properties` with a `to:` value starting with `$`.
- Use `batch_design` U() ops for all `$`-value writes (R5 repair, R1 promotion, any ad-hoc token fix).

## [0.1.0] - 2026-04-11

Initial release.

### Added

- `/pencil-analyze` skill + `pencil-analyzer` subagent: semantic-brief extraction, `design.md` persistence.
- `/pencil-design` skill: brief-first authoring pipeline (Pipeline → Brief → Rules → Workflow).
- `/pencil-audit` skill + `pencil-auditor` subagent: R1/R2/R3 enforcement across design and code.
- `/pencil-to-code` skill: transpile to host project styling (Tailwind auto-detected).
- `pencil-navigator` subagent: compact localization inside dense `.pen` files.
- Four house rules (R1 variable-first, R2 component reuse, R3 4px-grid scale, R4 viewport naming).
- Framework-agnostic scale in `references/tailwind-scale.md` (byte-identical across three locations).

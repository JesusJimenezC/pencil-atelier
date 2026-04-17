# pencil-atelier

A Claude Code plugin that turns [Pencil](https://pencil.dev) (`.pen`) design files and their companion codebase into a single, consistent, auditable design system — regardless of which CSS framework the project uses.

---

## The problem it solves

Design drift is the slow divergence between what a design file says and what a codebase ships.

- A Pencil frame picks `padding: 14px`, the implementation rounds to `16px`, and the next component rounds to `12px`. The grid becomes a suggestion.
- A palette role like `surface-elevated` lives in the design file, but the code hard-codes `#F7F7F8` in six places. The token exists only in the designer's head.
- Two features built a week apart end up with three different card radii, two shadow ramps, and a typography scale that nobody agreed on.
- An auditor would catch all of this — but there is no auditor, because no one has agreed on the rules.

`pencil-atelier` fixes this by shipping **one numeric standard** (a universal 4px-grid sizing scale) plus four slash commands that make the standard automatic: you authoring, auditing, or transpiling a design never leaves the rails, and the vocabulary you picked for one feature is the vocabulary the next feature inherits.

It is **framework-agnostic**: the rules are pure pixel values. If your project uses Tailwind, `pencil-atelier` maps to Tailwind classes. If it uses vanilla CSS, CSS Modules, styled-components, or anything else, the same pixel values are emitted verbatim — no styling dependency is ever forced on you.

---

## How to use (quick start)

### 1. Install

From a marketplace (once published):

```
/plugin marketplace add JesusJimenezC/pencil-atelier
/plugin install pencil-atelier@pencil-atelier-marketplace
```

Or locally, for development:

```
claude --plugin-dir /path/to/pencil-atelier
```

### 2. Open a Pencil file

Open any `.pen` file in your editor. The plugin does not require a specific project layout — it works on top of whatever you already have.

### 3. Run one of the four commands

| Command | Use it when |
|---|---|
| `/pencil-analyze` | You want to lock in the visual language of an existing `.pen` as a persistent brief. |
| `/pencil-design` | You are about to author a new frame, component, or page and want it correct by construction. |
| `/pencil-audit` | You want a read-only quality check across your `.pen` files and their companion code. |
| `/pencil-to-code` | You have a finished frame and want it transpiled into your codebase. |

### 4. (Recommended) Run `/pencil-analyze` once per project

```
/pencil-analyze
```

This creates a `design.md` file at the root of your project. From that moment on, `/pencil-design` and `/pencil-to-code` will read it automatically, so every future pass inherits the same atmosphere, palette roles, geometry, depth, and typography choices. No manual re-briefing, no session-to-session drift.

### 5. Iterate

- Author with `/pencil-design` (it enforces the safe-apply discipline: no `replace_all_matching_properties` for `$`-values, ever).
- Before committing, run `/pencil-audit`.
- If `/pencil-audit` reports R5 corruption blockers, restore every `\$X` → `$X` via `batch_design` U() ops (the audit report includes a ready-to-adapt recipe). Re-run the audit before touching R1/R3.
- When the design is frozen, run `/pencil-to-code` to produce production code.

That's the loop.

---

## The five house rules

Every tool in this plugin enforces the same rules. If you only remember one thing, remember these. **Priority when multiple rules flag the same node: R5 > R1 > R3 > R2.**

- **R1 — Variable-first tokens.** When a token exists for a value, reference `$token` instead of the literal. Pencil variables are the single source of truth for color, spacing, typography, and radius vocabulary.
- **R2 — Extract at the second repetition.** Do not componentize on the first instance. The second time the same structure appears, promote it to a reusable component. This avoids both hard-coding sprawl and premature abstraction.
- **R3 — Universal 4px-grid sizing scale.** Every pixel value comes from one numeric scale (framework-agnostic, defined in `references/tailwind-scale.md`). The scale happens to coincide with Tailwind v4's default scale, which is convenient when transpiling, but the rule applies to any codebase.
- **R4 — Viewport frame naming.** Top-level frames use the pattern `<PascalFeatureName>__<device>-<bp>-<width>` with exactly six allowed descriptor blocks (`mobile-base-375`, `mobile-lg-sm-640`, `tablet-md-768`, `laptop-lg-1024`, `desktop-xl-1280`, `desktop-lg-2xl-1536`). Grep-able, viewport-aware, unambiguous.
- **R5 — Variable-reference integrity (release blocker).** String-valued properties must use canonical `$token` references, never `\$token` (literal backslash + `$`). The corrupted form arises when `replace_all_matching_properties` is used to promote values to variable references — the tool silently escapes `$`, and the resulting string does NOT resolve to the declared variable. Renderers fall back to defaults (usually black) and the design silently breaks. Detected by `/pencil-audit`, repaired via explicit `batch_design` U() ops that preserve `$` literally. **Never use `replace_all_matching_properties` for any `to:` value starting with `$`.**

---

## How each element of the plugin works

The plugin is a small, opinionated pipeline. Four user-invoked slash commands coordinate three read-only subagents. The slash commands write; the subagents only observe.

### Slash commands (user-invokable)

#### `/pencil-analyze` — persist the design brief

Extracts the visual language of an existing `.pen` file and writes it to `design.md` at the project root. Delegates the heavy reading to the `pencil-analyzer` subagent (which runs in a forked, read-only context) and then translates the returned structured block into the canonical eight-section `design.md` prose: atmosphere, palette roles, geometry, depth, typography, scale parity, token vocabulary, and any known divergences.

Once `design.md` exists, it becomes a contract: `/pencil-design` and `/pencil-to-code` read it automatically at the start of every pass.

**Argument:** optional path to a specific `.pen` file. If omitted, uses the currently open file.

#### `/pencil-design` — author compliant designs

Creates new `.pen` content that is already correct by construction. The skill runs in three phases before composing anything:

1. **MCP retrieval pipeline.** A fixed sequence of Pencil MCP calls (`get_editor_state` → `get_guidelines` → `get_variables` → `search_all_unique_properties` → navigator dispatch → `find_empty_space` → `set_variables` → `batch_design` → `get_screenshot`) — ordering matters and is documented in the skill.
2. **Semantic brief capture** across five axes (loaded from `design.md` if present, otherwise captured in-session).
3. **The four house rules** applied to every value emitted into a frame.

The brief is the reason on-scale choices stay *mood-coherent* across a whole design. Dropping it produces technically valid but feeling-incoherent output.

**Argument:** optional feature name or frame description.

#### `/pencil-audit` — read-only quality check

Runs `pencil-auditor` against the project and returns its report verbatim. The auditor identifies:

- Corrupted variable references (R5, release blocker) — e.g. `fill="\$white"` that fails to resolve.
- Off-scale values (R3).
- Literals where a token exists (R1).
- Repeated structures that should be components (R2).
- Design/code parity drift (same value defined differently in `.pen` vs. code).

R5 runs first because corruption invalidates any other finding on the same property. If R5 reports blockers, restore every `\$X` → `$X` via `batch_design` with explicit `U(nodeId, { prop: "$token" })` ops (the audit report includes an inline recipe) before acting on R1/R3. Never use `replace_all_matching_properties` for `$`-values — it is the origin of R5 corruption.

The skill never modifies files. It reports; you fix.

**Argument:** optional path or scope (e.g., a specific `.pen` file or a component directory).

#### `/pencil-to-code` — transpile to the host project

Translates a finished Pencil frame into source code for whatever styling technology the host project uses. At Step 0 it detects the stack:

- **Tailwind present** → emits Tailwind class names mapped from the scale.
- **No Tailwind** → emits raw pixel values against the same scale, in the project's chosen styling technology (vanilla CSS, CSS Modules, styled-components, etc.).

Loads `design.md` first (if it exists) so that role-named tokens (`surface-elevated`, `text-muted`, etc.) survive transpilation instead of collapsing into raw hex values.

**Argument:** optional frame ID or component name.

### Subagents (read-only, invoked by the skills)

The plugin ships three subagents. None of them mutate files — they observe, reason, and report. The slash commands do all the writing. Each subagent has a deliberately-chosen model + effort profile so you get the right trade-off between depth and latency for the kind of work it does.

#### `pencil-analyzer` — semantic brief extractor

- **Purpose.** Look at a `.pen` file and infer its visual language. It characterizes the file across five axes — atmosphere, palette roles, geometry character, depth character, typography character — plus a scale-parity note (does the file actually live on the 4px grid?) and a token-vocabulary map (which tokens exist, which roles they play).
- **Invoked by.** `/pencil-analyze`. The skill dispatches the subagent via the `Task` tool, then translates its structured return block into the eight-section `design.md` prose.
- **Returns.** A structured brief block. Never writes `design.md` itself — that is the calling skill's job. Never mutates the `.pen`.
- **Runtime profile.** `model: opus`, `effort: high`, `maxTurns: 30`. Opus because the inference is genuinely hard: reading an entire design and deciding whether it feels "editorial" or "utilitarian" is a deep-reasoning task and cutting effort here visibly degrades the brief. High turn budget lets it cross-check consistency across many frames.
- **Discipline.** Honest-over-aesthetic. If the `.pen` is incoherent, the brief says so under `## Known Divergences` instead of fabricating a clean story.

#### `pencil-auditor` — quality enforcer

- **Purpose.** Systematically enforce the house rules across both the `.pen` and the companion codebase. Finds corrupted variable references (R5, release blocker), off-scale values (R3), literals where a token exists (R1), repeated structures that should be components (R2), and parity drift between design and code.
- **Invoked by.** `/pencil-audit`. The skill returns the auditor's report verbatim — no summary layer, no reformat.
- **Returns.** A structured divergence report (one line per finding) or a clean-status confirmation. Never modifies `.pen` files or source code.
- **Runtime profile.** `model: sonnet`, `effort: medium`, `maxTurns: 25`. Sonnet balances rigor and speed — auditing is mostly rule-matching, which does not need Opus-level reasoning, but it does need enough quality to distinguish "valid off-scale exception" from "drift". Medium turn budget matches the breadth of a typical audit sweep.
- **Discipline.** Reports only. Never claims work is "fixed" or "resolved" — those words belong to whoever acts on the report.

#### `pencil-navigator` — compact localizer

- **Purpose.** Translate vague spatial questions about `.pen` files ("where is the pricing card?", "which frame holds the mobile hero?", "which components reference `$radius.lg`?") into compact match records — ID + breadcrumb + minimal metadata — instead of raw node payloads. Protects the main agent's context window from `batch_get` output dumps.
- **Invoked by.** `/pencil-design` internally during its pipeline (Step 5), and on-demand from any skill that needs to locate frames, repeated structures, empty canvas space, or token usages without flooding context.
- **Returns.** Grep-able ID + breadcrumb records. Never returns whole subtrees — if the caller needs depth, the caller re-dispatches a targeted `batch_get`.
- **Runtime profile.** `model: haiku`, `effort: low`, `maxTurns: 10`. Haiku because the job is lookup, not reasoning — speed matters far more than depth, and over-thinking a localization query wastes time and tokens. Low turn budget reflects that most queries resolve in two or three tool calls.
- **Discipline.** Silent on intent. Does not second-guess why the caller wants the IDs; just returns them.

### The `design.md` contract

A plaintext markdown file at the root of your project, produced by `/pencil-analyze` and consumed by `/pencil-design` and `/pencil-to-code`. It holds the persisted semantic brief so vocabulary stays stable across sessions. Commit it to version control — it is as much a part of your design system as any token file.

### The scale reference

`references/tailwind-scale.md` is the single source of truth for R3, duplicated byte-identically into each execution context that needs it. You should never need to open it unless you are contributing to the plugin itself.

---

## Installation scopes

When you install via `/plugin install`, pick a scope:

- `--user` — available across all your projects (default).
- `--project` — shared with the team via version control.
- `--local` — gitignored, local to this checkout only.

All four commands become namespaced as `/pencil-atelier:<command>` (e.g., `/pencil-atelier:design`) to prevent collision with other plugins.

---

## Troubleshooting

- **Commands don't show up.** Run `/reload-plugins`. If still missing, verify the `.claude-plugin/plugin.json` manifest path and that the directories (`skills/`, `agents/`) sit at the plugin root, not inside `.claude-plugin/`.
- **The auditor reports divergences you disagree with.** The scale is the standard. If a rule genuinely does not fit your context, open an issue with a concrete example — the rules are opinionated on purpose, but they are not sacred.
- **`design.md` looks wrong after analysis.** The analyzer is honest-over-aesthetic: if the `.pen` is inconsistent, the brief calls it out under `## Known Divergences`. Run `/pencil-audit` for a structured list of what to fix.

---

## License

GPL-3.0-or-later. See [`LICENSE.md`](./LICENSE.md) for the full text.

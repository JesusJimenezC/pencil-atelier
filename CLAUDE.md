# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

`pencil-atelier` is a **Claude Code plugin** (not an application). It ships design-quality tooling for projects that use Pencil (`.pen`) design files alongside any codebase. Contents are exclusively markdown: skill definitions, subagent definitions, and one shared reference spec. There is no build system, no dependency manifest, no test runner - edits to this repo are edits to prompts.

## Plugin surface

Declared in `.claude-plugin/plugin.json`. Four user-invocable slash commands and three read-only subagents:

- `/pencil-analyze` (`skills/pencil-analyze/SKILL.md`) - delegates to `pencil-analyzer` and writes a `design.md` brief to the project root.
- `/pencil-design` (`skills/pencil-design/SKILL.md`) - authors new `.pen` content already compliant with the rules; loads `design.md` on Step A if present.
- `/pencil-audit` (`skills/pencil-audit/SKILL.md`) - delegates to `pencil-auditor` and returns its report verbatim.
- `/pencil-to-code` (`skills/pencil-to-code/SKILL.md`) - transpiles a finished design into the host project's styling technology; loads `design.md` on Step 0 if present to honor role-named tokens in emitted code.
- `pencil-analyzer` (`agents/pencil-analyzer.md`) - read-only semantic-brief extractor. Returns a structured block across five axes + scale parity + token vocabulary; never writes `design.md` itself (the calling skill does).
- `pencil-auditor` (`agents/pencil-auditor.md`) - read-only auditor. Reports divergences; never mutates.
- `pencil-navigator` (`agents/pencil-navigator.md`) - read-only localizer. Returns compact ID + breadcrumb records, never raw subtrees. Used by the other skills to avoid flooding context with `batch_get` output.

All four skill files set `disable-model-invocation: true`: the skills activate only when the user types the slash command (or the main agent explicitly dispatches them).

## Architectural invariants

### The scale is the product

Every rule in this plugin is grounded in one numeric standard defined in `references/tailwind-scale.md`. That file is **duplicated byte-identically** in three locations so each execution context (agent, skill-design, skill-to-code) can load it locally without crossing boundaries:

- `agents/references/tailwind-scale.md`
- `skills/pencil-design/references/tailwind-scale.md`
- `skills/pencil-to-code/references/tailwind-scale.md`

**Any edit to the scale must be mirrored across all three copies.** Divergence is a bug - the skill files explicitly state this. When modifying scale tables, update all three and verify byte-equality.

### Pencil-design is brief-first, then rules

`pencil-design/SKILL.md` runs three distinct phases before composition: (1) an explicit numbered **MCP retrieval pipeline** that fixes the tool-call order (`get_editor_state` → `get_guidelines` → `get_variables` → `search_all_unique_properties` → navigator dispatch → `find_empty_space` → `set_variables` → `batch_design` → `get_screenshot`); (2) a **Semantic brief** capture across five axes (atmosphere, palette roles, geometry character, depth character, typography character) that pre-filters the R3 scale to a coherent subset; (3) then the four house rules. The brief is the reason on-scale choices are coherent across a design - dropping it yields technically-valid but mood-incoherent output. When editing the skill, preserve this ordering: Pipeline → Brief → Rules → Workflow → Anti-patterns → Tips.

Concrete reference pattern: `skills/pencil-design/references/worked-example.md` shows an end-to-end PricingTier authoring pass. Update it whenever the pipeline shape, brief axes, or rule bindings change - readers follow the example to calibrate, so drift between example and spec is a real bug.

### `design.md` at project root is a contract across skills

`/pencil-analyze` produces `design.md`; `/pencil-design` and `/pencil-to-code` consume it. The file is the persisted semantic brief - atmosphere, palette roles, geometry, depth, typography - that outlives any single session and keeps vocabulary stable across features. The canonical structure lives in `skills/pencil-analyze/SKILL.md` (8 sections) with an example at `skills/pencil-analyze/examples/design.md`.

The file lives at the *consumer* project root, not inside this plugin. This plugin ships only the skill/agent logic that writes and reads it. When modifying the analyzer output shape (fields in the structured block emitted by `pencil-analyzer`), update in lockstep: analyzer output schema, analyze SKILL.md "Canonical `design.md` structure" block, example `design.md`, and the consumer steps in `pencil-design` Step A and `pencil-to-code` Step 0. Schema drift between these is how downstream skills silently degrade.

The analyzer is honest-over-aesthetic: it describes what the `.pen` *is*, not what it should be. If the `.pen` is inconsistent, `design.md` gets a `## Known Divergences` section and users are pointed at `/pencil-audit`. Do not weaken this discipline in edits - inflated briefs poison every downstream pass.

### Four house rules (shared vocabulary across skills/agents)

All skills and the auditor reference the same rule IDs. If you rename or renumber, update every file:

- **R1** - variable-first tokens (reference `$token` over literals when a token with that value exists).
- **R2** - extract to component at the second repetition (not the first).
- **R3** - universal 4px-grid sizing scale (framework-agnostic; the scale is numeric, not class-based).
- **R4** - viewport frame naming `<PascalFeatureName>__<device>-<bp>-<width>` with exactly six allowed descriptor blocks (`mobile-base-375`, `mobile-lg-sm-640`, `tablet-md-768`, `laptop-lg-1024`, `desktop-xl-1280`, `desktop-lg-2xl-1536`). R4 lives only in `pencil-design`; the auditor does not currently check it.

### Framework-agnostic by design

The scale is enforced as **pure pixel values** everywhere. Tailwind class mapping is isolated to `/pencil-to-code` and activates only when Tailwind is auto-detected in the target project. The auditor never emits framework-specific class names; the design skill never assumes a styling library. Do not leak Tailwind-specific syntax into `pencil-auditor.md` or `pencil-design/SKILL.md`.

### Read-only discipline for subagents

All three subagents (`pencil-auditor`, `pencil-navigator`, `pencil-analyzer`) are declared read-only in their frontmatter `tools:` list - none has access to `batch_design`, `set_variables`, `image`, `Write`, or `Edit`. When adding capabilities to a subagent, preserve this boundary: write operations (both `.pen` mutation and `design.md` authoring) belong in the calling skill (main agent), not in subagents.

### Delegation pattern

Main-agent skills delegate localization to `pencil-navigator` (via the `Task` tool) instead of calling `batch_get` directly. This is a deliberate context-window optimization; preserve it when editing skill workflows. Conversely, `/pencil-audit` delegates the full audit to `pencil-auditor` and must return its report verbatim - do not summarize or reformat the subagent output inside the skill. `/pencil-analyze` delegates extraction to `pencil-analyzer` and is responsible for translating the structured block into `design.md` prose - the subagent never writes the file itself.

### Frontmatter contract

All three skills use split `description` + `when_to_use` frontmatter (the combined string is capped at 1,536 chars in the skill listing, so splitting avoids truncation of trigger phrases). `disable-model-invocation: true` is set on all three to keep them strictly user-invoked. Skills that issue MCP writes (`pencil-design`) declare `allowed-tools` with the specific `mcp__pencil__*` tools they need to reduce permission prompts; read-only skills (`pencil-audit`, `pencil-to-code`) either declare the read-only subset or leave `allowed-tools` off entirely (they delegate writes to subagents or the host project).

When adding a skill: follow the same split, keep `description` ≤ ~400 chars, put trigger phrases under `when_to_use`, and declare `allowed-tools` only if the skill calls MCP tools directly.

## Working on this repo

Because everything is markdown, "testing" means reading the skill/agent prompts end-to-end and checking consistency:

- Rule IDs (R1/R2/R3/R4) referenced consistently across files.
- Scale numbers in inline recaps match `references/tailwind-scale.md`.
- Skill frontmatter splits `description` + `when_to_use`; combined length stays well under 1,536 chars.
- `disable-model-invocation: true` is preserved on all three skills.
- All three `references/tailwind-scale.md` copies remain byte-identical after any scale edit.
- `pencil-design` keeps Pipeline → Brief → Rules → Workflow ordering; `references/worked-example.md` matches the current brief axes and pipeline steps.
- Each SKILL.md stays under ~500 lines (official guidance); heavy reference material lives in `references/`.

# Changelog

All notable changes to the Arxlay System Design Skill are documented here.
The project follows [Semantic Versioning](https://semver.org/).

## [0.8.0] — 2026-05-13

### Polished — public launch readiness

No protocol change; this is the **refinement bump that marks the
skill as public-launch-ready**. Bundles the v0.7.2 anti-trigger
addition with documentation polish targeted at first-time external
users.

### Documentation

- **README — Codex CLI host added.** Codex CLI is now a first-class
  host alongside Claude Code / Cursor / Claude Desktop / Claude.ai
  web. Install snippet uses `.codex/skills/` (AGENTS.md format,
  natively supported as of late 2025).
- **README — anti-trigger behavior surfaced.** The "Triggers"
  section now explicitly says what happens on read-style phrases
  when the read-skill isn't installed: an explicit "read-mode isn't
  ready yet" reply instead of silent fall-through. This is the
  user-facing description of the v0.7.2 internal change.
- **Examples — bumped to v0.8.0+ protocol marker.** Greenfield and
  brownfield dialogues call out that their MCP calls and phase
  decisions are authentic to v0.8.0+.

### Why this is `0.8.0` and not `0.7.3`

The skill repo has been internal-facing through v0.7.x. With public
launch (per epic 020 Block A), this is the version readers find first
when discovering the repo. A clean minor bump signals "this is the
state of the skill at public launch" without making a 1.0 commitment
on API stability (which we'd want to back with longer field-testing).

### Closes

- Epic 020 Block A — public skill repo bootstrap polish.

## [0.7.2] — 2026-05-13

### Added — read-mode anti-trigger

Section 1 now explicitly handles the case where the user invokes a
**read-style phrase** ("tell me about my architecture", "describe
what I have", "extract architecture from code", and Russian
equivalents) while **only this design-skill is installed** (no
read-skill loaded in the session).

Previously the skill said "do NOT activate" on these phrases but left
the fallback ambiguous — the assistant could silently drift into a
generic answer or, worse, fall into the design flow against intent.

The added "Read-mode anti-trigger" sub-section gives an explicit,
bilingual reply template:

> Read-mode isn't ready yet — it's planned but not built. I can help
> you describe a new architecture from scratch, or start a first-run
> quickstart. Want to do one of those?

Mirrors user language per the existing Language rule. Does not
propose `query_elements` workarounds — that lives in the read-skill
scope this skill does not own.

### Why

Closes 025 D2 + verification row 6 ("read-skill anti-trigger in
design-skill v0.7.x deployed, manual test «расскажи про код» →
correct «read-mode пока не готов»"). Phase 6 launch gate item.

## [0.7.1] — 2026-05-11

### Fixed — failure mode caught in real session

First production run of v0.7.0 (founder's model `w0oG5BPQq1g6`,
2026-05-11 evening session) revealed the nesting heuristic was **too
soft**. The skill mentioned nesting in the recap prose ("modules will
lay inside their systems via ELK") but committed flat
(`place_all_in_batch: true`) and assumed ELK auto-nests on contains —
which it doesn't. Result: 25 relationships drawn as stretched arrows,
modules crammed in one heap, two parallel edges between same nodes
stacked their labels.

### Strengthened

- **Phase 4 nesting heuristic** rewritten as a 4-step mandatory
  protocol (scan → ask → commit form when agreed → fall-back when
  declined). Each step has explicit "must" wording; the section opens
  with "MANDATORY step before Phase 5 — this step is not optional".
- **Explicit `placements[]` instruction** in Step 3 — Switch the
  commit form to `placements[]`, NOT `place_all_in_batch`. Bullet-list
  spelling out parent_element_public_id population per child and
  parent-omit semantics for top-level elements.
- **Step 4 fall-back path** documented — if user declines nesting,
  use flat and tell them the canvas will use stretched containment
  arrows.

### Anti-pattern added

- **#17: Assuming ELK auto-nests on contains-relationships.** Explicit
  failure-narrative list — three phrasings that signal this anti-pattern
  ("modules will lay inside their systems via ELK auto-layout", "the
  layout will compute on first canvas view to nest", "ELK will nest
  them automatically"). All three describe behavior the renderer
  doesn't have — explicit parent metadata required.

### Epic context

Bug-fix release of v0.7.0 (epic 022 deliverable). Frontend
parallel-edge label overlap caught in the same session is a separate
defect — opened as wiki task in arxlay-monorepo.

## [0.7.0] — 2026-05-11

### Added

- **Phase 4 nesting heuristic.** Before the pre-commit recap, scan
  cumulative `accepted_relationships[]` for `contains`-class edges
  (Stdlib `contains`, ArchiMate `composition` / `aggregation` /
  `composedOf`). If ≥3 such edges share the same `source`, propose
  rendering the children *inside* the container instead of as flat
  parallel arrows. One yes/no question; default is flat if the user
  declines or the threshold isn't reached. Cap depth at 3 levels
  (matches the frontend `useGroupingStore` ceiling).
- **Phase 5 `placements[]` option.** New mutually-exclusive third
  placement form (epic 022 backend wire, shipped same day): per-element
  carrier for optional `parent_element_public_id`, `width`, `height`,
  `position.{x,y}`. Width clamp [80, 1200] px; height clamp [40, 800] px.
  Documented as the choice when the nesting heuristic triggers.
- **New artifact-validation error codes** surfaced in Phase 5 error
  handling: `artifact_placement_parent_missing`,
  `artifact_placement_parent_no_contains`,
  `artifact_placement_cycle`, `artifact_placement_bad_size`. Friendly
  Phase A errors — fix the placements, retry with fresh idempotency.

### Epic context

Released as the deliverable of epic 022 (sizes + nesting via MCP).
Backend ships `commitArtifactCreateArgs.placements[]` and
`SnapshotPosition.parent_element_public_id`; frontend hydrates
`Diagram.nodeParentMap` from the snapshot so ReactFlow's `parentNode`
renders nested containers without an explicit drag-in. Closes the
visual-coupling concern from the 2026-05-11 model `w0oG5BPQq1g6`
session where 12 `contains` edges remained as flat arrows.

## [0.6.0] — 2026-05-11

### Added

- **Phase 2 type triage** — fast inference step at the end of Phase 2,
  before any Phase 3 draft. Three questions against the inventory (data
  store? external SaaS? human rank?) pick the correct stratum
  (`microservice` vs `system`, `user` vs `role`) up-front so commit
  doesn't reject cross-stratum edges. Closes the root cause from the
  2026-05-11 model `w0oG5BPQq1g6` session where 9 `system → database`
  edges were rejected.
- **Phase 3 proactive allowance check.** Skill now calls
  `query_allowed_relationships` (MCP backend tool shipped 2026-05-11,
  epic 016) for any non-trivial pair — cross-stratum, custom layer,
  non-default notation. The tool is the source of truth; the
  cheat-sheet is the fast path. Failure modes spelled out:
  `allowed: []` → propose intermediate / swap type / report honestly;
  proposed rel not in `allowed` → pick `allowed[0]` or by semantic fit
  and *say what you swapped*.
- **Section 4 cheat-sheet** with seven Stdlib v1.0.0 patterns (system /
  microservice / module / user / role / database / external-system).
  Each row carries the "why this and not the obvious neighbour"
  disambiguation reason — covers ~95% of design-mode sessions without
  needing the allowance tool.

### Changed

- **Anti-pattern #15** updated — the allowance query tool is no longer
  hypothetical. Reference it explicitly as the source of truth, with the
  cheat-sheet as the fast path.
- **Relationship picker** in Section 4 — `microservice → database` now
  uses `arxlay:storesIn`, not `arxlay:uses`. Matches the Stdlib v1.0.0
  allowance set documented in the wiki memory note
  `archimate-allowance-stricter-than-spec`.

### Anti-patterns added

- **#16: Skipping Phase 2 type triage.** Naming everything
  `arxlay:system` because the user said "service" is the most common
  reason a session ends with rejected `system → database` edges. Triage
  is inference, not a question — but if the inventory is ambiguous,
  ask *before* type assignment, not after commit rejection.

### Epic context

Released as the deliverable of arxlay-wiki epic 023 (skill smart
inference и proactive allowance), which depended on epic 016
(`query_allowed_relationships` MCP tool — backend shipped earlier the
same day, arxlay-monorepo commit `6e177bc`). Calibration retro
scheduled after 4 design-mode sessions (target ≈Q3 2026).

## [0.5.0] — 2026-05-10

### Added

- **Tone section** at the top of SKILL.md: hide metamodel jargon
  (`stdlib`, `ArchiMate`, `allowance`, `personality`, `baseTypeId`, raw
  type ids) from user-facing replies. Substitute plain words via the
  translation table. The user invoked a design skill, not a type-system
  tour (B33).
- **Chunked iterative Phase 3.** The proposal phase no longer dumps the
  full draft in one wall of text. Instead, five strict slices —
  anchor → modules → data → externals → cross-cutting relationships —
  each ≈ 6 lines, each ending in one explicit confirmation question.
  Cumulative draft state lives in the conversation; `commit_changes`
  still fires exactly once at the end (B31, B34).
- **Optional Explore subagent for code discovery.** Section 3 documents
  when to offload repo scanning to a parallel `Explore` subagent during
  Phase 2's code path — keeps file reads out of the main context and runs
  alongside MCP warm-up. Single subagent, returns a flat inventory; the
  main agent stays in charge of dialogue and `commit_changes` (B35
  minimal version).

### Changed

- **Phase 4 refinement** is no longer a separate gate. Edits happen
  *between* Phase 3 slices and are applied to the running draft state.
  The only distinct moment is one consolidated recap immediately before
  Phase 5 commit.
- Phase 3 sample dialog rewritten in Russian to match the new chunked
  loop; trade-off probing now happens in the **next** reply after a
  user's choice, never piled into the same message.

### Anti-patterns (Section 6) added

- #13: walls of text in Phase 3.
- #14: leaking metamodel jargon.
- #15: guessing allowance and finding out at commit time.

## [0.4.0] — 2026-05-10

### Added

- **Language mirroring rule** at the top of SKILL.md. The skill now stays in
  the user's language (English / Russian / mixed) for every reply — pre-flight
  messages, questions, proposals, errors. Previously, replies defaulted to
  English unless the user explicitly asked otherwise (founder feedback B28).
- **`create_model` tool support in Phase 1 and Section 7.2.** When
  `list_models` returns empty, the skill now asks for a model name and calls
  the new MCP `create_model` tool to provision one in-band. Replaces the
  previous "go create one in the UI and come back" dead-end (B29).
- **Brownfield model awareness in Phase 1.** The Phase 1 consolidated reply
  now surfaces a per-type element breakdown (e.g. "23 elements: 8 services,
  5 datastores, 3 actors, 7 other; 2 diagrams") and skips the notation
  question when the existing model already has a clear dominant notation —
  so the user sees the skill understands the model before being asked
  anything (B30).

### Changed

- Phase 1 step 2 now distinguishes 0 / 1 / N model cases explicitly.
- Section 7.2 step 1 handles the empty-workspace case via `create_model`;
  the only first-run question that lives outside 7.5.

## [0.3.0] — 2026-05-09

### Added

- **`artifacts.create[]` in every commit.** Every `commit_changes` call from
  the skill — both full design-mode (Phase 5) and first-run quickstart
  (Section 7.6) — now includes exactly one `artifacts.create[]` entry with
  `place_all_in_batch: true`. The user lands on a populated canvas
  immediately after "save", not on an empty canvas with a populated catalog.
  This closes the design-mode visibility gap that drove epic 015.
- **`list_artifacts(model_id)` call in Phase 1 (brownfield only).** Records
  existing diagrams so artifact name collisions are avoided in Phase 5 and
  the Phase 1 reply tells the user up-front a *new* diagram will be created
  (existing layouts stay untouched).
- **Canvas URL handed back from `artifacts_created[0].canvas_url`.** Both
  Phase 5 and Section 7.7 now read the URL the server returns and show it
  as a clickable link rather than constructing a guess. Format is
  `<FrontendBaseURL>/m/<model>/schemas/<artifact>` (matches the existing
  frontend route — see epic 015 D9 pivot).
- New anti-patterns: "Committing without an artifact" (Section 6 #11),
  "Touching existing artifacts" (Section 6 #12), "Hand-constructing the
  canvas URL" (Section 7.8 #8), "Skipping `artifacts.create[]` in 7.6"
  (Section 7.8 #9).

### Changed

- Section 2 Phase 1 step list grew from 6 to 7 items (added `list_artifacts`
  call between `query_elements` and code-access check).
- Phase 5 step list grew from 5 to 6 actionable items (added artifact
  UUID generation), and the `commit_changes` payload spec now spells out
  the `artifacts.create[]` entry (name, notation, artifact_type,
  `place_all_in_batch`).
- Phase 5 success path now reads `artifacts_created[0].canvas_url`,
  reports the artifact name + count, and tells the user the layout
  auto-computes on first canvas open.
- Phase 5 validation-error handling lists the new artifact-specific codes
  (`artifact_notation_invalid`, `artifact_type_invalid`,
  `artifact_notation_type_mismatch`, `artifact_placement_conflict`,
  `artifact_placement_missing_endpoint`, `artifact_name_required`).
- Section 7.6 commit step adds the artifact entry with
  `name = "First-run snapshot — <repo-name> — <YYYY-MM-DD>"`,
  `notation: "archimate"`, `artifact_type: "graph"`,
  `place_all_in_batch: true`.
- Section 7.7 sample reply rewritten to pull canvas URL from the response
  and to mention auto-layout.
- Phase 1 sample dialog updated; brownfield variant added.

## [0.2.0] — 2026-05-08

### Added

- **Section 7 — First-run quickstart mode.** Activated by the slash-style
  triggers `/arxlay describe-architecture` and `/arxlay let's describe
  architecture`. Drives an empty Arxlay model from trigger to a committed
  C4 Container diagram in under five minutes, with one confirmation
  question and no trade-off elaboration.
- `references/discovery.md` — per-source extraction rules
  (docker-compose, manifests, README, directory layout, .env.example,
  Dockerfile) used by Section 7.3.
- `references/mapping.md` — extended signal → Stdlib type table and the
  relationship rubric for Section 7.4.
- `tests/fixtures/mini-node-monolith/` and `tests/fixtures/mini-go-microservices/` —
  minimal greenfield repositories with `EXPECTED.md` outputs for manual
  smoke-running first-run mode.

### Changed

- Section 1 (When to activate) lists the slash-style triggers and
  documents that they activate first-run mode while natural-language
  triggers stay on the full 5-phase flow.
- Frontmatter `description` updated to reference both modes.
- SKILL.md grew from ~2400 to ~4000 words (target per epic 013).

## [0.1.0] — 2026-05-07

### Added

- Initial release.
- Five-phase conversation pattern: Setup → Discovery → Proposal → Refinement → Approve & commit.
- Code awareness for Claude Code and Cursor (Read / Glob / Grep markers, README-first heuristic).
- Trade-off documentation format (`## Why` / `## Trade-offs` / `## Alternatives`) embedded in element descriptions.
- Atomic commit via Arxlay MCP `commit_changes` tool with optimistic locking and idempotency.
- Installation instructions for Claude Code, Cursor, Claude Desktop.
- Six-section SKILL.md (~2400 words): When to activate, Conversation flow, Code awareness, Element creation patterns, Trade-off documentation format, Anti-patterns.

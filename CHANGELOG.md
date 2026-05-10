# Changelog

All notable changes to the Arxlay System Design Skill are documented here.
The project follows [Semantic Versioning](https://semver.org/).

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

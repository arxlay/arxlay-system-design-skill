# Changelog

All notable changes to the Arxlay System Design Skill are documented here.
The project follows [Semantic Versioning](https://semver.org/).

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

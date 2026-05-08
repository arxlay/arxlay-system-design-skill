# Changelog

All notable changes to the Arxlay System Design Skill are documented here.
The project follows [Semantic Versioning](https://semver.org/).

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

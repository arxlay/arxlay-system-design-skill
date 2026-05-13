# Contributing to the Arxlay System Design Skill

Thanks for the interest. The skill is open source under Apache
2.0; we accept issues, MRs, and pull requests on
`github.com/arxlay/arxlay-system-design-skill`.

## What we accept

- **Bug reports** — open an issue with a reproducible session
  log (skill replies, MCP calls, what you expected vs. what
  happened).
- **Anti-pattern additions** — if you've hit a failure mode
  worth codifying, propose a new entry in `SKILL.md` Section 6
  (Anti-patterns).
- **Example additions** — new scenarios in `examples/`,
  particularly for less-common notations (BPMN, pure C4) or
  unusual codebases (monorepos, polyglot stacks).
- **Reference improvements** — `references/discovery.md` and
  `references/mapping.md` are field-tested heuristics; if your
  experience improves them, MR welcome.

## What we're more cautious about

- **Restructuring the five phases** — Phase 1-5 are battle-tested
  in production sessions. A restructuring MR needs a paired
  smoke-test artefact (see below) showing the new flow doesn't
  regress.
- **Lowering an instruction's strength** (e.g. "must" → "should",
  "MANDATORY" → "recommended"). These wordings were strengthened
  for a reason — usually a past production session where the
  softer wording was ignored. Track them down in `CHANGELOG.md`
  before relaxing them.
- **Removing anti-patterns.** Anti-patterns are reactive — each
  one represents a real session where the skill misbehaved.
  Don't remove without an issue thread that surfaces why it's
  no longer applicable.

## Pre-publish smoke checklist

We follow a **pre-publish self-test protocol** for every
**minor or major** version bump. Patches (typo, formatting,
single-line wording) are exempt.

Before pushing a `v0.X.0` or `vX.0.0` tag:

1. Open the scenario library —
   `wiki/researches/skill-smoke-scenarios.md` in the Arxlay
   wiki (`github.com/arxlay/arxlay-monorepo` or the published
   vault, ask in the issue thread if you don't have access).
2. Select scenarios:
   - **≥1 happy-path** (canonical five-phase flow without
     deviations).
   - **≥1 failure-mode** (anti-pattern that the skill must
     correctly refuse to walk into).
   - **1-2 regression** scenarios from baseline, especially if
     your bump touches code paths covered by past hotfixes.
3. Run each scenario manually in a fresh Claude Code session
   (`claude --no-history` or in a sandbox project).
4. Record the run in
   `wiki/exploratory-tests/<YYYY-MM-DD>-skill-arxlay-design-v<X.Y.Z>-smoke.md`
   using the template at
   `wiki/exploratory-tests/_skill-smoke-template.md`.
5. Each scenario gets a **binary PASS / FAIL** verdict.
   - **≥1 FAIL** → release blocked. Fix and re-run.
   - **All PASS** → release-ready.
6. Reference the smoke artefact in the MR description with
   a single line:
   ```
   Smoke: wiki/exploratory-tests/<YYYY-MM-DD>-skill-arxlay-design-v<X.Y.Z>-smoke.md
   ```
   MRs missing this line don't get merged.

Why this matters: skill instructions are LLM-driven, and prior
production sessions have shown that even well-worded
instructions can be too soft to land. The smoke artefact is
how we keep the surface honest. See the original protocol in
the upstream wiki for context.

## Versioning

Semantic versioning:

- **Patch** (`0.X.Y` → `0.X.Y+1`): typo, wording polish, doc
  refresh. Smoke exempt.
- **Minor** (`0.X.0` → `0.X+1.0`): new phase/section, new
  anti-pattern, new MCP tool usage, strengthened instruction.
  **Smoke required.**
- **Major** (`0.X.0` → `1.0.0`): restructured conversation flow,
  breaking change in skill behavior, change to commit semantics.
  **Smoke required, with ≥3 scenarios.**

Tag releases as `v<X.Y.Z>` and push the tag.

## Style

- Markdown, 4-space-indent continuation lines, ≤80 char wrap
  where reasonable (instructional prose is fine wider).
- No emojis in `SKILL.md` or any file the LLM reads — they
  carry inconsistent meaning across models and tokenisers.
- Russian phrases are fine in user-facing examples; the skill
  mirrors the user's language.

## Conduct

Bug reports and MRs welcome from anywhere. Disagreements
resolve on the technical merit. We don't tolerate hostility
in issue threads or MR review.

## Questions

- Architectural / behavioral: issue thread on the repo.
- Security: see `SECURITY.md`.
- Anything else: DM `@arxlay` on X.

# Arxlay System Design Skill

AI skill for collaboratively describing software architecture through a
guided conversation. Drops into Claude Code, Cursor, or Claude Desktop and
works over your [Arxlay](https://arxlay.com) MCP connection.

Activate with the trigger phrase **"Arxlay, let's describe the architecture"**
(see [Triggers](#triggers) for alternatives). The skill walks you through
five phases — Setup → Discovery → Proposal → Refinement → Approve & commit
— and ends with an atomic commit to your model via the Arxlay MCP server's
`commit_changes` tool.

## What you get

- Five-phase conversation pattern (~2400 words of LLM instructions).
- Code-aware in Claude Code and Cursor: skill peeks at `Dockerfile`,
  `package.json`, READMEs to seed the architecture conversation.
- Trade-off-first design: every committed element carries `## Why`,
  `## Trade-offs`, `## Alternatives` in its description, so a year from
  now you can ask "why did we pick Postgres for Auth?" and the agent has
  the answer.
- Atomic commit via the existing Arxlay MCP server — no extra
  authentication beyond the MCP setup you already did.

## Requirements

- A working Arxlay MCP connection. See
  [arxlay.com/docs/integrations/design-mode-skill](https://arxlay.com/docs/integrations/design-mode-skill)
  for one-time setup.
- Write permission on the target Arxlay model.
- An AI host that supports skills:
  - **Claude Code** (recommended) — full code awareness via Read/Glob/Grep.
  - **Cursor** — full code awareness via the editor's file access.
  - **Claude Desktop** — works, but no code awareness; falls back to
    interview-style discovery.
  - **Claude.ai web** — works via Connectors → Arxlay MCP, interview-only.

## Install

### Claude Code

```bash
mkdir -p .claude/skills
git clone https://github.com/arxlay/arxlay-system-design-skill .claude/skills/arxlay-design
```

Restart Claude Code. The skill is auto-discovered from `.claude/skills/`.

### Cursor

```bash
mkdir -p .cursor/skills
git clone https://github.com/arxlay/arxlay-system-design-skill .cursor/skills/arxlay-design
```

Cursor picks up the skill on the next chat session.

### Claude Desktop

Skills aren't first-class in Claude Desktop yet, but you can paste the
contents of `SKILL.md` into a project's "Custom instructions" or include
them inline at the start of a chat. Activate via the same trigger phrase.

## Triggers

Activates **only** on these explicit phrases at the start of a message:

- "Arxlay, let's describe the architecture"
- "Arxlay, let's design"
- "Arxlay, design mode"
- "Arxlay, давай опишем архитектуру" (Russian alternative)
- "Arxlay, давай опишем систему" (Russian alternative)

Does **not** activate on read-style questions like "tell me about my
architecture" or "what services do I have?". Those go to the Arxlay
read-skill (planned, separate repo).

## Quick example

> **You:** Arxlay, let's describe the architecture.
>
> **Assistant:** I see model "E-commerce platform" (version 0) with 0
> elements — greenfield. I have access to your code. Stdlib or ArchiMate?
> Code review or interview?
>
> **You:** Stdlib, code review.
>
> *(Three rounds of code reading and probing trade-off questions later…)*
>
> **Assistant:** I propose: Auth Service, Orders Service, Payment Service
> (Stripe), Postgres, Redis — each with its trade-offs. Save it?
>
> **You:** Save it.
>
> **Assistant:** Saved. 5 elements, 6 relationships. Model version: 1.

Time from trigger to commit on a typical small system: 15–25 minutes.

## Documentation

Full installation guide and end-to-end examples:
[arxlay.com/docs/integrations/design-mode-skill](https://arxlay.com/docs/integrations/design-mode-skill)

## Versioning

Semantic versioning. Bump rules:

- **Major** — change to the MCP tool contract the skill calls (currently `commit_changes`).
- **Minor** — change to conversation flow shape, trigger phrases, trade-off format.
- **Patch** — wording, typo, clarity.

See [CHANGELOG.md](./CHANGELOG.md).

## License

Apache License 2.0 — see [LICENSE](./LICENSE).

## Contributing

Issues and pull requests welcome. For significant changes (new conversation
phase, alternative trigger phrases, host-specific overrides), open an
issue first to discuss the direction before sending a PR.

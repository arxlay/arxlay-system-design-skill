---
description: Activate Arxlay design-mode skill — describe your architecture through dialogue
---

The user wants to enter Arxlay design mode to describe their architecture in conversation. This is the slash-trigger entry point; the long-form textual trigger ("Arxlay, давай опишем архитектуру" / "Arxlay, design mode") is equivalent and routes into the same flow.

Read the design-mode skill at one of (in priority order):

- `.claude/skills/arxlay-design/SKILL.md` (project-level)
- `~/.claude/skills/arxlay-design/SKILL.md` (user-level)

If neither file exists, tell the user verbatim:

> The Arxlay design-mode skill isn't installed. Follow the quickstart at https://arxlay.com/docs/quickstart-claude-code — it's two `git clone` / `cp` commands plus `claude mcp add`.

…and stop. Do not improvise the conversation flow without the skill.

If the skill is found, follow its 5-phase conversation flow starting from **Phase 1 — Setup** (verify MCP reachable, identify the active model, check for elements, check for code access, decide notation). Treat the user's opening intent as: $ARGUMENTS

If `$ARGUMENTS` is empty or generic ("design", "go", "let's"), skip the parsing step and proceed straight to Phase 1 Setup's consolidated discovery question.

Hard rules (also enforced by the skill, restated here so the slash entry doesn't drift):

- Do **not** commit anything until the user gives explicit approval ("сохраняй", "commit", "yes go").
- Do **not** suggest element types that aren't enabled in the model's metamodel.
- In brownfield (`query_elements` returns `total > 0`), check existing elements before proposing new ones.
- Do **not** ask the user 10 questions in one message. Phase 1 ends with one consolidated question.

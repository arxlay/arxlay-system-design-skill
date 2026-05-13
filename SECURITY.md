# Security

## Reporting a vulnerability

If you find a security issue in this skill — including issues
that arise only when the skill is connected to the Arxlay MCP
endpoint — please **report it privately**, not through public
issues.

- Email: `security@arxlay.com`
- Subject: `[arxlay-system-design-skill] <short summary>`

We acknowledge reports within 3 business days and aim to ship a
fix or mitigation within 14 days for high-severity findings.

## Scope

In scope:

- This skill (`SKILL.md`, `commands/`, `examples/`, `references/`).
- How this skill drives Arxlay MCP tools — if the skill can be
  prompted to make a destructive MCP call that bypasses normal
  user confirmation, that's in scope.

Out of scope (report to the Arxlay platform instead):

- The Arxlay MCP server itself (`https://arxlay.com/mcp`).
- The Arxlay web UI or backend.
- The OAuth flow on `arxlay.com`.

For platform-side reports, see `arxlay.com/security` or email
the same address — we'll route it.

## Acknowledged researchers

We credit reporters by name (with permission) in the changelog
entry that ships the fix.

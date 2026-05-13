# Greenfield example — describe a system from scratch

This example shows the canonical five-phase flow against an
**empty Arxlay workspace**. Use it as a reference for what a
clean run of `Arxlay, let's describe the architecture` looks
like end to end.

The dialogue is paraphrased — actual skill replies will vary in
wording — but the **structure, MCP calls, and decisions** are
authentic to the v0.8.0+ protocol.

## Pre-state

- Empty Arxlay workspace (zero models).
- User opens Claude Code in a fresh directory.
- MCP `arxlay` already added via `claude mcp add arxlay https://arxlay.com/mcp`.

## Conversation

**User:**
> Arxlay, let's describe the architecture.

**Skill (Phase 1 — Setup):**

[Internal: `list_models()` → returns `[]`]

> Got it. The workspace is empty — I'll create a new model for
> us. What should we call it? A short slug works
> (`twitter-clone`, `internal-billing`, etc.).

**User:**
> twitter-clone

**Skill:**

[Internal: `create_model({ name: "twitter-clone" })` →
returns `{ model_id: "abc123", ... }`]

[Internal: `list_artifacts(model_id)` → returns `[]`]

> Created. Empty model, no diagrams yet — I'll create one in the
> commit at the end.
>
> A few quick orientation questions before we draft:
>
> 1. Greenfield — building it now, or designing on paper?
> 2. Notation preference? Default is ArchiMate 3.2 + Arxlay
>    stdlib; that fits most service-oriented systems. (If you
>    have a strong preference for pure C4 or BPMN, say so.)
> 3. Do you want me to peek at any code in this directory?
>    There's no `package.json` or `Dockerfile`, so I'll assume
>    no code context for now.

**User:**
> Building it now. Default notation is fine. No code yet.

**Skill (Phase 2 — Discovery):**

> Tell me about the system in two or three lines. Just the
> mental model — services, users, key dependencies. I'll ask
> follow-ups.

**User:**
> Twitter clone. Web app for users. API gateway routes to two
> microservices: tweets-service (handles tweet CRUD) and
> users-service (handles auth/profile). Both share a Postgres
> database. Redis for rate limiting.

**Skill:**

> Inventory drafted:
>
> - **system:** `twitter-clone` (the whole product)
> - **microservices:** `tweets-service`, `users-service`,
>   `api-gateway`
> - **system (frontend):** `web-app`
> - **database:** `postgres-main`
> - **cache:** `redis-rate-limit`
> - **user:** `end-user`
>
> Type triage check: `api-gateway` reads more like a
> microservice (it has request-routing logic, not an aggregating
> business domain). `web-app` reads as a system (it's the
> user-facing product, not a service that other things call).
> Postgres and Redis are databases for our purposes.
>
> If that triage is wrong anywhere, correct me now — better to
> re-type before we draft relationships than to hit
> `relationship_not_allowed` at commit.

**User:**
> Looks right.

**Skill (Phase 3 — Proposal, slice 1 of 5: anchor):**

> **Anchor slice.** The root system is `twitter-clone`. The user
> (`end-user`) interacts with `web-app`. `web-app` lives inside
> `twitter-clone`.
>
> ```
> end-user → uses → web-app
> twitter-clone → contains → web-app
> ```
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 3, slice 2 of 5: services):**

> **Services slice.** The three microservices (`api-gateway`,
> `tweets-service`, `users-service`) all live inside
> `twitter-clone`. `web-app` calls `api-gateway`. `api-gateway`
> routes to the two backing services.
>
> ```
> twitter-clone → contains → api-gateway
> twitter-clone → contains → tweets-service
> twitter-clone → contains → users-service
> web-app → uses → api-gateway
> api-gateway → uses → tweets-service
> api-gateway → uses → users-service
> ```
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 3, slice 3 of 5: data):**

[Internal: brief cheat-sheet check — `microservice → database`
maps to `storesIn`, per Arxlay stdlib v1.0.0.]

> **Data slice.** Both microservices store in Postgres. Both
> use Redis for rate limiting.
>
> ```
> tweets-service → storesIn → postgres-main
> users-service  → storesIn → postgres-main
> tweets-service → uses → redis-rate-limit
> users-service  → uses → redis-rate-limit
> ```
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 3, slice 4 of 5: externals):**

> **Externals slice.** Anything outside the system to declare?
> Auth provider, payment processor, S3, third-party APIs?

**User:**
> Nothing yet.

**Skill (Phase 3, slice 5 of 5: cross-cutting):**

> **Cross-cutting slice.** Anything that contradicts a typical
> microservices pattern — shared library, sidecar, message
> queue, async fan-out? If not, we're complete.

**User:**
> Nothing.

**Skill (Phase 4 — Refinement / nesting check):**

> Quick scan: `twitter-clone` has 4 `contains` children
> (web-app, api-gateway, tweets-service, users-service).
>
> That's ≥3 contains from one source. **Should I draw the
> children visually nested inside `twitter-clone` on the canvas,
> or keep them flat (containment shown as arrows)?**
>
> Nested is usually clearer when there are 3+ children;
> flat is fine if you want each service to be visually
> independent. Your call.

**User:**
> Nested.

**Skill (Phase 5 — Approve and commit):**

> Final draft to commit:
>
> - 8 elements: `twitter-clone` (system), `web-app` (system),
>   `api-gateway` (microservice), `tweets-service`,
>   `users-service`, `postgres-main` (database),
>   `redis-rate-limit` (database), `end-user` (user).
> - 12 relationships (as drafted in slices 1-3).
> - 1 artifact (canvas): `twitter-clone — initial design —
>   2026-05-12`, with `placements[]` so the 4 children render
>   inside `twitter-clone`.
>
> Commit?

**User:**
> Yes.

**Skill:**

[Internal: `commit_changes({ ..., artifacts: { create: [{
notation: "archimate", artifact_type: "graph",
placements: [...] }] }})` → returns `{ version: 1,
artifacts_created: [{ canvas_url: "...", ... }] }`]

> Committed.
>
> - Version: 1
> - Canvas: https://arxlay.com/m/abc123/schemas/...
> - Layout: nested (4 children inside `twitter-clone`).
>
> Click through; the canvas auto-lays on first open. If
> anything looks off, edit in the UI or come back here with a
> change request.

## What this example shows

1. **Atomic commit** — exactly one `commit_changes` call after
   all five slices are confirmed.
2. **Type triage in Phase 2** — `api-gateway` re-typed as
   microservice before drafting, preventing a downstream
   allowance rejection.
3. **Allowance-validated relationship picker** — `microservice →
   database` uses `storesIn`, not `uses`.
4. **Nesting heuristic** — ≥3 contains from one source triggered
   a yes/no question before commit.
5. **`placements[]` commit form** — used because the user opted
   into nesting (not `place_all_in_batch: true`).
6. **Canvas URL** — read from the server response, not
   hand-constructed.

For the brownfield equivalent — adding to an existing model
instead of greenfield — see `examples/brownfield/dialogue.md`.

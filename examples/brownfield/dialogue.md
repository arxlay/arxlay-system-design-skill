# Brownfield example — extend an existing model

This example shows the skill operating on a **populated Arxlay
workspace** — adding new architecture incrementally without
disturbing what's already there.

The dialogue is paraphrased; the structure, MCP calls, and
decisions reflect the v0.8.0+ protocol.

## Pre-state

- Arxlay workspace with one existing model (`internal-platform`,
  23 elements, 2 diagrams).
- User has been using Arxlay for a few weeks; the model has been
  edited by hand and by previous `/arxlay` sessions.
- MCP `arxlay` already added.

## Conversation

**User:**
> Arxlay, let's describe the architecture.

**Skill (Phase 1 — Setup):**

[Internal: `list_models()` → returns
`[{ model_id: "abc123", name: "internal-platform", ... }]`]

[Internal: `query_elements({ model_id, limit: 0 })` → returns
element count + per-type breakdown]

[Internal: `list_artifacts(model_id)` → returns 2 artifacts:
`platform-overview`, `auth-flow`]

> Found one model: **`internal-platform`** (23 elements:
> 8 microservices, 5 databases, 3 users, 7 other; 2 diagrams).
> The model uses ArchiMate notation predominantly — I'll stay
> in that.
>
> What are we adding today? A new subsystem, a new connection,
> a new external integration?

**User:**
> Adding a billing subsystem. New service `billing-service`,
> uses a new Postgres database `billing-db`, integrates with
> Stripe externally.

**Skill (Phase 2 — Discovery):**

> Inventory for the new additions:
>
> - **microservice (new):** `billing-service`
> - **database (new):** `billing-db`
> - **external system (new):** `stripe`
>
> Triage:
> - `billing-service` is a service that other things will call →
>   `arxlay:microservice`.
> - `billing-db` is a Postgres → `arxlay:database`.
> - Stripe is third-party SaaS we depend on → external system,
>   represented as `arxlay:system` with `external: true` tag.
>
> Two questions before I draft:
>
> 1. Does `billing-service` belong inside an existing parent
>    (like a `commerce` or `revenue` system already in the
>    model), or is it a top-level addition under
>    `internal-platform`?
> 2. Which existing services will *call* `billing-service`?
>    I'll need that for the relationships.

**User:**
> Top-level under internal-platform. Called by `order-service`
> and `subscription-service` (both already in the model).

**Skill (Phase 3 — Proposal, slice 1 of 3: anchor):**

> **Anchor slice.** New elements join `internal-platform`. Stripe
> stays outside (external).
>
> ```
> internal-platform → contains → billing-service
> internal-platform → contains → billing-db
> ```
>
> Stripe is not contained — it's external.
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 3, slice 2 of 3: services):**

> **Services slice.** Two existing callers plus the external
> dependency.
>
> ```
> order-service        → uses → billing-service
> subscription-service → uses → billing-service
> billing-service      → uses → stripe   (external)
> ```
>
> Quick check: I'll verify the existing callers exist in the
> model before drafting.

[Internal: `query_elements({ model_id,
filter: { name: ["order-service", "subscription-service"] }})`
→ returns 2 matching microservices, confirms existence]

> Both callers found. Drafting the relationships above.
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 3, slice 3 of 3: data):**

> **Data slice.** Billing service stores in its own database.
>
> ```
> billing-service → storesIn → billing-db
> ```
>
> Confirm?

**User:**
> Yes.

**Skill (Phase 4 — Refinement / nesting check):**

> Scan: in this commit, `internal-platform` gains 2 contains
> children (billing-service, billing-db). Combined with existing
> contains, that's still under the nesting threshold for the
> root system — and any existing nesting on `internal-platform`
> is already set per its current canvas.
>
> I'll **not** modify the existing canvas (`platform-overview`,
> `auth-flow`). Instead I'll create a **new artifact** dedicated
> to the billing addition, so existing layouts stay untouched.
>
> Sound right?

**User:**
> Yes.

**Skill (Phase 5 — Approve and commit):**

> Final draft to commit:
>
> - 3 new elements: `billing-service` (microservice),
>   `billing-db` (database), `stripe` (external system).
> - 5 new relationships (as drafted in slices 1-3).
> - 1 new artifact (canvas): `billing-subsystem — 2026-05-12`,
>   containing only the new elements + linked existing callers
>   (`order-service`, `subscription-service`) for context.
> - **Existing artifacts** (`platform-overview`, `auth-flow`):
>   untouched.
>
> Commit?

**User:**
> Yes.

**Skill:**

[Internal: `commit_changes({ ..., artifacts: { create: [{
name: "billing-subsystem — 2026-05-12", notation: "archimate",
artifact_type: "graph", place_all_in_batch: true }] }})` →
returns `{ version: 14, artifacts_created: [...] }`]

> Committed.
>
> - Model version: 14 (was 13).
> - New canvas:
>   https://arxlay.com/m/abc123/schemas/billing-subsystem-...
> - Existing canvases unchanged.
>
> Layout will auto-compute on first open. If you want the
> billing components visualised inside `internal-platform`
> instead of as a standalone view, drag the elements into the
> `platform-overview` canvas — or come back here and ask me to
> create a nested variant.

## What this example shows

1. **Brownfield awareness in Phase 1** — skill reports element
   breakdown and existing artifacts *before* asking what's
   changing. The user sees the skill understands the model
   before being asked anything.
2. **No notation question** — the existing model has a dominant
   notation; skill stays in it without prompting.
3. **Existence verification mid-Phase 3** — `query_elements`
   confirms named callers exist before drafting relationships.
   This prevents "ghost references" at commit.
4. **No touching existing artifacts** — new commit creates a new
   canvas; existing layouts (`platform-overview`, `auth-flow`)
   stay intact. This is by design (anti-pattern #12).
5. **Atomic commit, single call** — even though 3 elements, 5
   relationships, and 1 new artifact all land together.

## Differences from the greenfield example

| Aspect | Greenfield | Brownfield |
|---|---|---|
| Phase 1 | `list_models` empty → `create_model` | `list_models` populated → pick model, report breakdown |
| Notation question | Asked (default offered) | Skipped (dominant notation already set) |
| Code peek | Optional, opt-in | Optional, usually skipped |
| Phase 2 | Full inventory | Only the *new* additions; verify references to existing |
| Phase 4 nesting | Decided fresh | Existing canvases not modified; new canvas may use placements |
| Phase 5 | One canvas created | New canvas created; existing untouched |

For the greenfield equivalent, see
`examples/greenfield/dialogue.md`.

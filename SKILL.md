---
name: arxlay-system-design-skill
description: Walks the user through describing their system architecture in Arxlay via conversation. Activates on the design trigger phrase ("Arxlay, design mode" / "Arxlay, let's describe the architecture" or Russian equivalent), or on the slash trigger "/arxlay describe-architecture" for first-run quickstart mode. Five-phase flow ending in an atomic commit through the Arxlay MCP server, plus a first-run quickstart for greenfield models.
version: 0.3.0
license: Apache-2.0
---

# Arxlay System Design Skill

You are an AI assistant helping a developer or architect describe their system in **Arxlay** — a typed-graph architecture tool — through a guided conversation. You commit the resulting model atomically via the Arxlay MCP server's `commit_changes` tool.

This skill is a **conversation pattern**. Follow it when the user explicitly invokes it; otherwise, do not impose this flow on regular questions.

## Section 1 — When to activate

Activate this skill **only** when the user opens with one of these triggers (or close paraphrases) at the **start** of a message. The Russian forms are listed because the team works in a Russian-speaking environment; both languages are first-class:

- "Arxlay, let's describe the architecture"
- "Arxlay, let's design"
- "Arxlay, design mode"
- "Arxlay, давай опишем архитектуру"
- "Arxlay, давай опишем систему"
- "/arxlay describe-architecture" (slash-style; activates **first-run mode**)
- "/arxlay let's describe architecture" (slash-style; activates **first-run mode**)

**Two modes:** the natural-language triggers above run the **full 5-phase flow** (Section 2), aimed at mature description with trade-offs. The slash-style triggers run **first-run quickstart mode** (Section 7), aimed at producing a first C4 Container diagram from an empty model in under five minutes. Pick by trigger style; do not negotiate it with the user.

**Do NOT activate** on read-style questions like:

- "Tell me about my architecture" → read-skill territory (use `query_elements`, answer in prose).
- "What's in model X?" → read-skill territory.
- "What services do I have?" → read-skill.

If both the read-skill and this design-skill are installed, the read-skill handles general inquiry; this skill activates **only** on the explicit design trigger. If unsure, ask: "Do you want to describe the architecture from scratch (design mode) or look at the existing one (query mode)?"

When activated, you commit to running the user through the 5 phases below. You do not bail mid-flow except on hard errors (MCP unreachable, no write permission, etc.).

## Section 2 — Conversation flow

The skill is a 5-phase pattern. Each phase has a goal, a "do" list, a "do NOT" list, and a sample dialog. You may iterate within and across phases — the order is a guideline, not a strict state machine.

### Phase 1 — Setup

**Goal:** confirm context before asking any architecture questions.

**Do:**

1. Verify MCP Arxlay is reachable. Call `list_models` — record the result.
2. Identify the active model. If multiple, ask the user which one. Record `model_id` and `version` (returned by `list_models`).
3. Check whether the model has elements: call `query_elements` with no name filter, default limit. If `total > 0`, you are in **brownfield**; otherwise **greenfield**.
4. **Brownfield only:** call `list_artifacts(model_id)` and record the existing diagrams (name, notation, artifact_type, element/relationship counts, created_via). You'll need this in Phase 5 to (a) pick an artifact name that doesn't collide and (b) tell the user up-front that you'll create a *new* diagram for this session rather than touching their existing layouts. Greenfield can skip this.
5. Check for code access: do native file-reading tools (Read, Glob, Grep, or equivalent) exist in your environment? If yes and the user is in a project, you can offer the **code path**; if not, default to **interview path**.
6. Decide the notation. Ask the user: Stdlib (microservices, modern apps), ArchiMate (enterprise context), or user-custom layer. The model's enabled types narrow this — only suggest types that exist in the resolved metamodel.
7. **Spit one consolidated reply** summarising what you discovered and the choice the user needs to make.

**Do NOT:**

- Skip `query_elements` in brownfield. Suggesting duplicates is the worst design-skill failure mode.
- Ask the user every detail upfront. One consolidated message, then one decision.
- Activate without a real model. If the user has zero models, respond: "You don't have any models in Arxlay yet — create one in the UI and come back."

**Sample dialog:**

> **Skill:** I see model "E-commerce platform" (version 7) with 0 elements and 0 diagrams — greenfield. I have access to the code in the current repository (Read/Glob/Grep tools available).
>
> Which vocabulary do you want to use — Stdlib (for microservice architectures) or ArchiMate (for enterprise context)? And — would you like to start with a code review or with a description in your own words?
>
> **User:** Stdlib. Take the code review path.

**Brownfield sample (with existing diagrams):**

> **Skill:** Model "Payment platform" (version 14) — 23 elements, 2 diagrams ("Domain data model — 2026-04-22", "Tenancy hierarchy"). I'll create a *new* diagram for this session so your existing layouts stay untouched. Code access: yes.
>
> Which area to extend, and Stdlib or ArchiMate?

### Phase 2 — Discovery

**Goal:** form a working hypothesis about the system's components, ownership, and pivotal trade-offs.

**Do (code path):**

- Glob for typical structural markers: `Dockerfile`, `docker-compose*.yml`, `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `requirements*.txt`, `pom.xml`, `**/README.md`, `services/**`, `apps/**`, `cmd/**`, `internal/**`.
- Read `README.md` first. It usually states the system's purpose in one paragraph.
- Read `Dockerfile`/`docker-compose*` to identify deployable units.
- Read package manifests for tech stack hints.
- Form a draft list: components (likely Stdlib `system`/`module`), data stores (`entity`), interfaces (`interface`), events (`event`).

**Do (interview path):**

Ask 2–3 questions per round, never the whole list at once:

1. Round 1: "What are you building? One sentence." (purpose) + "What are the main components or services?" (entities)
2. Round 2: "Where does the data live? Which databases, queues?" (data) + "Who owns what — are there explicit teams?" (ownership)
3. Round 3: "What are the key external dependencies?" (boundaries) + "Where are you currently uncertain about architecture decisions?" (pivots)

**Critical:** when a user states a non-trivial choice ("we use Postgres", "we're on gRPC"), **probe trade-offs in the moment**:

> **User:** We use Postgres for users.
>
> **Skill:** Did you consider MongoDB or DynamoDB? What drove the Postgres choice — joins, transactions, existing team expertise?

This converts trade-offs into first-class artefacts of the session, not post-hoc rationalisation.

**Do NOT:**

- Read `.env`, `secrets/`, `vendor/`, `node_modules/`, `target/`, `dist/`, `build/`, `.git/`. These are noise at best, secrets at worst.
- Ask the same question twice — keep state across rounds.
- Skip ownership questions in any team context.

### Phase 3 — Proposal

**Goal:** present the draft model in **prose**, with reasoning, trade-offs, and alternatives — never as raw JSON.

**Do:**

- For each proposed element: name, type (Stdlib or ArchiMate), one-line purpose, **Why / Trade-offs / Alternatives** (see Section 5).
- For each proposed relationship: source → target, type, why this relationship rather than another.
- Group related elements visually in the prose so the user can scan: "Core: Auth Service, Users DB. Platform: Payment Service, Orders DB. External: Stripe."
- End with: "Save or revise? If something looks wrong — be specific."

**Do NOT:**

- Show JSON or YAML to the user. They came here to avoid that.
- Commit at this stage. Phase 3 is draft-only — no MCP write call.
- Propose types not in the model's resolved metamodel. If you're unsure, ask the user or call `list_models`/`query_elements` again.
- Hide trade-offs in passive voice ("we considered MongoDB"). State which option, why rejected.

**Sample dialog:**

> **Skill:** Based on our conversation, I propose:
>
> **Auth Service** (Stdlib: `arxlay:system`)
> - **Why:** centralised authentication reduces duplication across services.
> - **Trade-offs:** tight coupling — every service depends on Auth; if Auth degrades, they all degrade.
> - **Alternatives:** per-service auth (rejected — scaling concerns as the team grows); JWT-only without a service (rejected — no centralised session management).
>
> **Users DB** (Stdlib: `arxlay:entity`)
> - **Why:** persistence for users and sessions.
> - **Trade-offs:** Postgres vs MongoDB — Postgres wins for joins across users/orgs/roles; MongoDB would be faster on pure reads but harder for reporting.
>
> **Relationships:**
> - Auth Service → Users DB (Stdlib: `arxlay:uses`)
>
> Save or revise?

### Phase 4 — Refinement

**Goal:** iterate on the draft until the user is satisfied.

**Do:**

- Treat each user comment as a discrete edit: add, remove, modify a single piece.
- For trade-off requests ("expand on this") — produce a fuller paragraph, not a bullet-point shuffle.
- Re-state the affected piece after each edit so the user confirms the change landed.
- Track the **draft state in conversation** — do NOT call MCP write tools mid-refinement.

**Do NOT:**

- Commit at the first user "ok". Wait for explicit approval phrases like "save it", "commit", "go", "ship it", or their Russian equivalents ("сохраняй", "залей это"). A bare "ok" is acknowledgement of your previous message, not a green light.
- Lose state across a long refinement loop. Re-summarise the current draft if the user gets lost ("here's what's in the draft right now: ...").
- Refuse legitimate edits. If the user wants to break a "best practice", document the trade-off and respect the choice.

**Sample dialog:**

> **User:** The trade-off for Payment Service is weak — write what we actually considered.
>
> **Skill:** Updated Payment Service:
> > **Trade-offs:** we picked Stripe (managed, fast onboarding, 2.9% fee) over an in-house gateway (more control but 6+ months of dev) and Adyen (lower fee at scale, but heavier integration overhead). Stripe is pragmatic for our current volume; we'll revisit when GMV grows.
> > **Alternatives:** in-house (rejected — time-to-market), Adyen (rejected — over-engineered for current volume).
>
> Anything else to revise, or shall we commit?

### Phase 5 — Approve & commit

**Goal:** atomically save the draft via `commit_changes` AND make it visible on a canvas the user can open.

**Do:**

1. Wait for an explicit approval phrase. Don't pre-commit.
2. Generate a fresh **UUIDv4** for `idempotency_key`. Store it — if the call needs retry, reuse the same key.
3. Generate a fresh UUIDv4 for `mcp_session_id` (per session, not per call — reuse across multi-step sessions if you do them).
4. Generate a UUIDv4 for each new element's `public_id`. The same UUID serves as the `source` or `target` reference in relationships within the same batch — there is no separate temp_id.
5. Generate **one** UUIDv4 for the artifact you'll create (next bullet). One artifact per design-mode session is the V1 contract.
6. Call `commit_changes` with:
   - `model_id` from Phase 1.
   - `expected_model_version` from Phase 1 (the version when you read the model).
   - `idempotency_key` and `client_metadata.{mcp_client_name, mcp_session_id}`.
   - `elements.create[]` with `public_id`, `type_id`, `name` (or full `fields.identity.name`), and any other fields.
   - `relationships.create[]` with `public_id`, `type_id`, `source`, `target`.
   - **`artifacts.create[]` (REQUIRED — exactly one entry):** without this the user lands on an empty canvas with the catalog populated but nothing to look at — the worst end-state. The entry is:
     - `public_id` from step 5.
     - `name` — short summary of *this* commit's slice (e.g. "Domain data model — 2026-05-09", "Auth surface", "Tenancy hierarchy"). In brownfield, make sure the name doesn't collide with the diagrams you saw in Phase 1; suffix " (2)" if it would.
     - `notation` — `"archimate"` for ArchiMate models, `"archimate"` or `"c4"` for stdlib models (stdlib types personality-map onto ArchiMate). Match the user's chosen vocabulary from Phase 1.
     - `artifact_type` — `"graph"` is the default and what you should use unless the user explicitly asked for a sequence diagram, BPMN flow, or whiteboard. The validator gates `bpmn`/`sequence`/`whiteboard` artifact types to matching notations.
     - `place_all_in_batch: true` — lets the server place every element + relationship from this commit on the new artifact without you spelling out the lists. This is the V1 default; only fall back to explicit `place_elements` / `place_relationships` if the user explicitly asks for a partial diagram.
   - The trade-off markdown goes in `fields.identity.description` of each element (see Section 5).
7. On success:
   - Read `artifacts_created[0].canvas_url` from the response — that's the direct link to the new diagram.
   - Reply with: counts (elements/relationships/artifacts), new `model_version`, and the **canvas URL as a clickable link**. Example: "Saved 8 elements + 9 relationships + 1 artifact 'Domain data model — 2026-05-09'. Model version: 14. Open it: <canvas_url>".
   - Tell the user the layout is auto-computed on first canvas view (ELK pass) — they may want to nudge nodes around.
8. On `version_conflict`: someone else committed since Phase 1. Re-read with `get_schema` or `list_models`, present the new state, offer to merge — do NOT auto-merge.
9. On `validation_failed`: walk back to Phase 4 with the specific `details[]` paths and explain the issue in user-friendly prose ("The type `microservice` is not enabled in this model — switch to `system`?"). Artifact-specific codes you may see: `artifact_notation_invalid`, `artifact_type_invalid`, `artifact_notation_type_mismatch`, `artifact_placement_conflict`, `artifact_placement_missing_endpoint`, `artifact_name_required`. All are friendly Phase A errors — surface them, propose the fix, retry with a fresh idempotency key.
10. On `permission_denied`: stop. The user lacks write access; tell them and end the session.

**Do NOT:**

- Loop on `version_conflict` automatically. Conflicts mean the user must decide.
- Cache the response client-side; the server's idempotency cache handles retry.
- Commit twice "to be safe". Idempotency lets you retry, but each fresh action should generate a new key.
- **Skip `artifacts.create[]`.** Without it, the commit succeeds but the user lands on an empty canvas — the failure mode this whole epic was opened to fix. One artifact, every commit.
- **Mix `place_all_in_batch: true` with explicit `place_elements` / `place_relationships`.** The validator rejects this combination as `artifact_placement_conflict`. Pick one form per artifact.
- **Touch existing artifacts.** V1 has no `place_elements_on_artifact` MCP tool — you can only create new diagrams. If the user wants their commit on an existing diagram, tell them they'll have to drag the new elements onto it via the UI; this MCP surface doesn't reach there yet (see Phase 2 of epic 015).

## Section 3 — Code awareness

Check for native code-reading tools at Phase 1. Different hosts expose different surfaces:

- **Claude Code (CLI):** Read, Glob, Grep, Bash. Full access.
- **Cursor:** the agent has direct project file access through the editor.
- **Claude Desktop:** depends on configured MCP servers — file access is not implicit.
- **Claude.ai web:** limited file access; usually no code path → interview path only.

If the tools exist, **prefer the code path** for greenfield or brownfield with code. Stop at the first user signal that they want pure interview ("let me just tell you"); do not insist.

**What to read:**

- Top-level `README.md` first.
- `Dockerfile`, `docker-compose*.yml` — deployable units, networks, ports.
- Package manifests: `package.json` (workspaces!), `go.mod`, `Cargo.toml`, `pyproject.toml`, `pom.xml`.
- `services/**/README.md`, `apps/**/README.md`, `cmd/**`.

**What to skip:**

- `node_modules/`, `vendor/`, `target/`, `dist/`, `build/`, `.git/`, `coverage/`.
- `.env`, `.env.*`, `secrets/`, `*.pem`, `*.key`. Even if `glob` returns them, don't `read`.
- Generated code (e.g. `*.pb.go`, `*.gen.ts`).

**When code reveals secrets:** if you accidentally read a file with credentials, do not echo them back. Tell the user "this file contains secrets — skipping", continue. Don't store them in the model `description`.

## Section 4 — Element creation patterns

Use Arxlay's typed metamodel. The active model has **enabled types** — only those are valid in `commit_changes`. The available layers are typically:

- **arxlay-stdlib** — modern microservices vocabulary: `arxlay:system`, `arxlay:module`, `arxlay:entity`, `arxlay:event`, `arxlay:interface`, `arxlay:team`, `arxlay:role`, `arxlay:department`, `arxlay:location`. Relationships: `arxlay:calls`, `arxlay:contains`, `arxlay:uses`, `arxlay:owns`, `arxlay:emits`, `arxlay:listens`, `arxlay:exposes`, `arxlay:invokes`, `arxlay:assignedTo`, `arxlay:leads`, `arxlay:locatedAt`, `arxlay:reportsTo`.
- **archimate-3.2** — full ArchiMate 3.2 element/relationship set for enterprise context.

**Choosing a type:**

- Public-facing application (auth, payments, web app) → `arxlay:system`.
- Internal logical component (auth-engine module inside Auth Service) → `arxlay:module`.
- Database / data store → `arxlay:entity` (the row-level concept). Don't confuse with ArchiMate `application-collaboration`.
- API endpoint or service boundary → `arxlay:interface`.
- Domain event (UserCreated, OrderShipped) → `arxlay:event`.
- Team or org unit → `arxlay:team` or `arxlay:department`.

**Choosing a relationship:**

- Network call API/RPC → `arxlay:calls`.
- Hierarchical containment (System contains Modules) → `arxlay:contains`.
- Reads/writes data → `arxlay:uses`.
- Owns the lifecycle → `arxlay:owns`.
- Producer of an event → `arxlay:emits` (target = event).
- Consumer of an event → `arxlay:listens` (source = event).

The metamodel's **allowance rules** restrict which (source-type, target-type, relationship-type) tuples are valid. If `commit_changes` returns `relationship_not_allowed`, the type is wrong — re-classify or swap to a more general relationship.

**`description` field** — this is where trade-offs live (see Section 5). Set via `fields.identity.description` (markdown). Keep elements under ~600 words of description; longer means it's actually two elements.

## Section 5 — Trade-off documentation format

Each non-trivial element gets a `description` with three sections in markdown:

```markdown
## Why

One paragraph explaining what this element does and why it exists at all.

## Trade-offs

What you give up by choosing this. State the give-ups explicitly: "tight
coupling between X and Y", "single point of failure", "harder to reason
about under load". Avoid empty trade-offs ("complexity") — name a concrete
operational, organisational, or evolutionary cost.

## Alternatives

What else was considered, and why each was rejected. Format:

- Option A — short reason for rejection.
- Option B — short reason.

If only one option was considered, say so explicitly: "No alternatives
considered; choice driven by the team's existing technology standard."
```

**Tone:** neutral, technical. No marketing fluff ("best-in-class", "enterprise-grade"). Decisions read as engineering trade-offs, not sales copy.

**When to skip:** if a trade-off is genuinely trivial (e.g. "Logger module — added because we need a logger"), omit the section rather than fabricating one. The skill should *ask the user* whether every trivially-named element triggers a trade-off interrogation.

**When the user objects:** if the user says "this isn't needed" — skip and move on. Document by default; don't gate.

## Section 6 — Anti-patterns

Things you must avoid:

1. **Committing without explicit approval.** "ok" is not "commit". Wait for "save it" / "commit" / "go" / "ship it" (or their Russian equivalents).
2. **Suggesting types absent from the model's enabled types.** If unsure, call `query_elements` (returns existing types in use) or ask.
3. **Ignoring brownfield context.** Always start with `query_elements` when `total > 0` from Phase 1; reference existing elements when relevant.
4. **Asking 10 questions at once.** Maximum 2–3 per round in interview path.
5. **Boilerplate trade-offs.** "We chose X because it's industry standard" is not a trade-off. Name the cost of not-X.
6. **Trade-offs in postscript.** Probe in Phase 2 when the user states a choice; don't wait until Phase 3 to invent rationale.
7. **Auto-retrying `version_conflict`.** Conflicts mean a human decision; report and ask.
8. **Reading secrets / `.env`.** Never. Even if listed by `glob`, skip.
9. **Splitting one concept across two sessions.** If the user runs out of time, save what's there with `commit_changes`, end the session, and tell them they can resume.
10. **Translating user terms.** If they say "Auth", don't rename to "AuthenticationService" for "consistency". Their words go in `name`.
11. **Committing without an artifact.** Every `commit_changes` from this skill MUST include exactly one `artifacts.create[]` entry with `place_all_in_batch: true`. Without it, the commit succeeds but the user lands on an empty canvas — the design-mode flow's worst end-state, and the reason epic 015 exists.
12. **Touching existing artifacts.** The MCP surface only creates new diagrams; there's no in-place add-elements-to-existing-artifact tool yet. In brownfield, your new artifact is a *new* diagram, not an extension of an existing one. Don't promise the user otherwise.

## Section 7 — First-run quickstart mode

This is a separate, fast path. Activate **only** on the slash-style triggers from Section 1:

- `/arxlay describe-architecture`
- `/arxlay let's describe architecture`

Goal: from trigger to an open canvas URL in **under five minutes**. The output is a C4-style Container diagram (5–12 elements, 5–15 relationships) drawn through the **arxlay-stdlib** types — Microservice, API, Database, External System, User. No trade-offs, no team assignments, no domain groupings — those come later via the full design-mode flow.

This mode runs differently from Section 2:

- **One** confirmation question. Not a dialog.
- Discovery is read-only file inspection — never any user questions until the confirmation.
- `description` is two short sentences (what it does + Inferred from), not the `## Why` / `## Trade-offs` / `## Alternatives` block from Section 5.
- The last reply ends with an explicit invitation back into design-mode for deeper work.

### 7.1 — When first-run mode applies

Activate first-run when **both** are true:

1. The user message starts with one of the slash triggers above.
2. The active model is empty: `query_elements` returns `total ≤ 4`.

If the model has 5 or more elements, **do not run first-run**. Tell the user: "I see N elements already in this model — first-run is for empty models. Switching to design-mode (Section 2) to extend the existing architecture instead." Then proceed with Section 2 Phase 1.

### 7.2 — Pre-flight (5 seconds, silent)

Before generating any output for the user:

1. Call `list_models` — verify MCP is reachable; record the active model's `model_id` and `expected_model_version`.
2. Call `query_elements` (no filter, default limit) — check `total`. Greenfield path requires `total ≤ 4`.
3. Confirm Read/Glob/Grep tools exist in your environment. If they do not (e.g. Claude Desktop with no file access), say so and switch to Section 2 Phase 2 interview path — first-run without code access is not a thing.

If any pre-flight step fails, surface it once and stop — don't keep trying.

### 7.3 — Discovery (1–2 minutes, no questions)

Run sources in this strict order. Stop early when the **first** source produces ≥ 5 candidate elements:

1. `docker-compose.yml` / `compose.yaml` (root or top-level subfolder).
2. `package.json`, `go.mod`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`.
3. `README.md` (top-level only).
4. `services/`, `apps/`, `packages/` directories — monorepo layout.
5. `.env.example` (never `.env`).
6. `Dockerfile` at root.

For full per-source markers and extraction rules, see `references/discovery.md`.

Build a working inventory with two lists:

- **Elements:** `name`, `type` (one of microservice / api / database / external-system / user), `inferred_from` (the file or files that produced this candidate).
- **Relationships:** `from`, `to`, `kind` (one of uses / storesIn / calls).

If discovery yielded fewer than 3 candidate elements, fall back to Section 2 Phase 2 interview path with five quick questions ("What are you building? Main components? Where does data live? External dependencies? Who's the user?") — first-run without enough signal is worse than a brief interview.

### 7.4 — Mapping signals to Stdlib types

Use this short table; broader rules live in `references/mapping.md`.

| Signal in repo                                         | Stdlib type             |
| ------------------------------------------------------ | ----------------------- |
| `services.X` in compose, application image             | `arxlay:microservice`   |
| `image: postgres\|mysql\|mongo\|redis\|cassandra\|...` | `arxlay:database`       |
| Direct dep on `stripe`/`@aws-sdk`/`openai`/`sendgrid`/`twilio` + import in code | `arxlay:external-system` |
| Frontend stack (`react`/`vue`/`svelte`/`next`)         | `arxlay:microservice` (tag: frontend) |
| Explicit gateway service (`api/`, `services/api`, `gateway`)   | `arxlay:api`    |
| README mentions end users / admins / customers         | `arxlay:user` (one or two, never more) |

Relationships:

- `depends_on` in compose, or `*_URL` env in another service → `arxlay:uses`.
- A service writing to a database → `arxlay:storesIn`.
- A clear event-driven flow (queue consumer) → `arxlay:calls` (rare in V1; skip if uncertain).

**Guardrail against false external systems:** add an `arxlay:external-system` only when the dependency is **direct** (top-level in the manifest) **and** there is at least one import or call site in the code (`grep`-able). Transitive deps in `node_modules` / `vendor/` do not count.

### 7.5 — One confirmation, in prose

Show the inventory as a short bulleted list — never as JSON, never as a table the user has to parse. Cap at 12 elements; if discovery found more, keep the most-grounded 12 (most files referenced) and mention the rest in one line. Then ask exactly one question.

Sample:

> I read `docker-compose.yml`, `services/auth/go.mod`, and `README.md`. I see:
>
> - **auth** (Microservice) — Go service, exposed on port 8080. Inferred from: `services/auth/Dockerfile + go.mod`.
> - **orders** (Microservice) — Go service. Inferred from: `services/orders/Dockerfile + go.mod`.
> - **postgres** (Database) — shared by auth and orders. Inferred from: `docker-compose.yml: services.postgres`.
> - **Stripe** (External System) — payment processing. Inferred from: `services/orders/go.mod: github.com/stripe/stripe-go`.
> - **Customer** (User) — end user of the platform. Inferred from: `README.md: "Customers can…"`.
>
> Connections: orders→auth (uses), auth→postgres (storesIn), orders→postgres (storesIn), orders→Stripe (uses), Customer→orders (uses).
>
> Save as your first diagram? (yes / no / tell me what to change in plain English)

Accept the answer:

- **yes / save it / go / ship it / сохраняй** → Section 7.6.
- **no** → reply with one sentence ("Got it — switch into design-mode and we'll build it from scratch with trade-offs?") and stop. The user can re-trigger with the natural-language phrase.
- **edit-text** ("drop Stripe, rename auth to identity") → apply the edit silently to the inventory, **re-show the updated list**, ask the same one question. Don't enter Section 2 — first-run stays one-shot.

### 7.6 — Commit

Single `commit_changes` call:

- `model_id` and `expected_model_version` from pre-flight.
- Fresh UUIDv4 `idempotency_key` and `mcp_session_id`.
- One `elements.create[]` entry per inventory item:
  - `type_id` from the mapping table.
  - `fields.identity.name` = the user-facing name (do not translate, do not "improve").
  - `fields.identity.description` = exactly two sentences:
    1. What this element does (one sentence — verb + object).
    2. `Inferred from: <source files separated by ", ">` (one sentence).

  Example: `"Backend service handling user authentication and JWT issuance.\n\nInferred from: services/auth/Dockerfile + go.mod"`. No `## Why` / `## Trade-offs` / `## Alternatives` blocks here — those belong to Section 5 / design-mode only.
- One `relationships.create[]` entry per inventory edge using the right `arxlay:uses` / `arxlay:storesIn` / `arxlay:calls`.
- **`artifacts.create[]` (REQUIRED — exactly one entry)** so the user lands on a populated canvas, not an empty one:
  - Fresh UUIDv4 `public_id`.
  - `name` = `"First-run snapshot — <repo-name> — <YYYY-MM-DD>"` where `<repo-name>` is the basename of the working directory the discovery ran in (e.g. `"First-run snapshot — arxlay — 2026-05-09"`).
  - `notation: "archimate"` (stdlib types personality-map onto ArchiMate; the artifact draws cleanly under the archimate notation).
  - `artifact_type: "graph"`.
  - `place_all_in_batch: true` — every discovered element + relationship lands on this artifact in one shot.

Handle errors per Section 2 Phase 5: `version_conflict` stops, `validation_failed` walks back to Section 7.5 with a re-shown inventory, `permission_denied` ends the session with a clear message. New artifact-specific validation codes (`artifact_*`) are surfaced in Phase 5 too — same handling.

### 7.7 — Show & next steps

After a successful commit, reply with **one** message containing:

1. The headline numbers: "Saved N elements, M relationships, and 1 diagram. Model version: X."
2. The **canvas URL** — read it from `artifacts_created[0].canvas_url` in the `commit_changes` response. **Always use this exact URL** (the server computes it from the model + artifact public_ids); never hand-construct one. Show it as a clickable Markdown link.
3. A one-liner about layout: "Layout auto-computes on first canvas open — feel free to drag nodes around."
4. An explicit bridge into design-mode:

   > Want to add trade-offs, owners, or break a service into modules? Say *"Arxlay, опиши X подробнее"* (or the English equivalent) and we'll go deeper.

This bridge is mandatory — first-run without it leaves the user wondering "and now what?".

Sample reply:

> Saved 7 elements, 6 relationships, and 1 diagram (`First-run snapshot — arxlay — 2026-05-09`). Model version: 1.
>
> Open it: <https://dev.arxlay.com/m/KSTBYFKfMBzA/schemas/2fbc153c-0eac-4e9b-9420-4f59c68fe842>
>
> Layout auto-computes on first canvas open — drag nodes around if anything overlaps.
>
> Want to add trade-offs, owners, or break a service into modules? Say *"Arxlay, давай опишем подробнее"* and we'll go deeper.

### 7.8 — Anti-patterns specific to first-run

In addition to the Section 6 anti-patterns:

1. **More than one question.** First-run asks exactly once, in Section 7.5. Anything beyond is design-mode in disguise.
2. **Inventing services.** If a service isn't in compose / a manifest / a top-level dir, do not invent one to "round out" the diagram. Five confirmed elements beat ten where half are guesses.
3. **JSON or YAML in user-facing replies.** The inventory is a bulleted list. The commit payload stays internal.
4. **Adding Teams, ADRs, Domains, Groupings.** These are non-goals here (epic 013 D-list). They live in design-mode.
5. **Component-level drilldown.** Don't list internal modules of a single service. Container-level is the contract — one node per deployable.
6. **Reading IaC manifests** (Kubernetes, Terraform, Pulumi). Not in scope for V1; ignore even if present.
7. **Echoing secrets.** If `.env.example` contains placeholder secret keys, list them generically ("uses Stripe — inferred from STRIPE_SECRET_KEY in .env.example"). Never the value, even when it's clearly a placeholder.
8. **Hand-constructing the canvas URL.** Use `artifacts_created[0].canvas_url` from the response verbatim. Never assemble `https://arxlay.com/m/<id>/canvas` or any other guess; the server's URL is the contract.
9. **Skipping `artifacts.create[]` in 7.6.** First-run with no artifact = empty canvas, exactly the failure mode this V1 was built to prevent. Every first-run commit creates exactly one artifact with `place_all_in_batch: true`.

### 7.9 — Failure modes

- **Pre-flight fails (no MCP).** Tell the user once: "Arxlay MCP isn't reachable — set it up via [arxlay.com/docs/integrations/quickstart-claude-desktop](https://arxlay.com/docs/integrations/quickstart-claude-desktop) and try again." Stop.
- **Pre-flight fails (model not empty).** Switch to design-mode (Section 2) silently — don't make the user re-trigger.
- **Discovery yielded < 3 elements.** Switch to Section 2 Phase 2 interview path with the five quick questions.
- **`commit_changes` returns `validation_failed`.** Re-show the inventory in Section 7.5 with the offending element marked, explain what the metamodel rejected (e.g. "External System → Database via `storesIn` is not allowed — switching to `uses`"), ask the same one question.

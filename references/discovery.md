# First-run discovery: per-source extraction rules

This file is consulted by the skill **only when** running first-run mode (Section 7 of SKILL.md). It expands the short discovery list with concrete markers, extraction rules, and explicit skip-cases.

The order of sources is deliberate. The first source that yields ≥ 5 candidate elements is enough — do not "round out" the picture by reading every source. The goal is the highest-grounded subset, not the largest.

## 1. `docker-compose.yml` / `compose.yaml`

The most authoritative source when present. Each top-level key under `services:` is a deployable unit.

**Markers:**

- Each `services.<name>` block → one candidate element. `name` becomes the element name verbatim — do not rename.
- `image: postgres:*`, `mysql:*`, `mariadb:*`, `mongo:*`, `redis:*`, `cassandra:*`, `cockroachdb:*`, `clickhouse:*`, `dynamodb-local:*` → `arxlay:database`. The image tag goes into description as the engine.
- `image: rabbitmq:*`, `kafka:*`, `nats:*`, `redpanda:*`, `pulsar:*` → **skip in V1** (message queues are deferred per epic 012 non-goals; revisit in V2).
- `image: nginx:*`, `traefik:*`, `envoy:*`, `caddy:*` → skip if no `command:` overrides; treat as edge infrastructure, not application.
- `image: *` for anything else (custom build or third-party app) → `arxlay:microservice`.
- `build:` block without `image:` → `arxlay:microservice`, name from the service key.

**Relationships:**

- `depends_on:` → `arxlay:uses` from depender to dependee.
- `environment:` keys matching `*_URL`, `*_HOST`, `*_DSN`, `*_ENDPOINT` whose value points to another service in the same compose file (`http://orders:8080`, `postgres://postgres:5432/...`) → `arxlay:uses` (or `arxlay:storesIn` if the target is a database).
- `networks:` alone don't imply a relationship — too noisy in real compose files.

**Skip:**

- Services with `profiles: [debug]`, `profiles: [test]` — environments, not architecture.
- One-shot helpers (`adminer`, `pgadmin`, `mailhog`) — they're tooling.
- `restart: "no"` jobs whose only purpose is migrations or seed data.

## 2. Manifests (`package.json`, `go.mod`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `pom.xml`)

In monorepos, search for these inside `services/*/`, `apps/*/`, `packages/*/` as well as the root.

**Markers (root manifest):**

- `name` field → root system name (use as a hint, not a separate element). Don't create a top-level System element in first-run.
- `dependencies` keys against the external-system table in `mapping.md`. Direct dep + import-call required (see guardrail below).

**Markers (per-service manifest in monorepo):**

- The directory name is the service name; the manifest confirms the language/runtime.
- A nested manifest with no parent ↔ service relationship in compose still counts as a microservice candidate, but its outbound relationships need another source (`.env`, hardcoded URLs).

**Skip:**

- `devDependencies` are not architecture signals.
- Lockfiles (`package-lock.json`, `go.sum`, `Cargo.lock`) — derived data, no new info.

**Guardrail against false external systems:** an external dep counts only when **both**:

1. It's in `dependencies` (top-level) — not transitively pulled in.
2. There's at least one import or call site you can `grep` for (e.g. `import stripe` or `from stripe import`).

If only (1) is true, the package is sitting unused — don't add a node.

## 3. `README.md` (top-level)

A README is the user's own description. Use it to *correct* what other sources told you, not as the primary source.

**Markers:**

- Sections titled `## Architecture`, `## Stack`, `## Tech stack`, `## Services`, `## Components` — usually contain a bulleted list. Cross-check each item against compose / manifests.
- Mentions of "user", "customer", "admin", "operator" → `arxlay:user`. Pick at most two; collapse "customers and admins" into one User element if the architecture treats them the same way.
- A pasted ASCII or Mermaid diagram block — not authoritative for first-run; if discovery is otherwise weak, mention it in the inventory note.

**Skip:**

- Marketing copy ("the leading platform for X") — extract no signals.
- Roadmap items ("we plan to add Stripe") — not yet in the architecture; don't fabricate.

## 4. Directory layout (`services/`, `apps/`, `packages/`)

Used when there's no compose and no top-level manifest tells you about boundaries.

**Markers:**

- One element per top-level child directory. Skip `shared/`, `common/`, `lib/`, `proto/`, `internal/` — those are usually shared code, not services.
- A subdirectory containing a `Dockerfile` or `main.go`/`main.py`/`server.ts`/`index.ts` → `arxlay:microservice`.
- A subdirectory whose name is `web`, `frontend`, `client`, `ui` → `arxlay:microservice` with the `frontend` tag.
- A subdirectory whose name is `api`, `gateway`, `bff` → `arxlay:api` if there are sibling microservices; otherwise `arxlay:microservice`.

**Skip:**

- `tests/`, `test/`, `spec/`, `__tests__/`, `e2e/` — testing infrastructure.
- `docs/`, `examples/`, `scripts/`, `tools/`, `bin/` — tooling.

## 5. `.env.example` (never `.env`)

A placeholder file is fair game; the real `.env` is off-limits regardless of what it might contain.

**Markers:**

- `DATABASE_URL`, `DB_HOST`, `POSTGRES_*` → confirm or add a `arxlay:database` if not already in inventory.
- `REDIS_URL`, `MEMCACHED_URL` → `arxlay:database` (cache is a kind of data store at this level).
- `STRIPE_SECRET_KEY`, `SENDGRID_API_KEY`, `OPENAI_API_KEY`, `AWS_ACCESS_KEY_ID` → `arxlay:external-system` if also confirmed by a manifest dep + import call.
- `<NAME>_URL=http://<other-svc>:<port>` → `arxlay:uses` from the current service to `<other-svc>`.

**Skip:** anything where the value side carries actual data (a URL with credentials, a JWT). Treat the file as a list of *keys*, never values.

## 6. `Dockerfile` (root or per-service)

Last source — typically used to attribute runtime / language for the description, not to discover new elements.

**Markers:**

- `FROM node:*`, `FROM golang:*`, `FROM python:*`, `FROM rust:*`, `FROM openjdk:*` → runtime tag in `description`.
- `EXPOSE <port>` → port goes into `description` ("exposed on port 8080").
- `ENTRYPOINT` / `CMD` running a non-server binary (cron, worker) → `arxlay:microservice` with note "background worker".

**Skip:** multi-stage builder stages — only the final stage matters.

## Heuristic fallbacks

Use these only when discovery would otherwise leave the diagram looking "plotholed":

- If compose / manifest gave a Frontend + Backend without any explicit relationship between them, draw `frontend → backend` as `arxlay:uses`. This is right roughly 99 % of the time.
- If a Backend exists with a Database in compose but no `depends_on` or `*_URL`, draw `backend → database` as `arxlay:storesIn`. Same likelihood.
- Don't fall back beyond these two — anything else risks fabricating connections.

## Stop conditions

- **First source ≥ 5 elements:** done with discovery, move to mapping.
- **All six sources tried, < 3 elements total:** abandon first-run, switch to Section 2 Phase 2 interview path.
- **Source produced obvious garbage** (binary file mistakenly read, generated proto definitions, a `vendor/` slip): skip and continue. Do not retry the same file.

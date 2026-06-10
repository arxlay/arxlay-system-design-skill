# First-run signal → Stdlib type mapping

This file is consulted by the skill **only when** running first-run mode (Section 7 of SKILL.md). It supplements the short table in Section 7.4 with edge cases and the relationship rubric.

## Stdlib types in scope

Five types from `arxlay-stdlib` are first-class for first-run. Anything outside this list is out of scope here — defer to design-mode (Section 2).

| Stdlib type             | ArchiMate base                  | Lucide icon  | When to pick                                                   |
| ----------------------- | ------------------------------- | ------------ | -------------------------------------------------------------- |
| `arxlay:microservice`   | `archimate:application-component` | `Box`      | Independently deployable application service                   |
| `arxlay:api`            | `archimate:application-interface` | `Plug`     | Programmatic interface a service publishes (REST/gRPC/GraphQL) |
| `arxlay:database`       | `archimate:data-object`           | `Database` | Any persistent or in-memory data store                         |
| `arxlay:external-system` | `archimate:application-component` | `Cloud`   | Third-party / out-of-scope system the architecture depends on  |
| `arxlay:user`           | `archimate:business-actor`        | `User`     | End user / actor interacting with the system                   |

## Signal table (extended)

| Signal                                                                                | Stdlib type                | Notes |
| ------------------------------------------------------------------------------------- | -------------------------- | ----- |
| `services.X` in compose with custom `image:` or `build:`                              | `arxlay:microservice`      | name = X (service key) |
| `image: postgres\|mysql\|mariadb\|cockroachdb\|cassandra\|...`                        | `arxlay:database`          | engine in description |
| `image: mongo\|dynamodb-local\|couchdb\|...`                                          | `arxlay:database`          | NoSQL — engine in description |
| `image: redis\|memcached\|...`                                                        | `arxlay:database`          | cache is a Database in C4 Container terms |
| `image: rabbitmq\|kafka\|nats\|redpanda\|pulsar\|...`                                 | **skip in V1**              | message queues deferred (epic 012 non-goal) |
| `image: nginx\|traefik\|envoy\|caddy\|haproxy` without `command:` overrides           | **skip**                    | edge infrastructure, not application |
| `image: vault\|consul\|etcd\|zookeeper\|...`                                          | **skip in V1**              | infra; revisit if user explicitly mentions |
| `image: elasticsearch\|opensearch\|meilisearch\|typesense\|...`                       | `arxlay:database`          | search index counts as a data store |
| `image: minio\|localstack\|...`                                                       | `arxlay:database`          | object storage |
| `image: prometheus\|grafana\|jaeger\|loki\|...`                                       | **skip**                    | observability stack — not user-facing architecture |
| Direct dep `stripe` + import in code                                                  | `arxlay:external-system`   | name "Stripe" |
| Direct dep `@aws-sdk/*` + import                                                      | `arxlay:external-system`   | name the concrete service ("S3", "SQS") via the package name |
| Direct dep `openai` + import                                                          | `arxlay:external-system`   | name "OpenAI API" |
| Direct dep `@sendgrid/mail`, `twilio`, `mailgun-js`, `postmark` + import              | `arxlay:external-system`   | name the vendor |
| Direct dep `auth0`, `clerk`, `firebase-admin`, `supabase-js` + import                 | `arxlay:external-system`   | name the vendor |
| Frontend stack (`react`, `vue`, `svelte`, `next`, `nuxt`, `solid-js`, `astro`)        | `arxlay:microservice`      | tag `frontend` in description; usually one element per app |
| Explicit `gateway`, `api-gateway`, `bff` directory or compose service                 | `arxlay:api`               | only when there are sibling microservices |
| README mentions end users / customers / admins                                        | `arxlay:user`              | one or two, never more |
| Mobile app reference (`mobile/`, `ios/`, `android/`, `react-native`)                  | `arxlay:microservice`      | tag `mobile` in description |

## Picking between Microservice / API / External System

This is the most common confusion source.

- **Microservice:** an application *we* built and deploy. Has source code in this repo.
- **API:** a *contract surface* of a service. Use only when there's a clear gateway/BFF layer fronting other microservices. In a typical 3–5 service repo, you usually do not need an API node.
- **External System:** a service we *do not control* — third-party SaaS or another team's system. Lives outside the repo.

When in doubt between Microservice and External System, ask: "Did we write this or pay someone for it?" If we wrote it (source in repo) → Microservice. If we use someone else's API (`stripe.com/api`, `api.openai.com`) → External System.

## Relationship rubric

Three relationships are in scope. Pick by *what the source-side does* to the target-side.

| Action                                                          | Relationship           |
| --------------------------------------------------------------- | ---------------------- |
| Service A makes an HTTP/gRPC/RPC call to service B (service → service) | `arxlay:calls`   |
| Frontend service calls the backend (service → service)          | `arxlay:calls`         |
| Service A reads or writes a Database (service → database)        | `arxlay:storesIn`      |
| Service A integrates with an External System (Stripe API, S3 SDK)| `arxlay:uses`          |
| End user / customer opens the app (user → microservice or api)   | `arxlay:uses`          |

`arxlay:calls` is the **service-to-service** edge (`microservice → microservice` — the metamodel does **not** allow `uses` between two services, only `calls`). `arxlay:uses` is for `user → microservice`/`api` and `microservice → external-system`. `arxlay:storesIn` is reserved for `service → database` (always that direction, never the reverse).

**Direction matters.** The consumer/caller is the *source*. When extracting from compose `depends_on:`, the block lives on the source — `A depends_on: [B]` means A is the source. Branch by B's type: B is another service → `A → calls → B`; B is a database → `A → storesIn → B`; B is third-party → `A → uses → B`.

## Anti-patterns specific to mapping

1. **Adding everything compose lists.** Compose is a deployment topology, not a domain model. Skip infra/observability/migration containers.
2. **Adding External Systems for transitive deps.** A package buried in `node_modules` you don't import anywhere is not architecture.
3. **Splitting one service into multiple types.** A backend service is one Microservice node, even if it exposes both REST and gRPC. The API node is reserved for actual gateway boundaries.
4. **Multiple User elements for slightly different actors.** `Customer`, `End user`, `Buyer`, `Visitor` collapse into one `User` element. Add a second only when access patterns differ materially (e.g. `User` and `Admin` with separate authentication paths).
5. **`storesIn` between two services.** `storesIn` is for data stores only — service-to-service is `arxlay:calls` (not `uses`, not `storesIn`). The metamodel will reject the wrong pair on commit.
6. **Renaming for "consistency".** If compose says `auth-svc`, the element is named `auth-svc`. Don't rewrite to `AuthService` or `Authentication Service`.

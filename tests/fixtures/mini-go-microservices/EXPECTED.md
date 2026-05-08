# Expected first-run output: mini-go-microservices

When the skill is run on this fixture with an empty Arxlay model, the inventory after Section 7.5 should match (or near-match — accept ≥ 80 % per epic 013) the following.

## Elements (4)

| Name       | Type                  | Inferred from                                                            |
| ---------- | --------------------- | ------------------------------------------------------------------------ |
| `auth`     | `arxlay:microservice` | `docker-compose.yml: services.auth` + `services/auth/Dockerfile + go.mod` |
| `orders`   | `arxlay:microservice` | `docker-compose.yml: services.orders` + `services/orders/Dockerfile + go.mod` |
| `postgres` | `arxlay:database`     | `docker-compose.yml: services.postgres (image: postgres:16-alpine)`       |
| `Buyer`    | `arxlay:user`         | `README.md: "Buyers interact with orders..."`                            |

## Relationships (5)

| Source     | Target     | Type              |
| ---------- | ---------- | ----------------- |
| `orders`   | `auth`     | `arxlay:uses`     |
| `auth`     | `postgres` | `arxlay:storesIn` |
| `orders`   | `postgres` | `arxlay:storesIn` |
| `Buyer`    | `orders`   | `arxlay:uses`     |

(That's 4 relationships — one per material connection. The skill may add a 5th `Buyer → auth` if it infers indirect login, but per anti-pattern "don't fabricate" it's preferable to skip.)

## Acceptable variations

- `Buyer` may be `User`, `Customer`, or `Buyers` — accept any reasonable form.
- `orders → auth` may be classified as `arxlay:calls` instead of `arxlay:uses` if the skill reads `AUTH_URL` and infers a call. Either is correct; `arxlay:uses` is the default per Section 7.4.
- The skill may name the service nodes from the directory (`services/auth` → `auth`) or from the compose key (`auth`) — they're identical here, so just check exact match.

## Out of acceptable

- A separate `arxlay:api` for either service — there's no gateway boundary.
- An External System for `gin-gonic/gin` or `pgx` or `golang-jwt` — those are framework deps, not third-party services.
- A node for the Docker volumes (none defined here, but skill should not invent them).
- Three User elements ("Buyer", "Admin", "Operator") — README only mentions one role.

## Manual smoke procedure

1. From this directory: `claude` (open Claude Code).
2. Make sure your Arxlay MCP connection points at a model with 0 elements.
3. Type `/arxlay describe-architecture`.
4. Compare the assistant's reply against the table above. Pass if ≥ 80 % of expected elements and relationships appear (4/4 elements, 3/4 relationships at minimum).
5. Confirm or decline per the skill's prompt — first-run is a one-shot.

# Expected first-run output: mini-node-monolith

When the skill is run on this fixture with an empty Arxlay model, the inventory after Section 7.5 should match (or near-match — accept ≥ 80 % per epic 013) the following.

## Elements (4)

| Name      | Type                     | Inferred from                                            |
| --------- | ------------------------ | -------------------------------------------------------- |
| `app`     | `arxlay:microservice`    | `docker-compose.yml: services.app` + `package.json`      |
| `postgres` | `arxlay:database`       | `docker-compose.yml: services.postgres (image: postgres:16-alpine)` |
| `Stripe`  | `arxlay:external-system` | `package.json: dependencies.stripe` + `.env.example: STRIPE_SECRET_KEY` |
| `Customer` | `arxlay:user`           | `README.md: "Customers browse..."`                        |

## Relationships (3)

| Source     | Target     | Type                |
| ---------- | ---------- | ------------------- |
| `app`      | `postgres` | `arxlay:storesIn`   |
| `app`      | `Stripe`   | `arxlay:uses`       |
| `Customer` | `app`      | `arxlay:uses`       |

## Acceptable variations

- The skill may collapse `Customer` into the diagram only when README is read; if it stops at compose alone (≥ 5 elements rule doesn't apply here, only 3 elements come from compose), it must continue to manifest + README before asking for confirmation.
- The skill may add the `frontend` tag to `app` if it inferred a UI from `express` + serving HTML, but should not split `app` into separate frontend/backend nodes — there isn't enough signal.
- `Stripe` may be named `stripe` (lowercase) — accept either form. `Customer` may be named `User` or `Customers` — accept any reasonable singular/plural form.

## Out of acceptable

- A separate `arxlay:api` element for the Express endpoints — there's only one service, no gateway.
- A `arxlay:external-system` for AWS / SendGrid / OpenAI — none are deps here. Adding any is a fabrication.
- A node for `shopfront-data` (the Docker volume) — volumes are not architecture elements.

## Manual smoke procedure

1. From this directory: `claude` (open Claude Code).
2. Make sure your Arxlay MCP connection points at a model with 0 elements.
3. Type `/arxlay describe-architecture`.
4. Compare the assistant's reply against the table above. Pass if ≥ 80 % of expected elements and relationships appear (4/4 elements, 2/3 relationships at minimum).
5. Confirm or decline per the skill's prompt — first-run is a one-shot.

# Widgets

Two-service Go monorepo: `auth` issues JWTs, `orders` calls `auth` to validate and persists order rows in Postgres. Both services share one Postgres database. Buyers interact with `orders` over HTTP.

## Services

- `services/auth` — authentication and JWT issuance.
- `services/orders` — order lifecycle (create, list, fulfil).

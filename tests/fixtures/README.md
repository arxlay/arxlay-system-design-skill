# Manual smoke fixtures for first-run

This directory contains minimum-viable greenfield repositories the first-run quickstart mode (SKILL.md Section 7) should be able to walk through end-to-end. Each fixture has an `EXPECTED.md` describing the model the skill should produce.

These are **not unit tests** — first-run is an LLM prompt, not deterministic code. The fixtures exist so we can manually re-run the skill on a known input and check the output against the expectation. Pass criterion is ≥ 80 % element/relationship match per epic 013 verification §7.

## Fixtures

- **mini-node-monolith/** — single Node.js/Express service + Postgres + Stripe. 4 elements, 3 relationships expected. Tests: compose + manifest + .env + README pipeline; external-system guardrail (Stripe is direct dep + would have an import in real code).
- **mini-go-microservices/** — two Go services with shared Postgres. 4 elements, 4 relationships expected. Tests: monorepo `services/*/` discovery, service-to-service `uses`, two services storing in one database.

## How to run

```sh
cd tests/fixtures/mini-node-monolith   # or mini-go-microservices
claude                                  # open Claude Code with this skill installed
# in the chat:
> /arxlay describe-architecture
```

Pre-requisites: Arxlay MCP connection set up, target model has 0 elements.

Compare the resulting inventory against the fixture's `EXPECTED.md`. Acceptable: ≥ 80 % match on both elements and relationships. Variations called out in `EXPECTED.md` ("Acceptable variations") are not failures.

## Adding a new fixture

When adding a fixture:

1. Make it the smallest possible repo that exercises a discovery rule (one new pattern per fixture).
2. Include only the files the skill is supposed to read: compose, manifest, README, .env.example, Dockerfile. No source code unless the rule needs it.
3. Write `EXPECTED.md` first — it forces you to think about what the skill *should* infer.
4. State exactly which `references/discovery.md` rule the fixture exercises in `EXPECTED.md`.

## What's deliberately out of scope

- Big real-world repos — those go to epic 014 (end-to-end smoke).
- Brownfield (model already populated) — first-run refuses to run, that's a separate path.
- IaC-only repos (K8s manifests, Terraform) — V2.

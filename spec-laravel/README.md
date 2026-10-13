# Manager Library — Laravel/React translation of the spec

This folder is the translation of the Manager Library spec to the challenge stack:

| Layer | Technology |
|---|---|
| Backend (`api` repo) | PHP 8.2, Laravel 12 |
| Frontend (`app` repo) | React 18 + Vite |
| Database | PostgreSQL 16, schema `library` |

## What is translated and what is not

Only the **technology decisions** are translated here: `constitution.md` (paths and tools), `plan.md`, `architecture.md`,
`arquitectura-panoramica.md`, `tasks.md` and `adr/`. Business content is not changed, extended or corrected.

The business documents are **not duplicated** in this folder. They live only in `../spec-python/`:

- `../spec-python/spec.md` — stories, acceptance criteria, business rules, NFRs.
- `../spec-python/data-model.md` — data model.
- `../spec-python/api-contract.md` — HTTP contract.

This gives a single source of truth: if a business rule changes, it changes in `spec-python/` only.

## Files

The structure is the same as `../spec-python/`:

| File | Content |
|---|---|
| `constitution.md` | Non-negotiable principles, with verification for each article |
| `plan.md` | Technical decisions D-01 to D-12, environment variables, test strategy, build order |
| `architecture.md` | Hexagonal architecture, Laravel folder layout, aggregates, lend flow, errors |
| `arquitectura-panoramica.md` | One-page overview of the system and its six repositories |
| `tasks.md` | Tasks T-01 to T-23 with dependencies and evidence |
| `adr/` | Architecture decision records 001 to 004 |

## Reading order

Same as the docs README (`../README.md`):

1. `constitution.md` — non-negotiable principles.
2. `../spec-python/spec.md` — what the system must do.
3. `plan.md`, `architecture.md` and `arquitectura-panoramica.md` — how it is built.
4. `../spec-python/data-model.md` and `../spec-python/api-contract.md` — data model and API contract.
5. `tasks.md` — what must be done and the evidence required.
6. `adr/` — architecture decisions, with alternatives and consequences.

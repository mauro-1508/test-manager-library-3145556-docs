# Tasks — Manager Library (Laravel/React)

Each task is finished when its **evidence** exists and can be run (article XII). Initial status of all of them: **pending**.

Story and criteria references (HU, CA, RN, RNF) point to [`../spec-python/spec.md`](../spec-python/spec.md). Route references point to
[`../spec-python/api-contract.md`](../spec-python/api-contract.md), and table references (§) to [`../spec-python/data-model.md`](../spec-python/data-model.md).

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` responds `{"status":"ok"}`; the container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes and the five seeded categories | T-01 | The queries of `data-model.md` §10 return what is documented; no column with `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Pest unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`withdraw`, `restock`, soft delete) | T-03 | Pest tests: never < 0, never > total |
| T-05 | Domain: `Loan` (`addItem`, `returnItem`, computed state, 14 days, max. 3) | T-04 | Pest tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (name normalization, valid role) | T-03 | `"Ana "` becomes `ana`; role `admin` sent through the API is rejected |
| T-07 | Ports and use cases for **authentication** (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | Ports and use cases for **books** (create, search, edit, delete, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | **Lend** use case (`LendBooksService`) with fakes | T-05, T-08 | HU-06 complete: limits, overdue, no copies |
| T-10 | **Return** use case and **overdue** use case | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | **Most borrowed report** use case and its read port | T-09 | HU-10; the old title is kept after renaming (ADR-004) |
| T-12 | Persistence adapter: book, member and user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + concurrency guard | T-12, T-09 | Two loans of the last copy: one 201, the other 409 (RNF-2) |
| T-14 | Security adapter: password hashing and JWT token | T-07 | The password never appears in logs or responses (RNF-3) |
| T-15 | Cover storage adapter (disk) | T-08 | PNG of 2 MB accepted; 3 MB → 422; no cover → `cover_key` null |
| T-16 | HTTP adapter: routes of [`../spec-python/api-contract.md`](../spec-python/api-contract.md), single error body, role authorization | T-10, T-11, T-14, T-15 | Contract test goes through all routes and all codes in §2 |
| T-17 | Composition and startup (initial admin from environment) | T-16 | The system starts with an empty database and the admin can log in |
| T-18 | Move the **domain-only** invariants down to the database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` through `psql` that violates them fails; `data-model.md` §4 is updated |
| T-19 | Architecture tests (articles I, II and III) | T-17 | Pest `arch()` tests pass, and fail when a forbidden import is introduced |
| T-20 | `infra`: `docker compose up` starts the empty engine, the API and the frontend | T-17 | One command; the database starts without tables until the migrations are run |
| T-21 | `tool`: demo seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-17 | After running it, `/loans/overdue` returns 3 lines and the report has rows |
| T-22 | `app`: login, catalogue, members, lend, return, overdue, report | T-16 | Manual end-to-end walkthrough at the presentation fair |
| T-23 | `page`: static public presentation site | — | Opens without the API; correct on mobile and desktop |

## Definition of done for the test

Whoever reaches T-17 with evidence has the **core** done. T-18 to T-23 are what separates good work from outstanding work. Performance
is assessed on SDD, not on the number of tasks: one task with evidence is worth more than three half-finished ones.

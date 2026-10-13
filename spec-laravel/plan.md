# Technical plan — Manager Library (Laravel/React)

How [`../spec-python/spec.md`](../spec-python/spec.md) and [`constitution.md`](constitution.md) are built on the challenge stack. The
schema is **not** described here: it lives in [`../spec-python/data-model.md`](../spec-python/data-model.md). The shape of the system is in
[`architecture.md`](architecture.md).

## 1. Technical decisions

| # | Decision | Choice (Laravel/React) | Why |
|---|---|---|---|
| D-01 | Language and runtime | API: PHP 8.2, Laravel 12. App: React 18 + Vite | Challenge stack |
| D-02 | HTTP routing | Laravel HTTP routing (`routes/api.php`, prefix `/api/v1`); controllers and Form Requests in `app/Adapters/Inbound/Http/` | Contract in [`../spec-python/api-contract.md`](../spec-python/api-contract.md) |
| D-03 | Persistence | Laravel migrations on PostgreSQL 16, schema `library` (created by the first migration; connection `search_path` is `library,public`, so Laravel's own `migrations` table lives in `public` and the six domain tables in `library`). Eloquent and Query Builder only inside `app/Adapters/Outbound/Persistence/` | The schema belongs to the migrations (ADR-001) |
| D-04 | Concurrency | Optimistic, per [ADR-002](adr/adr-002-optimistic-concurrency.md): the book write is a conditional `UPDATE` that only applies if the row still holds the values that were read; zero affected rows → `BOOK_NOT_AVAILABLE` (409); last barrier is the `ck_book_available_range` `CHECK` | Last copy (RNF-2) |
| D-05 | Dates | UTC instants (`timestampTz`); the clock is a port (`Clock`) | Test due dates without waiting 14 days |
| D-06 | Report | One aggregate query behind a read port (`ReportReader`) ([ADR-004](adr/adr-004-aggregated-frozen-report.md)) | Stable and cheap |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: live inside their owner's row, no table | No identity |
| D-08 | Covers | Binary in external storage (Laravel filesystem: local disk in development, S3 driver in production) behind `CoverStorage`; the database stores only `cover_key` | Opaque key, never a path |
| D-09 | Passwords | Argon2id (PHP `password_hash` with `PASSWORD_ARGON2ID`) behind `PasswordHasher`; the domain never sees the plain password | Article IX |
| D-10 | Startup | An Artisan command run at container start creates the initial admin from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot produce the hash |
| D-11 | Authentication | JWT HS256, 8 hours, key in `JWT_SECRET`, via `firebase/php-jwt` inside the security adapter behind `TokenIssuer` | Simple and stateless |
| D-12 | Tests | Pest; fakes for the ports; real PostgreSQL only in integration tests | Test pyramid |

Notes on D-03: Laravel's default migrations (`users`, `cache`, `jobs`, `sessions`) are removed. Only the six tables of
`../spec-python/data-model.md` exist, plus Laravel's own `migrations` bookkeeping table. Sessions and cache do not use the database.

## 2. Configuration per environment

| Variable | Required | Example |
|---|---|---|
| `DATABASE_URL` | yes | `postgresql://library:library@db:5432/library` |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | set in `.env`, never in the repository |
| `COVERS_DIR` | no | `./var/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

The same variable names as the original spec are used; they are mapped explicitly in `config/database.php` (`DATABASE_URL`), `config/filesystems.php` (`COVERS_DIR`) and `config/cors.php` (`CORS_ORIGIN`). The React app reads the API base URL from `VITE_API_URL`; this variable is used by the `app` repository.

## 3. Test strategy

| Level | What it tests | Against what |
|---|---|---|
| Domain | Invariants: maximum 3, due in 14 days, available within range | Nothing: pure objects |
| Application | Use cases: lend, return, report | Fakes of the ports |
| Integration | Repositories and the report query | Real PostgreSQL |
| Contract | Single error body and status codes | The running API |
| Architecture | Articles I, II and III | Pest architecture tests (`arch()`) in `tests/Architecture/` |
| Concurrency | Two loans of the last copy | Real PostgreSQL |

## 4. Build order

Skeleton and health → initial migration with seed data → domain → use cases with fakes → adapters → API → contract and concurrency tests.
The detail, with evidence, is in [`tasks.md`](tasks.md).

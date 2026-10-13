# Constitution — Manager Library (Laravel/React)

Non-negotiable principles of the project. They are not style or preference: each article states **how it is verified**, because a
principle that nobody can check is just an intention. A change that violates an article is rejected without discussing the rest.

Business references are in `../spec-python/` (`spec.md`, `data-model.md`, `api-contract.md`).

---

## Article I — The domain depends on nothing

`app/Domain/` does not import any other namespace of the project nor third-party code (no Laravel `Illuminate\*`, no Eloquent, no
HTTP or validation packages). Only plain PHP: `readonly` classes, `DateTimeImmutable`, enums, native exceptions.

**Why:** it is the only way to test the invariants without a database, without a server and without a container, and to change the
persistence layer without touching the rules.
**Verification:** Pest architecture tests (`arch()` expectations) in `tests/Architecture/` forbid the domain from depending on
`App\Application`, `App\Adapters` or `App\Bootstrap`, and forbid any `Illuminate\` or Eloquent import in `app/Domain/`.

## Article II — The core declares its ports; the outside world implements them

The application layer only knows the domain. Everything it needs from outside (repositories, clock, password hashing, cover storage,
token generation) is expressed as a **port that it defines itself**. No framework or ORM type appears in the signature of a port.

**Verification:** a Pest architecture test in `tests/Architecture/` prevents `App\Application` from importing `App\Adapters`. Application
tests run with fakes and never reference `App\Adapters`.

## Article III — A single composition point

A single class (`app/Bootstrap/Composition/PortBindings.php`, a Laravel service provider) is where a port meets its adapter. Outside it,
no infrastructure is instantiated.

**Verification:** no class of `App\Adapters` is instantiated outside that file; checked with a Pest architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (local disk → S3) does not touch domain or application | The new adapter only adds one line in `PortBindings` |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes against the fake and the real adapter |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer leaves methods unused |
| **DIP** | The port is defined by the consumer | Ports live in the application namespace (`app/Application/Ports/Outbound/`) |

## Article V — The schema is owned by the migrations

The `infra` repository starts an **empty** engine. All DDL (tables, indexes, constraints, extensions and seed data) lives in the
application's Laravel migrations in the `api` repository. See [ADR-001](adr/adr-001-schema-ownership.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — The invariants the database can enforce, the database enforces

A rule that lives only in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`,
FK) is declared there. What is not enforced is marked **pending** in [`../spec-python/data-model.md`](../spec-python/data-model.md); it is
not promised.
**Verification:** the queries of `../spec-python/data-model.md` §10 match what the document declares.

## Article VII — Derived data is computed, not stored

The state of a loan (`open`, `overdue`, `closed`) and the days late are computed. There is no column for them.
**Verification:** `../spec-python/data-model.md` does not list any `status` or `days_late` column.

## Article VIII — Frozen values do not follow the catalogue

The loan line stores title, ISBN and category as of the loan instant. No report reads them from the live catalogue.
**Verification:** a test renames a book after lending it and checks that the report keeps the old title ([ADR-004](adr/adr-004-aggregated-frozen-report.md)).

## Article IX — No secret enters the repository or a log

No passwords, no precomputed hashes, no token key. Every secret comes from the environment. The real `.env` is in `.gitignore`; only
`.env.example` is versioned.
**Verification:** `gitleaks` or a pattern `grep` in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The engine wins

If the data document and the database disagree, the document is broken and the document is corrected.

## Article XI — Language

Code, columns and routes in English. Documentation and user-facing messages in Spanish in the business spec.

For this challenge, the team writes **repository documentation in English** (every README and every file in `spec-laravel/`). User-facing
messages (the API error `message` field and UI texts) **stay in Spanish**.

## Article XII — Nothing is taken as done without evidence

A task in [`tasks.md`](tasks.md) is finished when its **evidence** exists and can be run, not when it "is already coded".

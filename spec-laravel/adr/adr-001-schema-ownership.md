# ADR-001 · The schema is owned by the application's migrations

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could put the DDL in a startup script of the container and
  someone else in the application's migrations; with two owners, nobody knows which schema runs in production.
- **Decision:** **all DDL lives in the application's Laravel migrations** (tables, indexes, constraints and category seed data).
  `infra` only creates the empty engine, the user and the volume.
- **Discarded alternatives:** SQL scripts in `infra/db/init/` (second source of truth); ORM "create tables on boot" mode, such as
  Eloquent-based schema creation at startup (no history, no rollback).
- **Consequences:** a PostgreSQL extension or a new index is created in the **same** migration that uses it. A schema changed by hand in
  the engine is a defect, not a variant.
- **How it is verified:** `infra` contains no `CREATE TABLE`; the queries of [`../../spec-python/data-model.md`](../../spec-python/data-model.md)
  §10 match the migrations.

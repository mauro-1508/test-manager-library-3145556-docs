# Architecture in one page — Manager Library (Laravel/React)

A five-minute read before opening the rest. The detail is in [`architecture.md`](architecture.md).

## The system at a glance

```mermaid
flowchart TB
    U["Librarian / Admin"] --> APP["app (React + Vite)<br/>frontend"]
    V["Visitor"] --> PAGE["page<br/>static public site"]
    APP -->|"HTTP /api/v1"| API["api (Laravel)<br/>backend"]
    API --> PG[("PostgreSQL 16<br/>schema library")]
    API --> COV[("Covers<br/>disk / S3")]
    TOOL["tool<br/>demo data seeder"] -->|"uses the API"| API
    INFRA["infra<br/>containers and empty engine"] -.-> PG
    INFRA -.-> API
```

## The six repositories and their role

| Repo | Role | Tested with |
|---|---|---|
| `docs` | The specification | That this reading is enough to build |
| `api` | Business rules and HTTP contract (Laravel) | Domain, application, contract and concurrency tests |
| `app` | Operation screens (React + Vite) | An end-to-end walkthrough: login → lend → return → report |
| `page` | Public presentation, does not call the API | That it looks right on mobile and desktop |
| `infra` | `docker compose up` and the empty database | That it starts with a single command |
| `tool` | Demo data (books, members, loans) | That the presentation fair has real data |

## The five ideas behind the design

1. **A loan is an immutable fact**; only `returned_at` of a line changes.
2. **Copies are taken and returned within the same operation** that creates or closes the line.
3. **The state is computed**: `open`, `overdue` or `closed` are not in the database.
4. **Frozen data does not follow the catalogue**: the October report does not change because a book is renamed in November.
5. **The database is the last barrier**: if the copies `CHECK` fires, something wrote outside the system.

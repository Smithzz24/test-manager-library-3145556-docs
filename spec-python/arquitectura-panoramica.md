# Architecture at a glance — Manager Library

Read this in five minutes before opening the rest. Details are in [`architecture.md`](architecture.md).

## The system at a glance

```mermaid
flowchart TB
    U["Librarian / Admin"] --> APP["app (React)<br/>frontend"]
    V["Visitor"] --> PAGE["page<br/>static public site"]
    APP -->|"HTTP /api/v1"| API["api<br/>Python backend (Laravel in the challenge)"]
    API --> PG[("PostgreSQL 16<br/>library schema")]
    API --> COV[("Covers<br/>disk / S3")]
    TOOL["tool<br/>demo data seeder"] -->|"uses the API"| API
    INFRA["infra<br/>containers and empty database engine"] -.-> PG
    INFRA -.-> API
```

## The six repositories and their roles

| Repository | Role | Verified by |
|---|---|---|
| `docs` | The specification | Whether it is sufficient to build the system |
| `api` | Business rules and HTTP contract | Domain, application, contract, and concurrency tests |
| `app` | Operational screens | End-to-end flow: login → lend → return → report |
| `page` | Public presentation, with no API calls | Appearance on mobile and desktop |
| `infra` | `docker compose up` and the empty database | Starts with one command |
| `tool` | Demonstration data (books, members, loans) | The demo has realistic data |

## Five ideas behind the design

1. **A loan is an immutable fact**; only a loan item's `returned_at` can change.
2. **Copies are deducted and returned in the same operation** that creates or closes the item.
3. **Status is calculated**: `open`, `overdue`, or `closed` are not stored in the database.
4. **Frozen values do not follow the catalog**: October's report does not change because a book is renamed in November.
5. **The database is the final safeguard**: if the copies `CHECK` fails, something wrote outside the system.

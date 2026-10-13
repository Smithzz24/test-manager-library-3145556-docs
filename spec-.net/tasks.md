# Tasks — Manager Library (.NET)

Each task is complete when its **evidence** exists and can be executed (Article XII). Initial status for all tasks: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` returns `{"status":"ok"}`; container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes, and the five seeded categories | T-01 | Queries in `data-model.md` §10 return the documented results; no column has a `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` value objects | T-01 | Unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, soft delete) | T-03 | Tests: copies never < 0 or > total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, calculated status, 14 days, max 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (username normalization, valid role) | T-03 | `"Ana "` becomes `ana`; the API rejects the `admin` role |
| T-07 | Authentication ports and use cases (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | Book use cases (create, search, edit, withdraw, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | Lending use case (`LendBooksService`) with fakes | T-05, T-08 | HU-06 end to end: limits, overdue items, unavailable copies |
| T-10 | Return and overdue use cases | T-09 | HU-07 and HU-09; returning twice yields `ITEM_ALREADY_RETURNED` |
| T-11 | Most-borrowed report use case and read port | T-09 | HU-10; old title remains after rename (ADR-004) |
| T-12 | Persistence adapters: book, member, and user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + version token | T-12, T-09 | Two loans for the last copy: one 201, one 409 (RNF-2) |
| T-14 | Security adapter: password hashing and JWT | T-07 | Password never appears in logs or responses (RNF-3) |
| T-15 | Cover adapter (disk) | T-08 | Upload a 2 MB PNG successfully; 3 MB → 422; no cover → `cover_key` is null |
| T-16 | HTTP adapter: routes in [`api-contract.md`](api-contract.md), common error body, role authorization | T-10, T-11, T-14, T-15 | Contract test exercises every route and every code in §2 |
| T-17 | Composition and startup (initial admin from environment) | T-16 | System starts with an empty database and allows login as admin |
| T-18 | Move domain-only invariants into the database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` through `psql` that violates them fails; update `data-model.md` §4 |
| T-19 | Architecture tests (Articles I, II, and III) | T-17 | NetArchTest passes and fails when a forbidden dependency is introduced |
| T-20 | `infra`: `docker compose up` starts database engine, API, and frontend | T-17 | One command; database starts without tables until migrations run |
| T-21 | `tool`: demonstration seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-17 | After running it, `/loans/overdue` returns 3 lines and the report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue, report | T-16 | Manual end-to-end walkthrough at the showcase |
| T-23 | `page`: static public presentation site | — | Opens without the API; works on mobile and desktop |

## Definition of done for the assessment

Reaching T-17 with evidence means the **core** is done. T-18 through T-23 distinguish good work from outstanding work. Performance with SDD is evaluated, not the number of tasks: one task with evidence is worth more than three half-finished tasks.

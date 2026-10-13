# Constitution — Manager Library (Python)

The project's non-negotiable principles. These are not style choices or preferences: each article includes **how it is verified**, because a principle nobody can check is only an intention. A change that violates an article is rejected regardless of the rest.

---

## Article I — The domain depends on nothing

`src/library/domain/` imports no other project module or third-party library: only the standard library (`dataclass(frozen=True)`, `datetime`, `typing`); no SQLAlchemy, FastAPI, or Pydantic.

**Why:** this is what allows invariants to be tested without a database, server, or container, and allows the ORM to be changed without touching rules.
**Verification:** an `import-linter` contract and a test in `tests/architecture/` prohibit the domain from depending on `application`, `adapters`, or `bootstrap`.

## Article II — The core defines its ports; the outside implements them

The application layer knows only the domain. Everything it needs from the outside (repositories, clock, password hashing, cover storage, token generation) is expressed as a **port defined by the application itself**. No framework or ORM type appears in a port signature.

**Verification:** an `import-linter` contract and a test in `tests/architecture/` prevent `application` from importing `adapters`. Application tests run with fakes and without referencing `adapters`.

## Article III — One composition point

Only one module (`src/library/bootstrap/composition/port_bindings.py`) connects a port to its adapter. Infrastructure is not instantiated elsewhere.
**Verification:** no `adapters` class is instantiated outside that module; an architecture test checks this.

## Article IV — SOLID is not decoration

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | Split a service that has two reasons to change |
| **OCP** | Changing cover storage (disk → S3) does not touch the domain or application | A new adapter adds only one line to composition |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes with both fake and real |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer has unused methods |
| **DIP** | The consumer defines the port | Ports live in the application package |

## Article V — Migrations own the schema

The `infra` repository starts an **empty** database engine. All DDL (tables, indexes, constraints, extensions, and seed data) lives in the application's migrations (Alembic revisions). See [ADR-001](adr/adr-001-propiedad-del-esquema.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — The database enforces every invariant it can

A rule that exists only in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`, FK) is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are calculated, not stored

Loan status (`open`, `overdue`, `closed`) and days late are calculated. There are no columns for them.
**Verification:** `data-model.md` lists no `status` or `days_late` column.

## Article VIII — Frozen values do not follow the catalog

A loan item stores the title, ISBN, and category from the time of the loan. No report reads those values from the live catalog.
**Verification:** a test renames a book after it has been lent and confirms that the report preserves the old title ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)).

## Article IX — No secrets enter the repository or logs

No passwords, precomputed hashes, or token key. All secrets come from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or a pattern grep in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The database wins

If the data document and the database disagree, the document is broken and must be corrected.

## Article XI — Language

Code, columns, and routes are in English. Documentation and user-facing messages are in Spanish.

**Translation note:** the sentence above preserves the original Article XI. The user requested English documentation for this fork and reconfirmed that choice during the source review. This is a documentation-language deviation, not evidence that the project owner has waived Article XI. User-facing messages remain in Spanish; no product rule changes.

## Article XII — Nothing is complete without evidence

A task in [`tasks.md`](tasks.md) is complete when its **evidence** exists and can be executed, not when it is “already programmed.”

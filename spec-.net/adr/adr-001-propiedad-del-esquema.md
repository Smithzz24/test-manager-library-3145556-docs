# ADR-001 · The application migrations own the schema

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could put DDL in a container startup script while someone else puts it in application migrations; with two owners, nobody knows which schema is running in production.
- **Decision:** **all DDL belongs in the application migrations** (tables, indexes, constraints, and seed data). `infra` creates only the empty database engine, user, and volume.
- **Rejected alternatives:** SQL scripts in `infra/db/init/` (a second source of truth); ORM auto-schema creation at startup (no history or rollback).
- **Consequences:** a PostgreSQL extension or a new index is created in the same migration that uses it. A schema changed manually in the database is a defect, not a variant.
- **Verification:** `infra` contains no `CREATE TABLE`; the queries in `data-model.md` §10 match the migrations.

# Laravel and React implementation guide

This guide separates product requirements from technology choices in the Python and .NET reference specifications. It records how to approach the challenge stack without changing the product behavior or inventing unspecified decisions.

## Confirmed challenge stack

| Layer | Required challenge technology |
|---|---|
| Frontend | React |
| Backend | PHP with Laravel |

The repositories have separate roles: `api` contains the Laravel backend, `app` contains the React frontend, `infra` starts the empty database engine and application containers, `tool` seeds demonstration data through the API, and `page` is the static public presentation site. The `docs` repository is the specification.

## Requirements to preserve

Regardless of implementation technology, preserve:

- The actors, permissions, scope, user stories, and acceptance criteria in `spec-python/spec.md` and `spec-.net/spec.md`.
- The routes, field names, payload shapes, status codes, error envelope, authentication behavior, and role rules in both `api-contract.md` files. The API contract is language-independent.
- The six entities, relationships, constraints, frozen loan-item values, soft deletion, and calculated loan status in both `data-model.md` files.
- The four accepted architecture decisions in `spec-python/adr/` and `spec-.net/adr/`: migrations own the schema; optimistic concurrency protects copy counts; books are soft-deleted; the most-borrowed report uses frozen values.
- The measurable non-functional requirements, including the last-copy concurrency test and the search target. Where the measurement procedure is unspecified, do not claim the exact target has been verified.
- Spanish user-facing error messages and English code, columns, and routes.

## Technology-specific reference decisions

These are differences between the reference stacks, not differences in product behavior:

| Concern | Python reference | .NET reference | Challenge translation |
|---|---|---|---|
| HTTP framework | FastAPI | ASP.NET Core Minimal API | Implement the language-independent contract with Laravel |
| Persistence and migrations | SQLAlchemy 2 + Alembic | EF Core 8 migrations | Use Laravel's persistence and migration mechanisms; keep migrations as the only owner of DDL |
| Password hashing | Argon2id | BCrypt, work factor 12 | Both require a password hash and prohibit plaintext or hash disclosure. The challenge documents do not select a Laravel hashing algorithm; do not treat either reference choice as a product requirement. |
| Automated tests | pytest | xUnit | Select tools compatible with PHP/JavaScript; preserve the evidence and behaviors the tests must verify |
| Architecture tests | `import-linter` | NetArchTest | Preserve dependency direction and separation where practical; the challenge documents do not prescribe a Laravel architecture-testing package |

The two references also agree on PostgreSQL 16, a signed HS256 JWT valid for eight hours, UTC instants, a version-token concurrency decision, and an external cover store whose database value is an opaque key. These requirements or decisions must not be silently replaced with framework defaults. Where their physical implementation is not fully specified, see the open items below.

## Mapping principles, not folder names

The reference architecture uses ports and adapters: business rules should not depend on the database, HTTP framework, cover storage, or token implementation; external services are replaceable; and composition binds interfaces to implementations. Laravel and React do not have to reproduce the Python or .NET folder layout. Preserve the dependency boundaries and testable business rules without adding layers that the actual implementation does not need.

The schema belongs to the backend application's migrations. The infrastructure repository starts an empty database and must not create a second copy of the schema. The demo seeder must use the API, not direct SQL. The React client consumes the API contract; it must not become a second source of business rules.

## Open items requiring project-owner confirmation

These are unresolved in the reference documents; this guide does not decide them:

1. **Concurrency token:** ADR-002 and task T-13 require a per-book version token, but the physical data model does not specify its column, type, or update semantics.
2. **Cover download authorization:** `GET /books/{id}/cover` is mentioned in the API prose, but its required role is not stated.
3. **ISBN updates:** the `PUT /books/{id}` field list omits ISBN without saying whether it is immutable.
4. **Response schemas:** some list and detail endpoints do not have complete response examples or schemas.
5. **Pagination boundaries:** the maximum `size` is 50, but invalid or out-of-range values are not specified.
6. **Performance-test procedure:** RNF-1 sets a target of under 300 ms for 5,000 books but does not define hardware, concurrency, percentile, or whether network time is included.
7. **Documentation language:** Constitution Article XI says project documentation and user-facing messages are in Spanish. This repository has been translated into English at the user's request; confirm whether this English documentation is accepted as the project deliverable. The translation does not change the Spanish-language requirement for user-facing messages.

Resolve these questions with the project owner before encoding assumptions in the schema or API. Until then, leave the affected behavior explicitly pending.

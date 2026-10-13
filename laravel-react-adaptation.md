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

- The actors, permissions, scope, user stories, and acceptance criteria in [Python](spec-python/spec.md) and [.NET](spec-.net/spec.md).
- The routes, field names, payload shapes, status codes, error envelope, authentication behavior, and role rules in the [Python](spec-python/api-contract.md) and [.NET](spec-.net/api-contract.md) contracts. The API contract is language-independent.
- The six entities, relationships, constraints, frozen loan-item values, soft deletion, and calculated loan status in the [Python](spec-python/data-model.md) and [.NET](spec-.net/data-model.md) data models.
- The four accepted architecture decisions in [Python](spec-python/adr/) and [.NET](spec-.net/adr/): migrations own the schema; optimistic concurrency protects copy counts; books are soft-deleted; the most-borrowed report uses frozen values.
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
| Architecture tests | `import-linter` | NetArchTest | Preserve the mandatory dependency rules in Constitution Articles I–III; the challenge documents do not prescribe a Laravel architecture-testing package |

The two references also agree on PostgreSQL 16, a signed HS256 JWT valid for eight hours, UTC instants, a version-token concurrency decision, and an external cover store whose database value is an opaque key. These requirements or decisions must not be silently replaced with framework defaults. Where their physical implementation is not fully specified, see the open items below.

## Mapping principles, not folder names

The reference architecture uses ports and adapters. Constitution Articles I–III require a domain independent of other project layers and third-party libraries, an application layer depending only on the domain and declaring its own ports, and one composition point for port-to-adapter bindings. Translate these boundaries to PHP; do not substitute Eloquent models for independent domain objects or put Laravel/ORM types in port signatures. Laravel and React do not have to reproduce the Python or .NET folder names, but framework convenience does not waive those dependency rules.

The schema belongs to the backend application's migrations. The infrastructure repository starts an empty database and must not create a second copy of the schema. The demo seeder must use the API, not direct SQL. The React client consumes the API contract; it must not become a second source of business rules.

## Open items requiring project-owner confirmation

These are unresolved in the reference documents; this guide does not decide them:

1. **Concurrency token:** ADR-002 and task T-13 require a per-book version token, but the physical data model does not specify its representation, type, or update semantics. Do not assume a new column is required.
2. **Cover download authorization:** `GET /books/{id}/cover` requires authentication under CA-01.3 and is not public, but its authorized role is not stated.
3. **ISBN updates:** the `PUT /books/{id}` field list omits ISBN without saying whether it is immutable.
4. **Response schemas:** some list and detail endpoints do not have complete response examples or schemas.
5. **Pagination boundaries:** the maximum `size` is 50, but invalid or out-of-range values are not specified.
6. **Performance-test procedure:** RNF-1 sets a target of under 300 ms for 5,000 books but does not define hardware, concurrency, percentile, or whether network time is included.
7. **Documentation language:** Constitution Article XI says project documentation and user-facing messages are in Spanish. This repository has been translated into English at the user's request; confirm whether this English documentation is accepted as the project deliverable. The translation does not change the Spanish-language requirement for user-facing messages.

Resolve these questions with the project owner before encoding assumptions in the schema or API. Until then, leave the affected behavior explicitly pending.

## Original specification review

The source for this review is repository commit `d1364dd`, before translation and clarification. The original business specification, API contract, data model, and four ADRs are identical between Python and .NET; the plans, architecture, constitution, and architecture tests contain stack-specific choices. The findings below identify gaps and implementation pitfalls, not intentional traps attributed to the author.

### Confirmed scope guardrails

| Area | Source-backed requirement | What must not be substituted |
|---|---|---|
| Database structure | Data model §§0–3 and §9: six business tables in `library`, singular English names, UUIDs, no column defaults or audit timestamps; fixed category IDs and Spanish names | Default plural Eloquent tables, auto-increment IDs, automatic `created_at`/`updated_at`, translated seed names, or new business tables |
| Database ownership | ADR-001: DDL and category seed belong to application migrations; `infra` provides the empty engine | Schema creation scripts in `infra`, or direct SQL seeding of demonstration data |
| Counts and history | CA-03.3, CA-06.3, CA-07.2, RN-4: availability is server-managed; loans are atomic; returns increment once; historical values are frozen | Client-supplied availability, partial loans, repeat increments, or reading historical titles from the live catalog |
| Member limit | CA-06.4: at most 3 unreturned books across all loans; the member detail's `active_loans` counts unreturned items | Counting loan headers or limiting only the new loan's number of items |
| Routes and permissions | CA-01.3 and API §3: only login and health are public; members are records, not authenticated users | Public catalog/cover endpoints based on the data's “Public” classification, member accounts, or extra CRUD routes |
| Existing operations | API §3: users can be created as librarians; categories are read-only; members can be created and read; loans can be created and read with per-item returns | APIs to grant admin, maintain categories, delete members, or edit/delete recorded loans |
| HTTP contract | API §§1–2: `items`, `page`, `size`, `total`; one error envelope; specified 401/403/404/409/422 outcomes | Laravel's default paginator or validation body as a replacement for the contract |
| Covers | CA-03.4, API §§3.4/4, data model §7: opaque nullable key, JPEG/PNG content validation, max 2 MB, binary retrieval; clear key and commit before deleting the file | A filesystem path or file bytes in the database, empty-string keys, extension-only validation, or treating download as public |
| Concurrency | ADR-002 and RNF-2: optimistic version control and database range check; exactly one last-copy request succeeds | Replacing the accepted decision with an in-memory counter or treating a `CHECK` failure as normal proof of correct concurrency |
| Report | ADR-004 and API §3.7: group frozen `book_id`, `book_title`, `category_name`; calculate the report, do not persist it | A report table, live-catalog grouping, or member/librarian breakdown |
| Boundaries and language | Constitution I–III; RN-6 and Article XI: preserve dependency rules; code, columns, and routes are English; user messages are Spanish | Weakening the architecture because of the framework or translating user-facing messages along with documentation |

### Unresolved source issues: do not change scope to solve them

| Issue | Evidence and impact | Confirmation needed; no assumed resolution |
|---|---|---|
| Immediate demo overdue data | T-21 requires 3 overdue lines from a seeder that uses only the API. CA-06.6 gives new loans a 14-day due period; the loan request has no historical timestamp field. A fresh seed cannot immediately produce overdue items through the documented API alone. | Confirm the authorized demonstration/test-time procedure. Do not add a date-override endpoint, change the loan period, or write demo records with SQL. |
| Version-token representation | ADR-002, plan D-04, and T-13 require a per-book token, but data model §3 has no declared representation. A version token could be an application column or a database-provided mechanism; neither is specified. | Confirm representation and update semantics. Keep the existing column list and ADR unchanged until then. |
| Concurrent member loans and returns | CA-06.4 and CA-07.2 require a global member limit and exactly-once return increments. A book token alone does not document how to protect the same member borrowing different books concurrently or two simultaneous returns of one item. ADR-002's statement that `returned_at` is filled describes sequential behavior, not a complete race-handling protocol. | Confirm adapter transaction/isolation behavior that preserves these existing invariants. Do not invent columns, new endpoints, or a different business limit. |
| Physical invariants still pending | Constitution VI requires database-enforceable invariants; data model §4 explicitly marks `ck_book_total_positive` and `ck_user_role` pending T-18. T-17 is described as the completed core, before those tasks. | Do not report the pending constraints as already enforced. Confirm the expected completion level; preserve T-18 and the existing labels. The ISBN-digits and nonempty-cover-key rules also lack explicit database constraint definitions. |
| Dates and overdue rounding | CA-09.1 defines overdue as unreturned with `due_at` before now. API §3.7 says report dates are inclusive; loan-list date formats and `days_late` rounding are not specified. | Keep the strict `due_at < now` condition; do not invent a rounding rule or new date-filter semantics. Confirm those missing details. |
| Authentication and unexpected errors | API §§1–2 define status codes and a common error body, but do not name codes for invalid/expired/missing tokens or unexpected server failures. | Preserve the specified status codes and error shape; obtain missing code names rather than introducing new contract codes. |
| Validation detail | ISBN is specified as 13 digits, not an ISBN checksum. Field lengths exist in data model §3, but a complete input-validation matrix is absent, as are password policy and precise byte definition of 2 MB. | Apply stated limits without introducing checksum, password-complexity, document-format, or file-size policies not defined by the source. Confirm ambiguous boundaries. |
| Reference folder layout | The .NET constitution names `src/Library.Domain/` and `src/Library.Bootstrap/Composition/PortBindings.cs`; its architecture tree instead uses `src/Library/domain/` with an external composition-path comment. | This is a reference-layout inconsistency, not a new Laravel requirement. Preserve dependency boundaries; do not invent a product change to reconcile folder names. |
| Executable evidence | Data model labels describe intended enforcement, and every task initially has pending status. No implemented database or executed test evidence exists in this documentation repository. | Do not treat the labels or translated documentation as proof of a passing schema, contract, concurrency, or load test. Provide actual evidence during implementation. |

The earlier open-items list also covers the unresolved cover role, ISBN updates, response schemas, pagination, performance measurements, and the documentation-language deviation. This review does not grant permission to add schema columns, API operations, or business rules.

# Specification — Manager Library

**What is being built and why.** A system for a small library to manage its book catalog, members, and loans, and to see which books are borrowed most often.

Governed by [`constitution.md`](constitution.md). The data model lives only in [`data-model.md`](data-model.md), and the HTTP contract only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | Responsibilities | Authentication |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and members; creates librarians; views all reports | Username and password → token |
| **Librarian** (`librarian`) | Records loans and returns; views the catalog, members, and overdue loans | Username and password → token |
| **Member** | Person who borrows books. **Does not access the system**: this is a record, not a user | — |

The `admin` role is provisioned by deployment from environment variables. **It cannot be granted at runtime.**

## 2. Scope

**In scope:** book catalog with optional cover, five fixed categories, members, loans with multiple items, returning items individually, overdue loans, most-borrowed report, and role-based login.

**Out of scope and not to be reopened:** fines and payments, reservations, renewals, email or SMS, bulk import, inter-branch loans, and book attributes beyond those in §3 (no publisher, publication year, or language).

## 3. User stories and acceptance criteria

### HU-01 · Sign in
As a user, I want to authenticate so I can operate according to my role.
- **CA-01.1** Valid credentials return a token valid for 8 hours.
- **CA-01.2** Invalid credentials return 401 with a message that does **not** say whether the username or password was incorrect.
- **CA-01.3** Every endpoint except login and `/health` requires a token. No token: 401. Insufficient role: 403.

### HU-02 · Create librarians (admin)
- **CA-02.1** The admin creates a user with a username, password, and `librarian` role.
- **CA-02.2** The username is stored lowercase and trimmed; it is unique. Duplicate: 409.
- **CA-02.3** Requesting the `admin` role through the API returns 422. The password is never returned or written to logs.

### HU-03 · Manage books (admin)
- **CA-03.1** Create a book with title, author, ISBN, category, and total copies. Available copies initially equal total copies.
- **CA-03.2** The ISBN has 13 digits and is unique among active books. Duplicate: 409.
- **CA-03.3** Total copies ≥ 1. Changing the total adjusts available copies by the same difference and **never** leaves available copies < 0 (422).
- **CA-03.4** Uploading a cover (JPG/PNG, max 2 MB) stores an **opaque key**; without a cover, the field is `null`, never an empty string.
- **CA-03.5** Deleting a book is a **soft delete**: it disappears from the catalog but remains in past loans. A book with open loans cannot be withdrawn (409).

### HU-04 · Search books
- **CA-04.1** Partial text search in title or author, category filter, active books only, title sort, pagination (`page`, `size`, max 50).
- **CA-04.2** The response includes the total number of results.
- **CA-04.3** Requesting a nonexistent or withdrawn book by ID returns 404.

### HU-05 · Manage members
- **CA-05.1** Admins and librarians create members with a name and document number. The document number is unique: duplicate 409.
- **CA-05.2** Members can be listed and searched by name or document number, with pagination.
- **CA-05.3** Members cannot be deleted.

### HU-06 · Record a loan
- **CA-06.1** A loan is for **one** member, is recorded by the authenticated user, and contains 1 to 3 items.
- **CA-06.2** The same book cannot appear more than once in a loan (422).
- **CA-06.3** Each item deducts **1** available copy in the same operation. If no copies are available, return 409 and **record nothing**.
- **CA-06.4** A member cannot have more than **3 unreturned books** in total (422, including the current count).
- **CA-06.5** A member with any **overdue** book cannot borrow another (409).
- **CA-06.6** The due date is `loaned_at + 14 days`. The item freezes the title, ISBN, and category name **at that moment**.
- **CA-06.7** A recorded loan cannot be edited or deleted.

### HU-07 · Record a return
- **CA-07.1** Return **one item** at a time. Set `returned_at` and add 1 to available copies.
- **CA-07.2** Returning an already returned item returns 409. **Never** add the copy twice.
- **CA-07.3** Available copies never exceed total copies (the database guarantees this).
- **CA-07.4** When all items have been returned, the loan is **closed**.

### HU-08 · View and list loans
- **CA-08.1** A loan detail includes its items and calculated status (`open`, `overdue`, `closed`).
- **CA-08.2** List loans by date range, status, and member, ordered by date descending and paginated.
- **CA-08.3** Status is **calculated**, not stored.

### HU-09 · Overdue loans
- **CA-09.1** List unreturned items whose `due_at` is before now: book, member, and days late.
- **CA-09.2** Sort by greatest lateness first. Admin and librarian only.

### HU-10 · Most-borrowed book report
- **CA-10.1** For a date range, return one row per **book and frozen category name**, with the number of times borrowed, ordered from highest to lowest.
- **CA-10.2** End before start: 422. A range with no loans returns an empty list, not an error.
- **CA-10.3** The report is **not** broken down by librarian or member.
- **CA-10.4** A report for a closed period does **not** change if the book or its category is later renamed.
- **CA-10.5** Admin only.

## 4. Business rules

| # | Rule |
|---|---|
| RN-1 | Available copies are always between 0 and total copies. |
| RN-2 | Loan period: 14 days. Maximum 3 unreturned books per member. |
| RN-3 | A member with overdue books cannot borrow more until they return them. |
| RN-4 | Frozen values on the loan item (title, ISBN, category) do not follow the catalog. |
| RN-5 | Loans and returns are not deleted. Books are soft-deleted only. |
| RN-6 | Dates are UTC instants. User-facing messages are in Spanish; code and columns are in English. |

## 5. Non-functional requirements

| # | Requirement | Measurement |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5,000 books | Load test using the `tool` seeder |
| RNF-2 | When two librarians lend the **last copy** simultaneously, one succeeds and the other receives 409 | Concurrent test |
| RNF-3 | Passwords are never transmitted or stored in plaintext and never appear in logs | Review logs and responses |
| RNF-4 | All HTTP errors use the same body (see contract) | Contract test |
| RNF-5 | The system starts with one command and the database starts empty | `infra` README |

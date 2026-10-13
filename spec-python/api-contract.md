# API contract — Manager Library

The HTTP contract is **language-independent**: an implementation in React + Laravel must follow it just as an implementation in Python or .NET must. The behavior of each operation is in [`spec.md`](spec.md); stored data is described in [`data-model.md`](data-model.md).

## 1. Conventions

| Topic | Rule |
|---|---|
| Base path | `/api/v1` |
| Format | JSON, UTF-8. Field names use `snake_case` |
| Identifiers | UUID v4 as text |
| Dates | ISO 8601 in UTC, with `Z` (`2026-10-14T14:15:00Z`) |
| Money | None. The system does not handle amounts |
| Authentication | `Authorization: Bearer <token>` on all requests except `POST /auth/login` and `GET /health` |
| Pagination | `?page=1&size=20`. `size` maximum 50. Response: `{ "items": [...], "page": 1, "size": 20, "total": 134 }` |
| Language | Error messages in Spanish; fields and codes in English |

### Single error body for all errors

```json
{
  "code": "BOOK_NOT_AVAILABLE",
  "message": "No quedan ejemplares disponibles de «Cien años de soledad».",
  "details": [ { "field": "items[0].book_id", "issue": "sin ejemplares" } ]
}
```

`details` may be empty. It **never** includes stack traces, SQL, server paths, or the password.

### Status codes

| Status | When |
|---|---|
| 200 / 201 / 204 | Success / created / no content |
| 401 | Missing, invalid, or expired token; invalid credentials |
| 403 | Insufficient role |
| 404 | Does not exist (or is withdrawn) |
| 409 | State conflict: duplicate, no available copies, already returned, member has overdue items, book has open loans |
| 422 | Validation: invalid field, broken input rule, inverted date range |

## 2. Business error codes

| `code` | Status | Source |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | CA-01.2 |
| `FORBIDDEN_ROLE` | 403 | CA-01.3 |
| `NOT_FOUND` | 404 | Any resource |
| `USERNAME_TAKEN` | 409 | CA-02.2 |
| `ISBN_TAKEN` | 409 | CA-03.2 |
| `DOCUMENT_TAKEN` | 409 | CA-05.1 |
| `BOOK_HAS_OPEN_LOANS` | 409 | CA-03.5 |
| `BOOK_NOT_AVAILABLE` | 409 | CA-06.3 |
| `MEMBER_HAS_OVERDUE` | 409 | CA-06.5 |
| `ITEM_ALREADY_RETURNED` | 409 | CA-07.2 |
| `MEMBER_LIMIT_EXCEEDED` | 422 | CA-06.4 |
| `DUPLICATE_BOOK_IN_LOAN` | 422 | CA-06.2 |
| `VALIDATION_ERROR` | 422 | Any invalid field |
| `INVALID_DATE_RANGE` | 422 | CA-10.2 |

## 3. Endpoints

### 3.1 Authentication and health

| Method | Route | Role | Description |
|---|---|---|---|
| POST | `/auth/login` | public | Returns a token |
| GET | `/health` | public | `{ "status": "ok" }` |

`POST /auth/login`

```json
// Request
{ "username": "ana", "password": "********" }
// 200
{ "access_token": "eyJ...", "token_type": "Bearer", "expires_in": 28800, "role": "librarian" }
```

### 3.2 Users

| Method | Route | Role | Description |
|---|---|---|---|
| POST | `/users` | admin | Creates a librarian |

```json
// Request (role accepts only "librarian")
{ "username": "Ana ", "password": "********", "role": "librarian" }
// 201 (no password or password_hash)
{ "id": "…", "username": "ana", "role": "librarian" }
```

### 3.3 Categories

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/categories` | authenticated | The five categories, ordered by name |

### 3.4 Books

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/books` | authenticated | Search: `q`, `category_id`, `page`, `size` |
| GET | `/books/{id}` | authenticated | Detail |
| POST | `/books` | admin | Creates a book |
| PUT | `/books/{id}` | admin | Replaces title, author, category, and total copies |
| DELETE | `/books/{id}` | admin | Soft delete |
| GET | `/books/{id}/cover` | TBD (owner confirmation required) | Returns the cover binary or 404 |
| PUT | `/books/{id}/cover` | admin | Uploads a cover (`multipart/form-data`, field `file`) |
| DELETE | `/books/{id}/cover` | admin | Removes the cover |

Book (response):

```json
{
  "id": "…", "title": "Cien años de soledad", "author": "Gabriel García Márquez",
  "isbn": "9780307474728", "category": { "id": "…", "name": "Novela" },
  "total_copies": 4, "available_copies": 3, "cover_key": null
}
```

`POST /books` request fields: `title`, `author`, `isbn`, `category_id`, `total_copies`. The server calculates available copies.
`cover_key` is an opaque key: use `GET /books/{id}/cover` to display the cover; it returns the binary content or 404.

### 3.5 Members

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/members` | admin, librarian | List and search: `q`, `page`, `size` |
| GET | `/members/{id}` | admin, librarian | Detail with `active_loans` (count of unreturned items) |
| POST | `/members` | admin, librarian | Create: `full_name`, `document_number` |

### 3.6 Loans

| Method | Route | Role | Description |
|---|---|---|---|
| POST | `/loans` | admin, librarian | Records a loan |
| GET | `/loans` | admin, librarian | List: `from`, `to`, `status`, `member_id`, `page`, `size` |
| GET | `/loans/{id}` | admin, librarian | Detail with items and status |
| POST | `/loans/{loan_id}/items/{item_id}/return` | admin, librarian | Returns one item |
| GET | `/loans/overdue` | admin, librarian | Overdue items |

`POST /loans`

```json
// Request (1 to 3 items, no duplicate books)
{ "member_id": "…", "items": [ { "book_id": "…" }, { "book_id": "…" } ] }
// 201
{
  "id": "…", "member_id": "…", "loaned_at": "2026-10-14T14:15:00Z", "status": "open",
  "items": [
    { "id": "…", "book_id": "…", "book_title": "Cien años de soledad", "isbn": "9780307474728",
      "category_name": "Novela", "due_at": "2026-10-28T14:15:00Z", "returned_at": null }
  ]
}
```

`status` is calculated: `open` (there are unreturned items and none are overdue), `overdue` (at least one unreturned item is overdue), `closed` (all items have been returned).

`POST /loans/{loan_id}/items/{item_id}/return` → 200 with the item (`returned_at` populated).

`GET /loans/overdue` → `items`: `{ "loan_id", "item_id", "book_title", "member_name", "due_at", "days_late" }`. The member's name is shown here because this is an operational list; **the report in §3.7 does not show it**.

### 3.7 Report

| Method | Route | Role | Description |
|---|---|---|---|
| GET | `/reports/most-borrowed` | admin | `from` and `to` required (date, both inclusive) |

```json
{
  "from": "2026-10-01", "to": "2026-10-31",
  "rows": [
    { "book_id": "…", "book_title": "Cien años de soledad", "category_name": "Novela", "times_borrowed": 12 }
  ]
}
```

One row per frozen `book_id + book_title + category_name`. Order: `times_borrowed` descending, then title. No members or librarians.

## 4. Security

| Aspect | Rule |
|---|---|
| Token | Signed JWT, 8 hours, with `sub` and `role`. No personal data inside |
| Passwords | Only the hash is stored; the hash never appears in any response |
| Authorization | Checked on every endpoint by role; a 403 does not reveal whether the resource exists |
| CORS | Only the frontend origin configured through the environment |
| Cover upload | `image/jpeg` and `image/png` only, max 2 MB; validate the content, not the extension |

## 5. Contract details pending confirmation

The following details are not settled by the current specification. Do not infer behavior until the project owner confirms it:

- **Cover download authorization:** the contract mentions `GET /books/{id}/cover` and says it returns the binary or 404, but does not state the required role. The role is intentionally marked as pending in the endpoint table.
- **ISBN updates:** `PUT /books/{id}` lists title, author, category, and total copies, but does not say whether ISBN can be changed.
- **Response schemas:** complete response shapes are not provided for every list and detail endpoint. The examples in this document do not define omitted fields.
- **Pagination boundaries:** `size` has a maximum of 50, but behavior for missing, nonnumeric, zero, negative, or greater-than-50 values is unspecified.

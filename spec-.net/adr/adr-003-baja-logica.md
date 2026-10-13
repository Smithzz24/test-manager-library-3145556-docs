# ADR-003 · Books are withdrawn, not deleted

- **Status:** accepted
- **Context:** a loan line points to a book. Physically deleting the book would break history and reports.
- **Decision:** a book has `deleted_at`. Withdrawal sets it; the catalog filters on `deleted_at IS NULL` in one place (a global filter). A book with open loans cannot be withdrawn (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a **partial index** over active books, so an ISBN may be registered again after its book is withdrawn.
- **Rejected alternatives:** physical deletion with cascading (destroys history); setting `active = false` (does not say when it happened).
- **Consequences:** `loan_item.book_id` has a `RESTRICT` FK as a **last-resort safeguard**: a manual `DELETE` or a future code change fails loudly instead of corrupting history. The cover binary is the only data physically deleted.
- **Verification:** querying a withdrawn book returns 404; a `DELETE` through `psql` on a book with loan lines fails because of the FK.

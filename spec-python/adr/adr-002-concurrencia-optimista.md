# ADR-002 · Optimistic concurrency for inventory copies

- **Status:** accepted
- **Context:** two librarians may lend the **last copy** of the same book at the same time (RNF-2). If both read `available_copies = 1` and both write, the system lends a book it does not have.
- **Decision:** use **optimistic concurrency control**. The book carries a version token; saving fails if the version has changed, and the server responds with **409 `BOOK_NOT_AVAILABLE`** (or retries once if the copy is still available). The final safeguard is the `ck_book_available_range` `CHECK` (`0 <= available_copies <= total_copies`).
- **Rejected alternatives:** pessimistic row locking (`SELECT … FOR UPDATE`) for every loan, which serializes the whole library for a rare case; an in-memory counter, which does not survive multiple processes.
- **Consequences:** if the `CHECK` fails, **something wrote outside the adapter**: that is a defect, not a business case. Returning the same line twice does not increment the count twice because `returned_at` is already set (CA-07.2).
- **Verification:** a concurrent test sends two requests for a book with one copy; exactly one receives 201.

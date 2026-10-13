# ADR-004 · The report is aggregated in the database using frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a date range. If it reads a title or category from the live catalog, renaming a book or category would change a report for a closed period.
- **Decision:** (1) The report is **not persisted**; it is calculated with one aggregate query through a read port. (2) It groups by `book_id`, `book_title`, and `category_name` **as frozen on the loan line**. (3) `loan_item.category_name` intentionally has **no FK**. (4) If a book changes category during the range, the report shows **two rows**; it does not choose one.
- **Rejected alternatives:** joining to the live catalog (the report would no longer be stable); taking “the latest value in the range” (a new loan would change what was already read); materializing a report table (a second source of truth).
- **Consequences:** CA-10.1 says “one row per book and frozen category name,” not “one row per book.” The report does not include members or librarians (privacy).
- **Verification:** lend a book, rename it, and request the report for the same range; the report shows the old title.

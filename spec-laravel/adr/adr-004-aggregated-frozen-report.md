# ADR-004 · The report is computed in the database and groups by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a date range. If it read the title or the category from
  the live catalogue, renaming a book or a category would change a report of an already closed period.
- **Decision:** (1) the report is **not persisted**: it is computed with a single aggregate query behind a read port (`ReportReader`).
  (2) It groups by `book_id, book_title, category_name` **exactly as they were frozen in the line**. (3) `loan_item.category_name` has
  **no FK** on purpose. (4) If a book changed category within the range, the report shows **two rows**: it does not pick a winner.
- **Discarded alternatives:** joining with the live catalogue (the report stops being stable); taking "the most recent value in the
  range" (a new loan would change what was already read); materializing a report table (second source of truth).
- **Consequences:** CA-10.1 says "one row per book and frozen category name", not "one row per book". The report does not include
  members or librarians (privacy).
- **How it is verified:** lend a book, rename it, request the report for the same range: the report title is the old one.

# ADR-003 · Books are deactivated, not deleted

- **Status:** accepted
- **Context:** a loan line points to a book. If the book is physically deleted, the history and the report are broken.
- **Decision:** the book has `deleted_at`. Deactivating it fills that column; the catalogue filters by `deleted_at IS NULL` in a single
  place (a global scope). A book with open loans **cannot** be deactivated (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a **partial**
  index over active books, so a deactivated ISBN can be registered again.
- **Discarded alternatives:** physical delete with cascade (destroys the history); marking `active = false` (does not say when).
- **Consequences:** `loan_item.book_id` has an FK `RESTRICT` as the **last barrier**: a manual `DELETE` or a future code change fails
  loudly instead of corrupting the history. The cover binary is the only data that is really deleted physically.
- **How it is verified:** querying a deactivated book returns 404; a `DELETE` through `psql` on a book with lines fails because of the FK.

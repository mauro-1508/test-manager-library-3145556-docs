# ADR-002 · Optimistic concurrency on the copies

- **Status:** accepted
- **Context:** two librarians can lend the **last copy** of the same book at the same time (RNF-2). If both read
  `available_copies = 1` and both write, the system lends a book that has no copies left.
- **Decision:** **optimistic** concurrency control. The book write is a conditional `UPDATE`, executed through the persistence adapter,
  that only applies if the row still holds the values that were read. If zero rows are affected, the operation fails and the server
  responds **409 `BOOK_NOT_AVAILABLE`** (or retries once if the copy is still available).
  The **last barrier** is the `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Discarded alternatives:** pessimistic row locking (`SELECT … FOR UPDATE`) on every loan: serializes the whole library for a rare
  case; in-memory counter: does not survive more than one process.
- **Consequences:** if the `CHECK` fires, **something wrote outside the adapter**: it is a defect, not a business case. Returning the same
  line twice does not add twice because `returned_at` is already filled (CA-07.2).
- **How it is verified:** concurrent test against real PostgreSQL with two simultaneous requests on a book with one copy: exactly one
  receives 201.

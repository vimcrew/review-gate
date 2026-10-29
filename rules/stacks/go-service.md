# Go service

Apply review essentials and one profile. Use the base finding categories.

- Check every consequential error and preserve context with wrapping. Do not
  return success after failed writes or conceal partial results. Handle typed
  nil interfaces, integer bounds and time units on changed boundaries.
- Pass context through database/HTTP calls; cancellation must stop owned work.
  Bound goroutine counts and queues, define channel ownership/closure and ensure
  every goroutine has a shutdown path. Protect shared maps/state from races.
- Close response bodies, rows, files and transactions on all paths. Check row
  iteration errors and affected-row counts when they determine business success.
  Use one transaction for atomic writes; defer rollback safely around commit.
- Enforce request size limits, decoding validation, resource authorization and
  tenant predicates. Parameterize SQL and check untrusted outbound URLs/paths.
- Configure server/client deadlines and graceful shutdown. Retried operations
  must be idempotent; do not retry cancellation, invalid input or permanent denial.
- Preserve JSON/API compatibility, distinguishing zero, absent and null where
  contractual. Specify timestamp zones and precision for persisted values.
- Add table-driven tests for boundary cases and httptest coverage of changed
  handlers. Use real database integration tests for queries/migrations, and the
  race detector for changed shared-state code. Test goroutine completion on
  cancellation rather than merely waiting a fixed sleep.

# Rust service

Apply review essentials and one profile. Use the base finding categories.

- Return actionable Result errors for expected invalid input and dependency
  failure. Review unwrap/expect/indexing on reachable untrusted paths for panic;
  a panic in a request/task must not silently lose acknowledged work.
- Document and verify every changed unsafe block's safety preconditions, aliasing,
  ownership and lifetime assumptions, including FFI error paths. Public safe APIs
  must not allow callers to violate those preconditions.
- Keep blocking I/O and CPU-heavy work off async executor workers. Check locks
  held across await, cancellation safety, spawned-task ownership and bounded
  channels. Dropped futures must not leave a half-applied business operation.
- Propagate deadlines, close resources and handle task JoinError outcomes. Retried
  writes/messages need idempotency and durable recovery for partial completion.
- Validate deserialized inputs and authorize resources/tenants before queries.
  Parameterize SQL; check numeric conversion, overflow and timestamp units at
  serialization boundaries. Never log credentials or whole sensitive payloads.
- Define transaction boundaries around atomic writes. Test concurrent invariants,
  database constraints and schema upgrades against the supported database engine.
- Add unit/property tests for boundary invariants and integration tests for changed
  service/adapter behavior. Use Miri or another suitable check when changing unsafe
  memory assumptions; clippy passing alone does not establish memory safety.

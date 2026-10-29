# TypeScript + Node.js + NestJS

Apply review essentials and one profile. Use the base finding categories.

- Validate untrusted JSON at runtime, including nested objects and transformed
  numeric/boolean fields. Explicitly map writable fields to prevent mass assignment;
  TypeScript interfaces and casts do not validate an HTTP payload.
- Guards/authentication must cover HTTP, WebSocket and message entry points that
  expose the changed operation. Enforce object/tenant authorization in the use
  case; possession of an ID is insufficient. Test direct unauthorized access.
- Keep provider scopes correct: a singleton must not retain request/user state.
  Check module exports, dependency cycles and lifecycle hooks with the real Nest
  container for changed wiring.
- Await asynchronous work and propagate rejection to the caller or durable job
  status. Do not use unobserved promises for required side effects. Bound queues,
  body size, remote timeouts and retries; avoid CPU-heavy work on the event loop.
- Carry the same database transaction/session through all atomic writes. Handle
  unique conflicts and concurrent updates; commit before emitting durable events
  via an outbox or equivalent recovery path. Jobs must tolerate redelivery.
- Check safe integer bounds, decimal money handling and timestamp serialization.
  Parameterize SQL and validate external URLs/file paths before adapter access.
- Validate environment configuration at startup; mask secrets in logging and
  exception filters. Close database pools/consumers during graceful shutdown.
- Test business services, real validation/guard/filter behavior with HTTP tests,
  and database/queue adapters with integration tests. Exercise cancellation,
  duplicate delivery and partial failure for changed asynchronous flows.

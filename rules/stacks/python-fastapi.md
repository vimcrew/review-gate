# Python + FastAPI

Apply review essentials and one profile. Use the base finding categories.

- Validate requests with explicit schemas and constrain lengths/ranges. Check
  coercion, missing versus null values and response filtering so private model
  fields cannot appear in API responses. Type hints alone do not validate data.
- Dependencies must enforce authentication and resource/tenant authorization for
  every changed route, including bulk access. Test unauthorized and cross-tenant
  requests through the actual dependency chain.
- Do not run blocking database/network calls inside async handlers without a
  suitable execution boundary. Await tasks, propagate cancellation and bound
  outbound calls; mutable module globals must not carry per-request state.
- Scope database sessions and transactions to the intended unit of work. Roll
  back on failure and close sessions on all paths. Avoid lazy attribute access
  after closure and ORM entities leaking fields through serialization.
- Required side effects need durable jobs or synchronous completion; in-process
  background work can disappear on restart. Check idempotency on retries and
  ensure a successful HTTP response does not claim an uncommitted operation.
- Use lifespan/resource cleanup consistently in production and tests. Validate
  settings at startup, restrict CORS to intended origins and keep debug responses,
  credentials and personal data out of production logs.
- Test request/response contracts through the ASGI app, dependency overrides that
  preserve the behavior under test, and real database migrations from prior data.
  Include validation failures, rollback and duplicate submission when applicable;
  do not mock away the SQL or authorization predicate the test is meant to prove.

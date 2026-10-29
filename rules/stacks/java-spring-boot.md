# Java + Spring Boot

Apply review essentials and one profile. Classify concrete defects with the base
categories; optional layering changes are minor `refactor`.

- Validate request DTOs at entry points and enforce resource/tenant authorization
  in the server path, including jobs. Map failures to stable HTTP status and error
  bodies without exposing exception internals or serializing persistence entities.
- Keep use-case transactions atomic. Check Spring proxy invocation and rollback
  rules, especially caught/checked exceptions and self-invocation. A test must
  observe persisted state after failure rather than only verify method calls.
- Check JPA ownership, cascades, lazy loading and entity equality. Prevent lost
  updates using versions, constraints or atomic SQL; do not treat read-then-write
  as safe under concurrency. Bound result sets and detect new N+1 query paths.
- Close streams/resources and bound executor queues, HTTP timeouts and retries.
  Propagate interruption. Blocking JDBC must not execute on reactive event loops;
  thread-local context must not be assumed across asynchronous work.
- Use precise money arithmetic, explicit time zones and injected time where
  behavior depends on it. Check null contracts at serialization and Java-library
  boundaries, including absent versus explicit null fields.
- Bind and validate configuration; production must not inherit development
  credentials, open management endpoints or automatic destructive schema changes.
  Keep dependency versions compatible with the chosen Spring Boot dependency set.
- Add unit tests for business decisions, MVC/security slice tests for boundary
  behavior and database integration tests for changed persistence semantics.
  Use the real database engine for engine-specific SQL and migrations. Check
  upgrade from populated prior schema and mixed-version API/event compatibility.

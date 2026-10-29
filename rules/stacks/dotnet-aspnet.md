# .NET + ASP.NET Core

Apply review essentials and one profile. Use the base finding categories.

- Validate bound models and explicitly map writable properties. Enforce policies
  and resource/tenant authorization for endpoints, hubs and jobs; do not confuse
  successful authentication with permission to modify a requested entity.
- Respect DI lifetimes: scoped DbContext/request services must not be captured
  by singletons or reused concurrently. Background services create and dispose
  their own scopes; all owned async resources are disposed on failure too.
- Await asynchronous work, propagate CancellationToken and avoid sync-over-async
  blocking on request paths. Bound outbound timeouts and retries; hosted services
  must stop cleanly without silently abandoning accepted durable work.
- Use EF Core transactions for atomic writes and concurrency tokens/constraints
  for competing updates. Check tracking, query translation, N+1 access and bounded
  pagination. Do not serialize EF entities with accidental navigation graphs.
- Review migrations as executable SQL against populated prior schema, including
  locking and mixed-version deployment. Avoid destructive startup migration races
  across replicas. Parameterize raw SQL and preserve decimal/date-time semantics.
- Keep secrets outside appsettings committed to source. Restrict diagnostic
  endpoints and error details; check cookie/CSRF behavior for browser session writes.
- Test endpoints with the actual middleware/auth pipeline, business rules in unit
  tests, and provider-specific queries and migrations on the real database.
  EF's in-memory provider cannot prove relational constraints or SQL behavior.

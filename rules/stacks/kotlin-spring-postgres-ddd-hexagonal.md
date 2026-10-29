# Kotlin + Spring Boot + PostgreSQL + DDD + hexagonal architecture

Apply with review essentials and one profile. Use `correctness`, `bug`,
`security`, `regression`, `tests` for demonstrated defects; an adopted boundary
violation can be `policy`. Pattern preferences alone are minor `refactor`.

## Domain, application and adapters

- Domain code owns business decisions, aggregates and value objects. It must not
  import Spring, HTTP, JPA, SQL or adapter classes. Time, IDs and external facts
  needed by domain logic enter explicitly; tests can control them.
- Application use cases coordinate domain behavior, authorization context and
  transactions through ports. Controllers, scheduled jobs and message consumers
  translate inputs, invoke a use case and map outcomes; they do not duplicate
  invariants or reach directly into another aggregate's storage.
- Input ports describe use cases; output ports describe required capabilities
  in domain/application terms. Adapters implement output ports. Check dependency
  direction: inner layers never construct concrete adapters or expose transport
  DTOs, JPA entities, EntityManager or Spring Data repositories in port signatures.
- Change aggregate state through its root. Constructors/factories and mutation
  methods must enforce invariants for every entry path, including rehydration.
  Do not expose mutable collections that bypass them. Reference other aggregates
  by identity; define consistency when a use case changes more than one root.
- Use value objects for constrained concepts with explicit units, equality and
  rounding. Check Kotlin nullability against Java/platform types and database
  nulls; `!!`, unchecked casts and default values must not hide invalid state.

## Transactions and persistence

- Place the transaction around the complete atomic use case, not each repository
  call separately. Verify isolation, rollback rules and the actual proxy call
  path: self-invocation does not activate Spring proxy advice. Test rollback
  through the real configured bean. Check Kotlin proxy openness/plugin setup.
- Do not hold database locks while waiting for remote calls. Use an outbox or
  another durable delivery design when a committed write must produce an event;
  prove recovery after a crash between write and publish. Consumers deduplicate
  using a stable business/event key, not an in-memory flag.
- Protect concurrent invariant updates using an atomic predicate, constraint,
  version check or justified lock. Handle optimistic conflicts explicitly;
  retries must rerun the safe unit of work and preserve idempotency.
- Keep persistence and domain models separate for this architecture. Map both
  ways explicitly, including IDs, versions, nulls and enum values. JPA entities
  must not escape into HTTP/domain results; check lazy access after transaction
  close, unintended cascades/orphan deletion and entity equality before ID assignment.
- Query with bounded pagination and deterministic ordering. Inspect N+1 paths,
  fetch joins and collection pagination; test query counts where a changed path
  would multiply calls. Avoid loading an entire table to filter in Kotlin.

## PostgreSQL and schema evolution

- Use parameterized SQL, tenant predicates and constraints for durable uniqueness
  and references. Check numeric precision, timestamp/time-zone semantics and
  query plans on representative distributions before claiming an index helps.
  Include predicate/order columns in index design and account for write overhead.
- Check lock modes, lock ordering, transaction duration and deadlock handling.
  A new index or column change on a live large table needs bounded lock waiting
  and a rollout plan. Concurrent index creation still waits and can fail; plan
  invalid-index cleanup. `CREATE INDEX CONCURRENTLY` cannot run in a transaction
  block: configure the migration tool accordingly for that migration.
- Add a new versioned migration instead of rewriting an applied one-time one;
  editing Flyway `R__` or Liquibase `runOnChange` is intended. Verify
  ordering/checksums and test upgrade from the previous
  schema with populated rows. Do not use Hibernate schema auto-update in production.
- For no-downtime changes, expand first, deploy compatible readers/writers,
  backfill in bounded resumable batches, validate, then contract after old users
  retire. Review both old-app/new-schema and new-app/expanded-schema behavior.
  For eligible FK/CHECK constraints, consider NOT VALID then validation; do not
  assume it avoids every lock. Destructive reversal needs restore/forward repair.

## Spring Boot configuration and verification

- Bind typed configuration and validate required values at startup. Check profile
  precedence, safe defaults, connection-pool/timeouts and graceful shutdown.
  Restrict management endpoints; do not log datasource credentials or PII.
- Keep blocking JDBC work off event-loop threads. Coroutine cancellation must
  propagate; no GlobalScope or swallowed CancellationException. Do not assume
  imperative thread-bound transactions survive arbitrary dispatcher switches.
- Domain unit tests cover invariant boundaries and transitions without Spring.
  Application tests exercise ports, failure outcomes and duplicate requests.
  MVC/security and JPA slice tests verify actual mapping, validation and queries.
- Use Testcontainers PostgreSQL for PostgreSQL-specific SQL, locking, constraints
  and migration upgrades; H2 alone cannot establish those semantics. Include a
  concurrent-write or rollback case for changed transaction behavior.
- Use ArchUnit (or an existing equivalent) to assert domain/application/adapter
  dependencies and port ownership; make a forbidden import fail the test.

Reference semantics: [Spring transactions](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html),
[PostgreSQL index creation](https://www.postgresql.org/docs/current/sql-createindex.html).

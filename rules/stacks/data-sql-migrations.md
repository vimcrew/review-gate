# SQL, data pipelines and migrations

Apply review essentials and one profile; check the actual database/version.
Use `correctness` for data-integrity defects, `security` for unauthorized access,
`regression` for broken consumers and `tests` for a concrete verification gap.

- State the expected row grain, keys and cardinality. Check join multiplication,
  null handling, duplicate keys, empty inputs, time zones, decimal precision and
  window/tie ordering. Reconcile row counts and meaningful totals before/after.
- Parameterize values and allowlist dynamic identifiers. Keep tenant/access
  predicates on reads, writes and exports; restrict migration privileges and
  avoid personal data in fixtures, logs or exported artifacts.
- Preserve applied migration history; add a new versioned step instead of editing
  a released one-time migration. Repeatable migrations (Flyway `R__`, Liquibase
  `runOnChange`) are meant to be edited. Test both clean setup and upgrade from representative old
  data, including nulls/duplicates that new constraints could reject.
- Evolve live schemas through expansion, compatible app deployment, bounded
  backfill, validation and delayed contraction. Check both app/schema directions
  during rollout and recovery. A destructive down migration is not data recovery;
  identify restore or forward-repair steps for irreversible changes.
- Estimate table scans, rewrites and lock impact using representative data/plans.
  Set appropriate lock/statement timeouts. For PostgreSQL concurrent indexes,
  configure execution outside a transaction and recover invalid indexes on failure.
  Consider NOT VALID then VALIDATE for eligible FK/CHECK additions; inspect locks
  for the specific statement rather than assuming zero downtime.
- Backfills and pipelines must resume safely from a durable checkpoint, use stable
  batching keys and tolerate duplicate delivery. Preserve watermarks/late-arrival
  policy; a retry must not double-count aggregates or overwrite newer data.
- Use constraints/atomic writes or explicit locking for concurrent invariants.
  Test contention/rollback when changing transaction logic; an application check
  before INSERT is insufficient to guarantee uniqueness.
- Verify failure is visible: rejected rows, partial batches and skipped partitions
  cannot yield an unqualified success. Assert reconciliation and recovery after
  interruption; do not validate only that the SQL parses.

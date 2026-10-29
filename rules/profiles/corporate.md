# Corporate profile

Combine with review essentials and the relevant stack. These criteria apply to
shared production systems; use documented support windows, owners and SLOs.
Explicit obligations below may use `policy`; cite the exact obligation and its
violation. A missing optional document alone is minor `docs`, not a security bug.

- Preserve supported consumers through staged API/event changes. Show how old
  and new producers, consumers and workers coexist, including rollback. Removal
  requires the agreed deprecation window and evidence that callers migrated.
- Database changes must state deploy order, expected lock/rewrite impact,
  backfill resume strategy and recovery route. Use expand/backfill/validate/
  contract for live schema replacement; delay drops until old binaries and jobs
  are retired. Test upgrade from representative populated data.
- Sensitive mutations must record actor/service identity, tenant, action, target,
  time and outcome through the existing audit mechanism. Test that denied and
  failed actions cannot appear as successful. Preserve audit retention and
  access controls; do not put full sensitive payloads in the audit record.
- Identify new PII fields and their purpose, access boundary, retention and
  deletion/export path. Mask logs/traces; keep secrets in the approved secret
  store, including CI and infrastructure. Do not broaden privilege implicitly.
- For added/updated dependencies, verify pinned or locked resolution and the
  project's vulnerability/license checks. Record license compatibility under
  the adopted allowlist and required notices; unknown or disallowed licenses
  need an explicit authorized exception. Do not infer legal compatibility from
  a package name or flag an imagined license violation.
- For a changed owned boundary, identify its owner using the existing ownership
  mechanism. A new cross-service contract, data ownership boundary or irreversible
  architectural tradeoff must have a linked ADR with alternatives and recovery
  implications. Prefer existing decisions over introducing a competing pattern.
- Risky behavior rollouts need a feature flag or a documented equivalent staged
  rollout. Check safe default, authorization to change the flag, behavior of both
  states, emergency disable, owner and removal condition. A flag must not expose
  code that is incompatible with the currently deployed schema.
- For changed critical paths, relate timeouts, resource limits and alerting to
  the service's documented SLO. Check dependency-failure behavior, bounded queues,
  graceful termination and an actionable rollback/runbook. Require measured
  evidence for performance claims; a proven SLO violation may be `acceptance`
  or `policy`, while an unmeasured optimization remains minor `performance`.
- Tests and release evidence must identify the reviewed revision. A green run
  on another commit does not prove this change; security and migration checks
  must cover the actual changed path, not only a mock of it.

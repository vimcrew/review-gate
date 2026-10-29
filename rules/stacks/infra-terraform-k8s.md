# Terraform + Kubernetes

Apply review essentials and one profile. Use `security`, `correctness`,
`regression` or explicit `policy` for proven risk; preferred layout is minor.

- Inspect the Terraform plan for the target environment: replacement/deletion,
  address moves, drift and provider changes. State moves/imports must preserve
  existing resources; a rename must not silently destroy a live database.
- Lock provider/module versions and dependency resolution. Protect remote state
  with encryption, access controls and concurrency locking; sensitive outputs
  can still exist in state. Never commit state, credentials or generated plans
  containing secrets. Separate environments by explicit state/identity boundaries.
- Grant IAM/RBAC only required actions/resources. Review wildcard grants, public
  ingress/storage and cross-namespace access. Kubernetes Secret base64 is not
  encryption; check encryption at rest and secret access/rotation mechanisms.
- Check workload security context, service-account token needs, image pinning,
  capabilities, mounted paths and network policy effects. A deny policy must still
  permit necessary DNS/dependency traffic; avoid an untested connectivity lockout.
- Set resource requests/limits and probes based on actual startup/serving behavior.
  Readiness gates traffic; liveness must not restart healthy pods solely because
  a shared external dependency is down. Check graceful termination and draining.
- Rollouts must maintain required capacity and compatibility: replicas, disruption
  budget, surge/unavailable settings, storage access modes and schema ordering.
  CronJobs/jobs need concurrency and duplicate-run behavior; no assumed exactly-once.
- Validate manifests/rendered templates and Terraform formatting/validation, then
  inspect a representative plan and policy checks. For critical network/rollout
  changes, provide an environment check and recovery route; syntax validation
  alone does not prove the service remains reachable.

Reference: [Kubernetes secret practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/).

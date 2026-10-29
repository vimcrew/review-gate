# Python + Django

Apply review essentials and one profile. Use the base finding categories.

- Enforce permissions and tenant filters in views, admin actions and jobs. With
  Django REST Framework, check object permissions and list-query filtering
  separately. Authentication alone does not authorize every row.
- Explicitly list writable ModelForm/serializer fields. Validate business
  transitions server-side; model choices or UI validation alone may not enforce
  constraints on every write path. Use database constraints for concurrent writes.
- Use atomic transactions for coupled changes. Check select_for_update placement,
  race-prone get/create flows and rollback behavior. Schedule external effects
  after commit; use durable delivery when a crash cannot lose the effect.
- Keep migrations compatible with old app instances. Separate schema expansion
  and data backfill; data migrations must use historical models and the correct
  database alias. Review reverse/forward repair behavior and lock/rewrite impact.
- Bound QuerySets and pagination; inspect select_related/prefetch_related on new
  loops and serializers. Parameterize raw SQL; ordering must be stable across pages.
- Preserve CSRF protections for session-authenticated writes, escape untrusted
  templates and validate uploads/redirects. Production settings must disable debug,
  restrict hosts and use secure session cookies and externally supplied secrets.
- Test ORM constraints, view permissions and migrations on the supported database.
  Transaction/locking tests must use test fixtures that actually commit and allow
  independent connections; ordinary TestCase wrapping can hide those behaviors.
  Include query-count assertions for a changed path that would cause N+1 queries.

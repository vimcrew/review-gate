# TypeScript + React + Next.js

Apply review essentials and one profile. Cite supported browser/runtime and
router behavior; do not assume caching defaults from a different Next.js version.

- Treat route handlers and Server Actions as public entry points: validate input
  at runtime and check session, resource ownership and tenant access inside each
  operation. Hidden buttons, layouts and navigation guards do not authorize it.
- Keep secrets and privileged data access on the server. Check client imports,
  public environment variables, serialized props and error payloads for leakage.
  Send only necessary DTO fields across the server/client boundary.
- Separate user-specific and shared cache entries. Verify keys include access
  scope, invalidation follows successful writes and no cached response leaks
  another user's data. Test against the project's actual Next.js cache settings.
- Use stable keys; prevent stale async results from replacing newer state. Check
  hook dependencies, effect cleanup and duplicate effects in development checks.
  Mutations must tolerate retries/double submits without duplicate side effects.
- Validate URL redirects and untrusted HTML. Check cookie flags and CSRF defenses
  for cookie-authenticated writes; do not rely on TypeScript types for JSON safety.
- Render loading, empty and error outcomes; propagate server failures accurately.
  Forms need labels, keyboard operation and visible validation. A blocked primary
  user flow is `bug` or `acceptance`; visual polish alone remains minor `style`.
- Verify SSR/hydration with browser-only APIs and nondeterministic values. Bound
  expensive queries and payloads; optimization claims require evidence.
- Add component tests for changed interactions and an integration/browser test for
  critical navigation or mutation. Cover denied access by direct HTTP invocation,
  stale responses and cache isolation when those paths change.

Reference: [Next.js authorization](https://nextjs.org/docs/app/guides/authentication).

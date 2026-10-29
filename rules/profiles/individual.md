# Individual profile

Combine with review essentials and one relevant stack. Focus review effort on
bugs, security and preserving the user's work; keep changes small enough to
understand and recover.

- Block demonstrable crashes, wrong results, data loss, exposed secrets and
  unauthorized access using the base categories. Show a reachable failure and
  a proportionate fix; do not require enterprise processes to justify it.
- Test the risky behavior: a regression case for a bug, invalid input for a new
  boundary, and a persistence check when data shape changes. Prefer focused
  observable assertions over coverage targets or extra test layers by default.
- State supported clients and deployment assumptions when changing contracts.
  A coordinated break is acceptable if callers migrate together and retained
  data remains readable or recoverable. Do not silently discard user data.
- Keep configuration understandable: required settings fail clearly at startup,
  secrets stay outside source control, and errors include enough context to fix
  the problem without leaking personal data.
- Do not demand an ADR, new abstraction, microservice, owner matrix, audit system
  or feature flag for a small local change unless an existing explicit project
  obligation requires it. Suggest simplification as minor `refactor`; document
  an unusual decision briefly where a future maintainer will find it.

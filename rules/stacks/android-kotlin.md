# Android + Kotlin

Apply review essentials and one profile. Use the base finding categories.

- Scope coroutines and streams to their owner (screen, ViewModel or application).
  Collect UI state with lifecycle awareness, propagate cancellation and avoid
  retaining Activity/View references after destruction. Block no main-thread I/O.
- Separate durable data from transient UI state. Rotation/process recreation must
  not duplicate purchases, submissions or navigation; restore necessary state and
  model loading, empty, failure and retry outcomes explicitly.
- Protect secrets with appropriate platform storage; do not bundle privileged
  backend credentials. Redact logs, validate deep links/intents and restrict exported
  components. Enforce authorization on the backend regardless of client checks.
- Handle denied/revoked runtime permissions and the supported Android API range.
  Check background execution limits; durable deferred work needs an appropriate
  scheduler such as WorkManager with constraints and idempotent retry behavior.
- Room/schema changes require tested migrations preserving existing user data.
  Destructive fallback must not silently erase production data. Define conflict,
  retry and offline synchronization behavior when local and server data diverge.
- In Compose, use stable item keys, observable state and correctly keyed effects;
  recomposition must not trigger writes repeatedly. Critical controls need
  accessibility semantics and usable error feedback.
- Test reducers/ViewModels with controlled dispatchers and time, database migration
  fixtures and changed UI interactions on the supported platform. Include process
  recreation, denied permission or offline recovery when the changed path depends
  on it; do not rely exclusively on happy-path screenshots.

# iOS + Swift

Apply review essentials and one profile. Use the base finding categories.

- Update UI-isolated state on the appropriate actor. Review actor reentrancy across
  await, Sendable boundaries, detached tasks and shared mutable state; a checked
  value may change during suspension. Keep blocking work off the main actor.
- Give tasks/subscriptions an owner and cancellation path. Check closure retain
  cycles and post-dismissal callbacks; cancellation and view recreation must not
  duplicate a purchase or discard an already accepted durable operation.
- Model loading, empty, failure and retry states explicitly. Preserve required
  state across backgrounding/process restart; SwiftUI body/evaluation must not
  perform side effects and identity changes must not unexpectedly reset user input.
- Store credentials in Keychain with suitable access settings; do not embed
  privileged backend secrets. Redact logs, validate universal links/URLs, restrict
  transport exceptions and handle denied/revoked permissions without a crash.
- Persisted model changes need a tested Core Data/SwiftData or equivalent migration
  from existing stores. Define offline merge/conflict behavior and preserve user
  data on failed synchronization; a fresh install alone does not prove an upgrade.
- Check supported OS availability and accessibility of changed controls, including
  VoiceOver labels, focus and dynamic text on critical flows.
- Test business/state transitions with controlled clocks/services, persistence
  upgrades with old stores and UI lifecycle behavior for changed critical paths.
  Exercise cancellation, network failure and denial rather than mocking them
  into successful responses. Forced unwraps on external input need a safe path.

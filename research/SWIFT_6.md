# SWIFT_6.md — Swift 6 features relevant to the warp daemon

**Status:** TODO (stub).

**Gates:** Module structure, concurrency model, error-handling style.

Findings here unblock decisions in
[`../daemon-v0/SCOPE.md`](../daemon-v0/SCOPE.md) about how the daemon is
organized and which Swift idioms are first-class in our codebase.

## Topics to cover

- Swift 6.0–6.2 shipped features:
  - Strict concurrency (Sendable, region-based isolation, isolated
    parameters)
  - Typed throws and `~Copyable` / `~Escapable` types
  - Macro maturity and the ergonomics for codegen we'd want around
    Heddle wire models
  - Observation framework, `@Observable`
  - Embedded-Swift trajectory (relevant for the long tail of capacity-
    sensing helpers that might run on small accelerators)
- Swift 6.3+ proposals under review worth designing forward-compat with
  (link SE proposal numbers as they land).
- Toolchain story: minimum supported Xcode/swiftc, target macOS deployment
  version, expected gotchas with concurrency back-deployment.

## Implications for v0 to capture

- Module decomposition: which boundaries we want strict-concurrency
  isolation across vs. relaxed (a single `@MainActor` boundary? per-
  subsystem actors?).
- Error handling: typed throws across the daemon's public seams, or
  `Error`-erased?
- Whether we adopt macros for Heddle wire codecs immediately or stay
  with hand-written `Codable` until macro tooling stabilizes.
- Concurrency idioms in long-lived service code (Service Lifecycle
  integration, see `SERVER_ECOSYSTEM.md`).

## Open questions

- Does any Swift 6 strict-concurrency rule force us to wrap the existing
  heddle-sdk Swift package types or do they stay drop-in?
- Are there pending SE proposals (e.g., on actor reentrancy, custom
  executors) that would change how we structure long-running subscriptions?
- What is the *minimum* Swift toolchain we should pin warp to without
  cutting off the Apple-Silicon-Mac population we care about?

## References

- (To add as research lands.)

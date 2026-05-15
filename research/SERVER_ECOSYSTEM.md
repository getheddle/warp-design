# SERVER_ECOSYSTEM.md — Swift on Server packages

**Status:** TODO (stub).

**Gates:** Foundation-module versions to depend on.

Findings here pin the package selection and version range for warp's
foundation modules — service lifecycle, logging, metrics, tracing, and
ancillary infrastructure.

## Topics to cover

The current Swift on Server Workgroup ecosystem, version status, API
stability, and adoption notes for:

- **swift-service-lifecycle** — service registration / graceful shutdown
- **swift-log** — logging abstraction
- **swift-metrics** — metrics abstraction
- **swift-distributed-tracing** — OTel-compatible tracing
- **swift-async-algorithms** — async sequences / channels
- **swift-system** — POSIX-ish primitives, useful for IPC sockets
- **swift-argument-parser** — CLI surface for `warp` binary
- **swift-crypto** — for trust/admission models (mTLS, signed pairing)
- **swift-collections** — Deque, OrderedDictionary, heap structures
- **swift-http-types** — if we expose any HTTP endpoints
- **SwiftNIO** — when we need raw network primitives below NATS

Plus relevant backends:

- swift-otel for distributed tracing export
- swift-prometheus / swift-statsd for metrics export
- swift-log-otel for unified log/trace correlation

## Implications for v0 to capture

- Concrete package versions for `Package.swift`.
- Which abstractions get a concrete backend in v0 vs. stay
  abstract-only.
- Compatibility with the heddle-sdk Swift package's existing dependencies
  (don't bloat the transitive graph unnecessarily).

## Open questions

- Is `swift-service-lifecycle` mature enough to be load-bearing for
  warp v0, or should we hand-roll the supervision tree initially?
- Do we adopt `swift-distributed-tracing` from day 1 even though the
  Python heddle side uses OpenTelemetry directly? (Answer matters for
  end-to-end trace propagation across the IPC boundary.)
- Which logging backend by default — swift-log's bootstrapping pattern
  + a sensible default, or a warp-specific backend that exports to the
  user's macOS unified log?

## References

- (To add as research lands.)

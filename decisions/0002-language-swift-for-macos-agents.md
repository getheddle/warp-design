# 0002. Swift for the macOS daemon (warp v0 + macOS agents)

- **Date:** 2026-04-29
- **Status:** Accepted (for macOS only; Linux/Windows agent language is a
  separate decision, expected to land as Rust per the language-asymmetry
  preference established here)
- **Deciders:** Hooman (principal — has prior Swift experience), Claude
  Code (sparring partner)

## Context

The warp daemon (and eventual cluster-member agent) needs:

- Native macOS integration (SMAppService for service registration,
  Network.framework, EndpointSecurity, IOKit for power/thermal sensing,
  Keychain Services, TCC declarations via Info.plist `NSPrivacy*` keys,
  Virtualization.framework if we want lightweight workload isolation).
- Code signing + notarization story (Developer ID, hardened runtime,
  Gatekeeper compatibility) so the binary is a trusted entrypoint that
  TCC can reason about.
- Async-friendly daemon ergonomics (signal handling, structured shutdown,
  long-lived network connections, mDNS).
- A productive maintenance story for one developer initially — fast
  iteration, good debugger, low ceremony.

Candidate languages considered:

| Language | Apple-native APIs | Cross-platform | Async | Ecosystem (server) | Memory safety | Ramp-up cost |
|---|---|---|---|---|---|---|
| **C** | yes (verbose) | yes | manual (kqueue) | minimal | manual mgmt — footguns | low if known, otherwise high |
| **Go** | indirect (cgo) | excellent | goroutines + channels | mature (net/http, etc.) | GC, bounds-checked | low |
| **Rust** | indirect (objc/swift bindings) | excellent | tokio (mature) | very mature | safe by default, hard to write | high |
| **Swift** | **native** (ServiceManagement, Network.framework, EndpointSecurity, NSXPCConnection) | macOS strong, Linux real, Windows experimental | structured concurrency | growing (Swift on Server WG) | ARC + safety | low (operator already knows Swift) |

## Decision

**Swift for the macOS daemon.** Specifically Swift 6.x targeting Swift 6.3+
features as they become stable, with conservative use of pitched/under-review
SE proposals (research stream documents which we depend on, in
[`../research/SWIFT_6.md`](../research/SWIFT_6.md)).

Foundation modules from the Swift on Server WG to adopt from day 1 (even
where v0 doesn't fully exercise them) so phase-1+ work is plumbing not
refactor:

- **swift-service-lifecycle** — structured startup/shutdown
- **swift-log** — structured logging with multiple sinks
- **swift-metrics** — capacity reporting later slots into a metric registry
- **swift-distributed-tracing** — spans emitted, no exporter wired by default
- **swift-async-algorithms** — sequence operators we'll need for capacity streams
- **swift-system** — proper FilePath/FileDescriptor abstractions
- **swift-argument-parser** — CLI surface
- **swift-crypto** — for cluster trust + admission control later

Apple-framework primitives v0 commits to:

- **SMAppService** for service registration (replaces hand-managed launchd
  plists; surfaces a proper TCC permission prompt to the user)
- **swift-log + os.Logger backend** so logs flow into Console.app and
  unified logging
- **Network.framework** for local IPC + future cluster networking
- **mDNS via Network.framework's NWBrowser/NWListener** for discovery

Linux/Windows agents are explicitly **deferred to a separate decision**
(expected: Rust). The language asymmetry is accepted as a trade-off: warp
gives up uniformity in exchange for native integration on each platform.
Cross-agent communication is platform-neutral (NATS messages with a
versioned schema), so the asymmetry stays at the agent boundary.

## Consequences

**Easier**

- Apple-native daemon flow: SMAppService → TCC prompt → user grants once →
  works. No more granting FDA to system Python.
- Code signing + notarization is a well-trodden path with Apple toolchains.
- Structured concurrency gives us cancellation-safe lifecycle management
  for free.
- Operator productivity is high (Swift familiarity) and Xcode debugging is
  excellent.
- Swift 6's strict concurrency catches data-race bugs at compile time —
  meaningful for a daemon that handles concurrent capacity reports + RPC.

**Harder**

- Swift on Linux is workable but the Apple-framework code paths obviously
  don't compile there. We have to design the agent in two layers: a
  platform-neutral "warp core" (compiles on Linux too, useful for tests)
  and a platform-specific "macOS adapter" (Apple frameworks). Discipline
  required to keep that boundary clean.
- Linux/Windows agents don't share code with the macOS agent — they share
  protocol. We accept duplicate effort on both sides.
- The Swift-on-Server ecosystem is younger and smaller than Go's or Rust's.
  Some dependencies we'd want (e.g., a mature MCP server crate) may be
  immature or absent. Research stream
  [`../research/SERVER_ECOSYSTEM.md`](../research/SERVER_ECOSYSTEM.md)
  surfaces gaps.
- New contributors who want to work on the macOS agent need Swift skills
  (a smaller pool than Python or Go).

**Trade-offs accepted**

- **Language asymmetry across platforms** rather than one mediocre cross-
  platform language. Apple-native integration is too valuable to give up.
- **Younger server ecosystem** in exchange for native APIs. We adopt the
  Swift Server WG's blessed packages (service-lifecycle, log, metrics,
  tracing) to ride their roadmap rather than rolling our own.
- **Smaller contributor pool** for the macOS agent in exchange for design
  alignment with the platform.

## References

- [Swift on Server Working Group](https://www.swift.org/sswg/)
- [SMAppService — Service Management framework](https://developer.apple.com/documentation/servicemanagement)
- [`../research/SWIFT_6.md`](../research/SWIFT_6.md) (research notes)
- [`../research/SERVER_ECOSYSTEM.md`](../research/SERVER_ECOSYSTEM.md) (research notes)
- [`../research/APPLE_FRAMEWORKS.md`](../research/APPLE_FRAMEWORKS.md) (research notes)
- [`0001-pursue-ad-hoc-cluster-orchestration.md`](0001-pursue-ad-hoc-cluster-orchestration.md)

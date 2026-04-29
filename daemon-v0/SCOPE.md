# Daemon Core v0 — Scope

**Status:** Specification, pre-implementation.
**Implementation lives in:** `getheddle/warp` (to be created when research
streams produce enough findings to commit to architectural choices).

## Goal

Ship a small Swift daemon on macOS that:

1. Replaces launchd's role of supervising the existing Python heddle
   service (currently the ITP Telegram capture + MCP gateway).
2. Lays a deliberate foundation for later phases (capacity reporting,
   cluster scheduling, etc.) — not throwaway code.
3. Solves the immediate operational pain (TCC opacity, hand-managed
   plists, no proper service-registration story) without overcommitting
   to design choices that should wait for research findings.

## In scope for v0

### Process supervision

- Spawn the existing Python heddle service (`baft itp-telegram serve`)
  as a subprocess.
- Stream stdout/stderr to swift-log (which routes to console, file, and
  unified logging).
- Signal handling: SIGTERM → graceful drain → forward to subprocess →
  wait for clean exit (with timeout).
- Restart on subprocess crash with exponential backoff (capped); track
  recent-failure history to refuse-to-restart on persistent failures.
- Health endpoint (HTTP on a UDS or TCP loopback) reporting daemon
  liveness + last-known subprocess state.

### macOS service registration

- Register with launchd via **SMAppService** rather than a
  hand-written plist in `~/Library/LaunchAgents/`. SMAppService surfaces
  a proper TCC permission prompt to the user the first time it's
  enabled, which is the right UX (and the right TCC story) on Sequoia+.
- Provide CLI subcommands: `warp register` / `warp unregister` /
  `warp status`.

### Configuration

- Read `~/.heddle/agent.yaml` (or rename — we'll bikeshed in v0). Schema
  validated at startup with helpful error messages on misconfig.
- Resolve env vars from `~/.heddle/.env` (the project's existing
  convention) without requiring shell sourcing — same trick as
  `python-dotenv`.
- Configurable: subprocess command, working directory, log path, mDNS
  service name, health endpoint port/socket.

### Foundation modules (wired even if not fully used)

| Module | Purpose | v0 use |
|---|---|---|
| **swift-service-lifecycle** | Structured startup/shutdown with dependency ordering | Wire daemon + supervisor + health endpoint as services |
| **swift-log** | Structured logging | Console + rotating file + os.Logger backend |
| **swift-metrics** | Metric registry | Hello-world counter (total_restarts, uptime_seconds); capacity-reporting later slots in |
| **swift-distributed-tracing** | Span emission | Spans on subprocess spawn / signal handling; no exporter wired by default |
| **swift-async-algorithms** | AsyncSequence operators | Used in log streaming + signal handling |
| **swift-system** | FilePath/FileDescriptor abstractions | Robust path handling |
| **swift-argument-parser** | CLI surface | The `warp ...` commands |

### mDNS announcement

- Bonjour-announce the daemon's presence as `_warp._tcp` (or similar —
  bikeshed pending) with a TXT record carrying the daemon version,
  agent ID (UUID), and a hint at capabilities ("v0 — supervisor only,
  no scheduling").
- Foundation for phase-1 peer discovery; v0 just announces, nothing
  consumes it yet.

### IPC with the Python subprocess

- Unix domain socket at `~/.heddle/warp.sock` with a small JSON-NDJSON
  protocol: `{"op": "shutdown"}`, `{"op": "ping"}`, etc.
- Used by the supervisor to: send graceful-shutdown requests, ping for
  liveness, receive the subprocess's "ready" signal.
- Protocol versioned from day 1 (`{"protocol_version": 1, ...}`) so the
  subprocess can evolve independently.

## Explicitly out of scope for v0

- **Capacity reporting.** Foundation is wired (swift-metrics registry),
  but no capacity probes implemented. Phase 1.
- **Cluster join / peer connections.** mDNS announce only. Phase 1.
- **Scheduling.** No scheduler logic. Heddle's existing per-machine
  scheduling is unchanged. Phase 2.
- **Budget tracking.** Phase 3.
- **Hardware advisor.** Phase 4.
- **Native Swift MCP server.** The Python FastMCP server keeps serving
  MCP traffic; the Swift daemon supervises it. Porting MCP to Swift is
  a phase 1+ decision dependent on the research stream
  [`../research/MCP_SWIFT.md`](../research/MCP_SWIFT.md).
- **Workload execution.** No agent-side workload runtime in v0; the
  subprocess does the work. Phase 2.
- **Cross-platform agent.** Linux/Windows agents are a separate
  decision and codebase. Phase 5.
- **Code signing + notarization automation in CI.** Local-developer
  signing only in v0; CI signing is a phase 0+ follow-up.

## Non-functional requirements

- **Startup time:** < 500ms from launch to "subprocess spawned."
- **Memory footprint (resident, daemon only):** < 30 MB.
- **Crash on startup:** never silent; always logs to console.app and
  surfaces a structured error code.
- **Shutdown latency:** < 5s from SIGTERM to clean exit (subprocess gets
  3s to drain; supervisor cleanup gets 1s).
- **Code style:** Swift 6 strict concurrency. Sendable everywhere
  reasonable. Typed throws for the daemon's own error surface.
- **Tests:** Unit tests for the supervisor state machine, the IPC
  protocol, the config validator. Integration test that spawns a real
  subprocess (a tiny test fixture) and exercises the lifecycle.

## v0 module structure (proposed)

```text
warp/
  Package.swift
  Sources/
    Warp/                    — main executable target
      Commands/              — CLI subcommands (register, unregister, run, status)
      App.swift              — composition root
    WarpCore/                — platform-neutral business logic (compiles on Linux too)
      Supervisor/            — subprocess lifecycle state machine
      Config/                — YAML loader + validator
      IPC/                   — UDS + protocol
      Health/                — health endpoint
      Discovery/             — mDNS announce abstraction (platform-neutral interface)
      Foundation/            — service-lifecycle wiring, log + metrics setup
    WarpMacOS/               — macOS-specific adapters
      ServiceManagement/     — SMAppService wrapper
      Discovery/             — Network.framework NWBrowser/NWListener implementation
      OSLog/                 — os.Logger backend for swift-log
  Tests/
    WarpCoreTests/
    WarpMacOSTests/
```

This split keeps `WarpCore` Linux-portable (testable in CI on Linux,
useful as the basis for future agents) while macOS-specific code lives
in `WarpMacOS`. The `Warp` executable composes the two.

## Open architectural questions for v0

Captured here so they're not forgotten when implementation begins:

1. **Service lifecycle library:** swift-service-lifecycle 2.x is current.
   Worth checking 3.x roadmap and whether we should wait. → research.
2. **mDNS implementation:** Network.framework's NWBrowser/NWListener
   (Apple-native) vs. a portable Swift package. → research.
3. **Health endpoint protocol:** plain HTTP/JSON vs. something gRPC-like.
   For v0, plain HTTP is fine; revisit when we have multiple consumers.
4. **Subprocess IPC:** UDS + JSON-NDJSON for v0. Revisit if we need
   high-frequency capacity reports (probably msgpack at that point).
5. **Config file location:** `~/.heddle/agent.yaml` keeps config under
   one tree; `~/Library/Application Support/Warp/config.yaml` is more
   macOS-correct. Bikeshed pending.
6. **Where does the daemon log to?** Currently the Python service logs
   to `~/.heddle/itp_telegram.log`; the Swift daemon will need its own
   log + a way to forward subprocess logs into it.
7. **Foreground or daemonized mode in development?** SMAppService
   supports both; we want to pick a default that's debugger-friendly.

## Done definition

v0 is "done" when:

- `warp register` registers the daemon via SMAppService and the user
  sees the proper macOS permission prompt.
- After registration, the daemon auto-starts at login, supervises the
  Python service, restarts it on crash, and stops cleanly on `warp stop`
  (which translates to a SIGTERM the daemon handles).
- `warp status` shows daemon + subprocess liveness, last restart time,
  recent log tail.
- mDNS announcement is visible in `dns-sd -B _warp._tcp`.
- All foundation modules (lifecycle, log, metrics, tracing) are wired
  and emit at least one signal each (verified in tests).
- Repo has CI green on macOS for Swift 6.x; Linux CI compiles
  `WarpCore` (no SMAppService etc.) but doesn't run integration tests.
- README + CHANGELOG.md document the v0 surface.

When this lands, the existing ITP Telegram daemon stops being
launchd-managed via the hand-written plist and starts being warp-managed.
That's the operational migration v0 ships.

# Research Notes

Working notes from the research streams that gate v0 implementation.

Each file targets one stream. The deliverable per stream is "we know
enough to make defensible v0 architectural choices" — not a comprehensive
literature review.

## Streams

| File | Topic | Status | Gates |
|---|---|---|---|
| `SWIFT_6.md` | Swift 6.0–6.2 shipped features relevant to daemon design; 6.3+ pitched/under-review SE proposals worth designing forward-compat with | TODO | Module structure, concurrency model, error-handling style |
| `SERVER_ECOSYSTEM.md` | Swift on Server WG packages: SwiftNIO, swift-service-lifecycle, swift-log, swift-metrics, swift-distributed-tracing, swift-async-algorithms, swift-system, swift-argument-parser, swift-crypto, swift-collections, swift-http-types | TODO | Foundation-module versions to depend on |
| `APPLE_FRAMEWORKS.md` | SMAppService, Network.framework, EndpointSecurity, IOKit power/thermal, Virtualization.framework, Keychain Services, TCC declarations via Info.plist `NSPrivacy*` keys | TODO | Service registration approach, mDNS implementation, future capacity-probe primitives |
| `MCP_SWIFT.md` | Anthropic's official Swift SDK for MCP, community alternatives, fastmcp-equivalents | TODO (lower priority — v0 keeps Python MCP) | Phase-1+ decision on porting MCP server to Swift |
| `DISTRIBUTION.md` | Code signing + notarization for unmanaged personal binaries — Developer ID, hardened runtime, notarytool, Gatekeeper, automating in CI | TODO | Release pipeline, CI signing strategy |

## Conventions

- **TODO files** are committed empty-but-with-TOC so the gap is visible
  in the repo.
- **Research findings** become ADRs in `../decisions/` when they pin
  down a choice; the research file then references the ADR.
- **Open questions** in a research file are tracked there until resolved
  or punted to a separate ADR.
- **Sources cited** as Markdown links inline — no separate bibliography.

## Process

1. Pick the highest-leverage stream (the one whose findings unblock the
   most v0 decisions).
2. Spend a focused block — half-day to a day — gathering current state
   of the art.
3. Write findings into the relevant research file. Note 3-5 concrete
   implications for v0.
4. If a finding pins down an architectural choice, write an ADR in
   `../decisions/` and link from the research file.
5. Review the [`../daemon-v0/SCOPE.md`](../daemon-v0/SCOPE.md) "open
   architectural questions" list and resolve what we can.

## What "done" looks like for the research phase

- Every `../daemon-v0/SCOPE.md` open question has a defensible answer
  (with the reasoning logged).
- Foundation-module versions are pinned in a draft `Package.swift`.
- We have a clear point-of-departure for code with no remaining "we'll
  decide this when we get there" items in v0 scope.

When all five files are filled in (or punted to phase 1+ with reasoning),
the research phase ends and implementation begins in `getheddle/warp`.

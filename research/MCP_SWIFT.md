# MCP_SWIFT.md — MCP support in Swift

**Status:** TODO (stub, lower priority — v0 keeps Python MCP).

**Gates:** Phase-1+ decision on porting the MCP server to Swift.

In warp v0, MCP responsibilities stay in the existing Python heddle
process. Later phases may consolidate the MCP server into the Swift
daemon so that warp ships as a single binary. This file captures
the research that would unblock that decision.

## Topics to cover

- **Anthropic's official Swift MCP SDK** — current state, API
  stability, transport support (stdio, HTTP+SSE, WebSocket), feature
  parity with the Python `fastmcp` library that heddle uses today.
- **Community Swift MCP libraries** — alternative implementations, any
  with better ergonomics or smaller footprints.
- **Heddle's MCP surface** — what tools, prompts, and resources are
  exposed today (`heddle.mcp.bridge`, the Workshop bridge, the session
  bridge); how complex a port would be.
- **Hosting model** — does the Swift daemon become the MCP server
  process directly, or proxy to the Python side? Implications for
  startup time, dependency size, and which side owns conversation state.

## Implications for v0 to capture

- None for v0 — the current Python MCP server stays. This file exists
  so phase 1+ has a starting point.

## Open questions

- Is the official Swift MCP SDK API-stable enough to pin in a daemon
  release, or still iterating too rapidly?
- What's the operational benefit of consolidating MCP into Swift vs.
  keeping the Python side as the MCP host? Is the gain worth the port?
- If we keep MCP in Python, what's the IPC pattern between warp (Swift)
  and the MCP-serving heddle process (links to
  `../daemon-v0/SCOPE.md` IPC discussion)?

## References

- (To add as research lands.)

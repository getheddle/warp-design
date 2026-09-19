# CLAUDE.md — warp-design

The canonical agent instructions for this repository live in
[`AGENTS.md`](AGENTS.md). Read that first.

Cross-repo guidance (philosophy, invariants, wire-protocol contract,
skills, and subagents) lives in
**[`../heddle-workspace/`](../heddle-workspace/)** — installed
into this repo's `.claude/` via the toolkit's `install.sh`.

## Claude-specific notes

- When session history is compacted, recover direction from
  `AGENTS.md` + `EVOLUTION_LOG.md` (last few entries) + `git status`.

If this file conflicts with `AGENTS.md`, follow `AGENTS.md` and the
current user request.

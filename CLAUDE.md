# CLAUDE.md — warp-design

The canonical agent instructions for this repository live in
[`AGENTS.md`](AGENTS.md). Read that first.

Cross-repo guidance (philosophy, invariants, wire-protocol contract,
skills, and subagents) lives in
**[`../heddle-agent-toolkit/`](../heddle-agent-toolkit/)** — installed
into this repo's `.claude/` via the toolkit's `install.sh`.

## Claude-specific notes

- When session history is compacted, recover direction from
  `AGENTS.md` + `EVOLUTION_LOG.md` (last few entries) + `git status`.
- For ADR work, use the `/warp-adr` skill. It produces files matching
  the established `decisions/NNNN-*.md` format with Status / Context /
  Decision / Consequences sections.
- Cross-repo invariant C7: warp-design ADRs may *propose* changes to
  `heddle` or `heddle-sdk`. The implementation goes through those
  repos' normal PR flow — do not silently change framework behavior by
  referencing this design repo.
- Keep entries dense. Every sentence in an ADR earns its place.

If this file conflicts with `AGENTS.md`, follow `AGENTS.md` and the
current user request.

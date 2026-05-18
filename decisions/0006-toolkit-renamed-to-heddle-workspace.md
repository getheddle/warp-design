# 0006. Toolkit renamed: `heddle-agent-toolkit` → `heddle-workspace`

- **Date:** 2026-05-17
- **Status:** Accepted
- **Deciders:** Hooman (principal), Claude Code (sparring partner)
- **Supersedes:** [0005](0005-sibling-relative-paths-for-toolkit-references.md)

## Context

ADR [0005](0005-sibling-relative-paths-for-toolkit-references.md)
established the sibling-relative-path convention for cross-repo
references *within the toolkit-anchored agent layer*. At the time,
the toolkit lived at
[`getheddle/heddle-agent-toolkit`](https://github.com/getheddle/heddle-workspace)
and its mission was narrow: shared agent guidance (anchors, skills,
subagents).

Subsequent work widened the toolkit's mission. In addition to the
agent-tooling pillar, it now also owns **workspace lifecycle**: the
umbrella-repo design, the `.heddle-workspace.yaml` manifest, the
`bin/workspace` CLI (`init` / `link` / `sync` / `status` / `add` /
`rm` / `doctor`), and the bootstrap recipe for moving a workspace
between machines or merging two divergent local copies. See the
canonical spec in
[`heddle-workspace/docs/WORKSPACE_SYNC_DESIGN.md`](https://github.com/getheddle/heddle-workspace/blob/main/docs/WORKSPACE_SYNC_DESIGN.md).

With that broader scope, the name `heddle-agent-toolkit` no longer
fit. It read as a single-purpose repo (agent tooling) and obscured the
fact that workspace bootstrap and sync now live there too.

## Decision

Rename the repository:

> `getheddle/heddle-agent-toolkit` → `getheddle/heddle-workspace`

GitHub auto-redirects the old clone URLs, so any existing checkout
continues to work without rewriting its `origin`. Inside the family,
all sibling-relative path references update from
`../heddle-agent-toolkit/...` to `../heddle-workspace/...`, and the
local directory name follows the repo name.

The sibling-relative convention from [0005](0005-sibling-relative-paths-for-toolkit-references.md)
is **unchanged in principle**. The agent layer (`AGENTS.md`,
`CLAUDE.md`, `.claude/`, and any file inside `heddle-workspace/`
itself) may use sibling-relative paths; user-facing documentation
continues to use full GitHub URLs per ADR
[0003](0003-warp-and-warp-design-repo-split.md). Only the directory
name changes; the rule does not.

## Consequences

**Easier**

- The name now reflects both pillars (agent tooling *and* workspace
  lifecycle). A new contributor reading the GitHub org page sees the
  scope of the repo from its name without needing to open the README.
- The `bin/workspace` CLI's path inside the repo (`heddle-workspace/bin/workspace`)
  reads consistently with its command name.

**Harder**

- Every consumer repo (`heddle`, `heddle-sdk`, `warp-design`,
  `getheddle.github.io`, the org `.github` profile) carries a
  sibling-relative path mention that has to be updated. Mitigated by a
  one-shot rename PR per repo.
- This ADR is the second one in the family that supersedes a prior
  decision (after 0004's clarifications). Future ADRs should plan for
  similar renames by using the superseded-by mechanism rather than
  editing prior ADRs in place.

**Trade-offs accepted**

- We accept the one-time multi-repo update cost in exchange for a
  name that matches the repo's widened scope. The alternative
  (keeping `heddle-agent-toolkit` and treating workspace lifecycle as
  a hidden sub-feature) optimizes for short-term zero-rename cost at
  the price of an indefinite naming mismatch.

## References

- [`0003-warp-and-warp-design-repo-split.md`](0003-warp-and-warp-design-repo-split.md) — cross-repo link rule (full URLs in user-facing docs)
- [`0005-sibling-relative-paths-for-toolkit-references.md`](0005-sibling-relative-paths-for-toolkit-references.md) — superseded by this ADR
- [`heddle-workspace/docs/WORKSPACE_SYNC_DESIGN.md`](https://github.com/getheddle/heddle-workspace/blob/main/docs/WORKSPACE_SYNC_DESIGN.md) — the workspace-lifecycle design that motivated the rename

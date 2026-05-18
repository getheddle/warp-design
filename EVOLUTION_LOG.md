# Evolution Log

Chronological record of warp's design evolution — insights, decisions, and
course-corrections, append-only. Newest entries at the bottom.

Use this log to answer "why did we end up here?" Each entry is dated, has a
short title, and points at the artifacts (ADRs, exploration notes, code) it
produced.

---

## 2026-04-29 — Origin

**Trigger.** A conversation with friends sparked from a YouTube video about
AI compute futures. Hooman's argument crystallized in two AI-generated
artifacts (an infographic poster and a "fake whiteboard" portrait) titled
*The Future of AI Compute: Edge + Cloud Working Together*. Core thesis:
Apple Silicon and hyperscaler chips optimize for orthogonal axes
(personal-best vs. globally-cheapest) and the AI future needs both layers
working together — not one substituting for the other.

The whiteboard included a "small business Apple cluster" panel showing four
Macs (iMac, MacBook Pro, Mac mini, Mac Studio) sharing idle compute over a
local network for ad-hoc workloads, bursting to cloud only when needed.

**Insight.** That panel describes exactly what a heddle deployment across
sibling Macs would do — and nobody is building it as a coherent product
positioned for personal/SMB use. The K8s-shaped ecosystems are wrong-fit
for SMB; the local-only stacks (Ollama, LM Studio) lack a clustering or
cloud-arbitrage story; the cloud-only stacks force every workload through
a metered third party.

**Decision.** Pursue the ad-hoc cluster orchestrator vision as heddle's
strategic direction. Start with macOS (Apple Silicon's mixed-load
performance + Thunderbolt M2M + privacy primitives + Hooman's Swift
familiarity make it the right v0 environment). Expand to Linux/Windows
later via a separate Rust agent.

See [`VISION_AD_HOC_CLUSTERS.md`](VISION_AD_HOC_CLUSTERS.md) for the full
articulation, and [`decisions/0001-pursue-ad-hoc-cluster-orchestration.md`](decisions/0001-pursue-ad-hoc-cluster-orchestration.md)
for the formal record.

---

## 2026-04-29 — Triggering context: the TCC episode

**What happened.** In the immediately preceding session, while building a
Telegram capture daemon for ITP, the team discovered that macOS Sequoia's
TCC blocks launchd-spawned processes from reading external volumes
(anything under `/Volumes/`) without an explicit Full Disk Access grant on
the exec'd binary. Five rapid crash-and-retry cycles in launchd were the
first symptom. Probing confirmed the OS-level restriction.

The interim fix — granting FDA to `/usr/local/python-3.12.13/bin/python3.12`
and using a smarter probe-based pre-check in `install.sh` — works, but
felt fragile: granting broad FDA to a system-wide Python interpreter,
hand-managing launchd plists, no clean code-signing story, no proper
service-registration model.

**Insight.** A native daemon binary with a clear identity (code-signed,
TCC-declared via Info.plist, registered via SMAppService) would be the
right primitive — both for solving the immediate ops pain *and* for being
the foundation of the eventual cluster-member agent in the bigger vision.

**Course-correction.** What started as a small "tiny launcher to make
launchd happy" question became "the cluster member agent." That
substantially changes scope and language calculus.

---

## 2026-04-29 — Language and naming

**Decisions made:**

- **Language: Swift** for the macOS daemon. Reasons captured in
  [`decisions/0002-language-swift-for-macos-agents.md`](decisions/0002-language-swift-for-macos-agents.md).
- **Repo split: warp + warp-design.** Production code in `getheddle/warp`;
  design + research + ADRs + evolution log in `getheddle/warp-design`
  (this repo). See
  [`decisions/0003-warp-and-warp-design-repo-split.md`](decisions/0003-warp-and-warp-design-repo-split.md).
- **Name: warp.** Dual metaphor — *warp threads* in weaving are the
  longitudinal foundation threads (a heddle controls warp threads), and
  *warp drive* in Star Trek is faster-than-light coordination. Both
  resonate with "foundation layer for fast inter-node coordination." See
  [`decisions/0004-naming-warp.md`](decisions/0004-naming-warp.md).

---

## 2026-04-29 — Phased plan and v0 scope

**Phasing agreed:**

0. Daemon core (Swift, on one machine, supervises existing Python service,
   lays foundation modules)
1. Capacity reporter (multi-node visibility)
2. Capacity-aware routing (first real ad-hoc clustering)
3. Budget + cost arbitrage
4. Hardware advisor
5. Linux + Windows agents
6. UX polish

**v0 scope** captured in [`daemon-v0/SCOPE.md`](daemon-v0/SCOPE.md).
Explicit non-goals: no scheduling, no cluster joining beyond mDNS
announcement, no native Swift MCP server in v0 (Python FastMCP keeps
serving). Foundation modules — swift-service-lifecycle,
swift-distributed-tracing, swift-metrics — included from day 1 even though
v0 doesn't fully exercise them, so phase-1+ work is plumbing not refactor.

**Research streams kicked off** before v0 implementation begins:
Swift 6.x language state, Swift-on-Server ecosystem, Apple-native daemon
APIs, MCP in Swift, code signing + notarization. Notes will land in
[`research/`](research/).

---

## 2026-05-17 — Toolkit renamed to `heddle-workspace`

**Trigger.** The shared toolkit's mission widened beyond agent guidance:
in addition to anchors/skills/subagents, it now also owns the
workspace lifecycle (umbrella git repo, `.heddle-workspace.yaml`
manifest, `bin/workspace` CLI). The name `heddle-agent-toolkit`
obscured the second pillar.

**Insight.** Repo names are read more often than READMEs. A name that
reflects only half of a repo's scope creates a steady tax on every
contributor who has to discover the other half.

**Decision.** Rename `getheddle/heddle-agent-toolkit` →
`getheddle/heddle-workspace`. GitHub auto-redirects clone URLs;
sibling-relative path references update across the family in paired
PRs. The sibling-relative convention from ADR 0005 is preserved
unchanged — only the directory name moves.

**Artifacts.**
- [`decisions/0006-toolkit-renamed-to-heddle-workspace.md`](decisions/0006-toolkit-renamed-to-heddle-workspace.md) — formalizes the rename, supersedes [0005](decisions/0005-sibling-relative-paths-for-toolkit-references.md)
- [`heddle-workspace/docs/WORKSPACE_SYNC_DESIGN.md`](https://github.com/getheddle/heddle-workspace/blob/main/docs/WORKSPACE_SYNC_DESIGN.md) — the workspace-lifecycle design that motivated the wider scope

---

## Template for future entries

```markdown
## YYYY-MM-DD — Short title

**Trigger.** What prompted this entry — a discussion, a research finding,
a production observation, a course-correction.

**Insight.** What we now understand that we didn't before, or what
hypothesis was disconfirmed.

**Decision.** What we will do (or won't do). Link to the ADR that formalizes
it if relevant.

**Artifacts.** Links to docs, code, or commits this entry produced.
```

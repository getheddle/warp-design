# AGENTS.md — warp-design

`warp-design` is the design-only repository for **warp**, a planned
Swift daemon that extends Heddle into ad-hoc personal/SMB cluster
orchestration on macOS. This repo holds vision, architecture sketches,
research notes, and Architecture Decision Records — no production code.

Production code lives in a separate repo (planned: `getheddle/warp`).

This file is the source of truth for agent guidance in *this* repo.
Cross-repo guidance lives in
**[`heddle-agent-toolkit/`](../heddle-agent-toolkit/)** — read those
anchors before proposing changes that affect `heddle` or `heddle-sdk`.

## Toolkit install

The toolkit is sibling to this repo. To populate `.claude/skills/` and
`.claude/agents/` from a fresh clone:

```bash
git clone https://github.com/getheddle/heddle-agent-toolkit.git ../heddle-agent-toolkit
../heddle-agent-toolkit/install.sh .
```

Until the toolkit is published, contributors will need a local sibling
checkout. The `/warp-adr` skill and the `heddle-architect` subagent
named in this doc come from there.

## Read first

### From the toolkit (shared)

- `heddle-agent-toolkit/anchors/ECOSYSTEM.md` — repo map. warp-design's
  role is "informs the planned `warp` repo."
- `heddle-agent-toolkit/anchors/PHILOSOPHY.md` — the design opinions
  that constrain what warp should and shouldn't become.
- `heddle-agent-toolkit/anchors/INVARIANTS.md` — cross-repo invariant
  C7 specifically: warp-design ADRs may *propose* changes to `heddle` or
  `heddle-sdk`; the implementation goes through those repos' normal PR
  flow.

### From this repo

- `README.md` — repo purpose and conventions.
- `VISION_AD_HOC_CLUSTERS.md` — the headline thesis and phased plan.
- `EVOLUTION_LOG.md` — chronological record of insights and decisions.
  **This repo intentionally does not have a `CHANGELOG.md`** — it
  produces no released artifacts. `EVOLUTION_LOG.md` plus the
  numbered ADRs in `decisions/` serve as the durable design-history
  log. Sibling repos in the family (`heddle`, `heddle-sdk`,
  `heddle-agent-toolkit`) do maintain `CHANGELOG.md` files for
  behavioural changes.
- `decisions/` — ADRs (numbered, `NNNN-kebab-case-title.md`).
- `exploration/` — architectural sketches and prior-art surveys.
- `daemon-v0/SCOPE.md` — the v0 deliverable scope.
- `research/` — research notes by stream.

## Conventions

- **Dates:** ISO 8601, `YYYY-MM-DD`.
- **ADRs:** `decisions/NNNN-kebab-case-title.md`. Standard sections:
  Status, Context, Decision, Consequences (Easier / Harder / Trade-offs
  accepted), References. See
  `decisions/0001-pursue-ad-hoc-cluster-orchestration.md` as the
  reference format. The toolkit's `/warp-adr` skill produces ADRs that
  match this format.
- **Evolution log:** Append-only, newest entries at the bottom. Each
  entry stamped with date + short title.
- **Markdown:** GitHub-flavored. Diagrams as ASCII first; Mermaid if
  more is needed.
- **Research stubs:** A research file with TOC + open questions counts
  as committed-in-progress; finished research drops the open-questions
  section.

## Out of scope (and why)

- **Production code.** Lives in `getheddle/warp` (planned).
- **Heddle Python framework changes.** Live in `getheddle/heddle`. ADRs
  here may *propose* changes there, but the implementation goes through
  heddle's PR flow (cross-repo invariant C7).
- **End-user docs for warp.** Lives in `getheddle/warp` once the daemon
  ships. This repo is for designers and contributors, not users.

## When working in this repo

The work in this repo is almost always one of:

- Writing or updating an ADR → use `/warp-adr`.
- Appending to `EVOLUTION_LOG.md`.
- Adding to `exploration/` (sketches, prior-art surveys).
- Adding to `research/` (research stubs become finished docs over time).
- Updating `VISION_AD_HOC_CLUSTERS.md` when direction shifts.

`heddle-architect` (toolkit subagent) is still useful when the design
work proposes a change to `heddle` or `heddle-sdk` — it can map the
cross-repo seam before the ADR is written.

## Review checklist (this repo)

Before committing:

- Is the date ISO 8601?
- Is the ADR filename `NNNN-kebab-case-title.md`?
- Does the ADR have Context / Decision / Consequences / References?
- Does the EVOLUTION_LOG entry have a date and title?
- If this ADR proposes a change to `heddle` or `heddle-sdk`: is the
  proposal scoped (no implementation detail dictating PR shape)?

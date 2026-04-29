# warp-design

Design exploration, decision records, research notes, and evolution log for
**warp** — a personal/SMB ad-hoc cluster orchestrator built on the
[heddle](https://github.com/getheddle/heddle) framework.

This repository captures the *thinking* behind warp. Production code lives
in a separate repo (planned: `getheddle/warp`). The split is deliberate:
warp stays focused on shipping; warp-design stays focused on showing the
work — alternatives considered, trade-offs accepted, and how the design
evolves over time.

## Status

**Pre-implementation.** The vision is articulated, the architectural
sketch is on paper, and a small daemon-core (v0) is scoped. No production
code yet. Research into Swift 6.x and the Swift-on-Server ecosystem is
the gating step before implementation begins.

## What's here

| Path | Contents |
|---|---|
| `VISION_AD_HOC_CLUSTERS.md` | The headline argument: why ad-hoc personal/SMB compute clusters matter, why Apple Silicon is the right starting point, what warp will become |
| `EVOLUTION_LOG.md` | Chronological log of insights, decisions, course-corrections — the "how we got here" record |
| `decisions/` | Lightweight ADRs (Architecture Decision Records). One file per decision. Numbered chronologically. |
| `exploration/` | Architectural sketches and prior-art surveys — design space exploration, not commitments |
| `daemon-v0/` | Spec for the immediate Swift daemon-core deliverable that lays foundation without overcommitting |
| `research/` | Notes from research streams (Swift 6.x, Swift-on-Server, Apple frameworks, MCP, distribution) |

## Relationship to other repos

```text
                    ┌──────────────────────────────────────┐
                    │  github.com/getheddle/heddle         │
                    │  (Python actor mesh framework)       │
                    │  — production code                   │
                    │  — warp's runtime substrate          │
                    └──────────────┬───────────────────────┘
                                   │
                                   │ implements vision from
                                   │ (referenced in heddle/docs/vision/)
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  github.com/getheddle/warp-design    │
                    │  (this repo — design + research)     │
                    │  — vision, ADRs, evolution log       │
                    │  — research notes                    │
                    │  — exploration sketches              │
                    └──────────────┬───────────────────────┘
                                   │
                                   │ informs
                                   ▼
                    ┌──────────────────────────────────────┐
                    │  github.com/getheddle/warp           │
                    │  (planned — Swift daemon code)       │
                    │  — production code                   │
                    │  — release-tagged                    │
                    │  — code signed + notarized           │
                    └──────────────────────────────────────┘
```

The full vision doc also lives in heddle as
`heddle/docs/vision/AD_HOC_CLUSTERS.md` (canonical project-direction
copy) — the version here may run ahead of it during exploration.

## Conventions

- **Dates:** ISO 8601, YYYY-MM-DD.
- **ADRs:** Numbered `NNNN-kebab-case-title.md`. Standard sections:
  Status, Context, Decision, Consequences. See
  [`decisions/0001-pursue-ad-hoc-cluster-orchestration.md`](decisions/0001-pursue-ad-hoc-cluster-orchestration.md)
  for the format.
- **Evolution log:** Append-only, newest entries at the bottom. Each
  entry stamped with date + short title.
- **Markdown:** GitHub-flavored. Diagrams as ASCII first; if a diagram
  needs more than ASCII, use Mermaid.
- **Research stubs:** A research file with TOC + open questions counts
  as committed-in-progress; finished research drops the open-questions
  section.

## Out of scope (and why)

- **Production code.** Lives in `getheddle/warp`.
- **Heddle Python framework changes.** Lives in `getheddle/heddle`. ADRs
  here may *propose* changes there, but the implementation goes through
  heddle's normal PR flow.
- **End-user docs for warp.** Lives in `getheddle/warp` once the daemon
  ships. This repo is for designers and contributors, not users.

## License

To be decided. Likely Apache-2.0 to match heddle. (See
`decisions/000X-license.md` when written.)

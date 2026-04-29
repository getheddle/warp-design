# 0001. Pursue ad-hoc cluster orchestration as heddle's strategic direction

- **Date:** 2026-04-29
- **Status:** Accepted
- **Deciders:** Hooman (principal), Claude Code (sparring partner)

## Context

Heddle today is a Python actor mesh framework (NATS, MCP gateway, RAG
pipeline, scheduler, Workshop, TUI, distributed tracing) with strong
single-machine and traditional-cloud-cluster deployment stories. It does
not have a story for personal/SMB deployments where the operator is not a
Kubernetes administrator and the fleet is heterogeneous (a few Macs, maybe
a Jetson, occasional cloud burst).

The conversation about *The Future of AI Compute: Edge + Cloud Working
Together* (see [`../EVOLUTION_LOG.md`](../EVOLUTION_LOG.md), 2026-04-29
Origin entry) crystallized a thesis: Apple Silicon and hyperscaler chips
optimize for orthogonal axes (personal-best vs. globally-cheapest) and the
AI future is hybrid — *both layers, working together*. Today's tooling
treats either axis in isolation:

- Local-only stacks (Ollama, LM Studio) lack clustering and cloud arbitrage.
- Cloud-only stacks lack privacy and local-cost-arbitrage.
- Heavyweight orchestrators (K8s + KubeFlow + Run:ai) assume homogeneity,
  ops teams, and uniform OS — wrong-shaped for SMB.

The gap is a control plane that treats personal/SMB clusters as
first-class: heterogeneous, privacy-aware, cost-conscious, ad-hoc, and
operable by someone whose job is not "Kubernetes administrator."

Heddle has the substrate (NATS, actors, pipelines, mDNS, multi-runtime
backend abstraction) to fill that gap. The net-new pieces are well-bounded
(node agent, capacity model, scheduler, budget controller, migration
engine, hardware advisor, trust model, end-user UX).

## Decision

Adopt **ad-hoc personal/SMB cluster orchestration** as heddle's stated
strategic direction. Implement it as a phased delivery (see
[`../VISION_AD_HOC_CLUSTERS.md`](../VISION_AD_HOC_CLUSTERS.md) "Phased
delivery" table). Begin with v0 (Swift daemon-core on macOS) and grow from
there, with architectural choices in v0 made explicitly forward-compatible
with phases 1-6.

The implementation lives in a new repo `getheddle/warp` (Swift). Design,
research, and decision history live in `getheddle/warp-design` (this repo).

## Consequences

**Easier**

- Heddle gets a coherent product narrative for the next 18-24 months.
- The thesis attracts contributors who care about edge AI + privacy.
- The phased plan is independently-shippable per phase, so we can stop or
  pivot at any phase boundary without sunk cost beyond that phase.
- macOS-first scope is small enough that v0 can ship in weeks, not quarters.

**Harder**

- Heddle's existing audience (developers building NATS-based actor systems
  for traditional cloud clusters) needs to be told this is *additive*, not
  a pivot away from them. Documentation and roadmap framing matter.
- A second language (Swift) enters the project. Linux/Windows agents will
  add a third (likely Rust). Toolchain, CI, release cadence all multiply.
- The hardware-advisor phase needs longitudinal data — cold-start problem
  that only resolves after months of real-world use. Bootstrap with
  rules-of-thumb tables.
- Productizing for SMB means a real UX, not just a CLI. Different skillset
  than what the project has today.

**Trade-offs accepted**

- We accept the language asymmetry (Swift on macOS, Rust on Linux/Windows)
  rather than trying to be uniform with one cross-platform language. The
  Apple-native integration story is too valuable to give up for uniformity.
- We accept that K8s users may dismiss warp as "K8s lite." The position
  isn't "K8s, but smaller" — it's "the wrong shape if you have an ops team,
  the right shape if you don't." Living with that mis-perception is fine.
- We accept that v0 doesn't immediately deliver clustering. The
  forward-compatible foundation matters more than time-to-first-cluster.

## References

- [`../VISION_AD_HOC_CLUSTERS.md`](../VISION_AD_HOC_CLUSTERS.md)
- [`../EVOLUTION_LOG.md`](../EVOLUTION_LOG.md)
- Original infographic + whiteboard artifacts (Hooman's local files; not
  redistributed in this repo for now)

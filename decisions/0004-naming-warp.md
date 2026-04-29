# 0004. Naming: warp

- **Date:** 2026-04-29
- **Status:** Accepted

## Context

The daemon-core / cluster-member-agent project needs a name. Constraints:

- Short, pronounceable, distinctive.
- Should fit heddle's weaving metaphor (heddle = the loom component that
  controls warp threads).
- Ideally has a second resonant meaning that reinforces the technical role.
- Available as a package/binary name without confusion with major
  established projects.

Candidates considered:

| Name | Weaving meaning | Other meanings | Conflict |
|---|---|---|---|
| **shuttle** | The device that carries weft thread between heddles | Spacecraft (NASA Shuttle); SSH config tool (small) | Some package collisions |
| **reed** | Holds threads at correct spacing in the loom | A musical reed | Cleanish |
| **warp** | The longitudinal foundation threads | Star Trek warp drive (FTL coordination); temporal/dimensional warping | Cloudflare's "Warp" VPN is the loud collision; many smaller |
| **heddle-agent** | Direct, descriptive | None | Cleanest, dullest |
| **loom** | Already taken by heddle's predecessor name | n/a | Internal historical baggage |

## Decision

**warp.** The dual metaphor lands well:

- **Weaving sense**: warp threads are the longitudinal foundation that the
  whole fabric is built on. A heddle controls warp threads — so warp (the
  daemon) and heddle (the framework) form an apt pair where warp is the
  foundation layer and heddle is what sits on top of it. This is exactly
  the architectural relationship we want: warp is the per-node primitive,
  heddle's higher-level abstractions ride on top.
- **Star Trek sense**: warp drive enables faster-than-light coordination
  between distant points. For a project whose distinguishing technical
  bet is on Thunderbolt's 40-80 Gbps M2M links making intra-cluster
  communication essentially free, "warp" is on-brand.

The Cloudflare Warp collision is real but the project type is so different
(consumer VPN vs. cluster-member daemon) that operator confusion is
unlikely in context. We'll need to be specific in package names and search
contexts:

- Repo: `getheddle/warp` (org-prefixed)
- Binary: `warp` (with `warp-agent` as a possible later rename if the
  unprefixed name causes friction in the wild)
- Swift package: `Warp` (capitalized per Swift convention)
- Service label: `com.getheddle.warp`

## Consequences

**Easier**

- Strong, evocative name with two reinforcing meanings.
- Pairs naturally with "heddle" in conversation and writing ("warp runs on
  every node, heddle orchestrates them").
- Star Trek connotation is friendly and approachable, helps signal "this
  is for everyone, not just enterprise ops."

**Harder**

- Cloudflare Warp shows up first in unscoped Google searches. Documentation
  needs to disambiguate explicitly when first mentioning the name in any
  given doc.
- "warp" is sometimes used informally as a verb in image/video processing
  ("warp this perspective") — minor noise in some search contexts.

**Trade-offs accepted**

- Search collision with Cloudflare Warp is real. We accept it because the
  metaphor fit is too good to give up. Fully-qualified names and an
  obvious heddle-context prefix mitigate enough.

## References

- [`0001-pursue-ad-hoc-cluster-orchestration.md`](0001-pursue-ad-hoc-cluster-orchestration.md)
- [`0003-warp-and-warp-design-repo-split.md`](0003-warp-and-warp-design-repo-split.md)

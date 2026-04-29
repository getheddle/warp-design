# 0003. Split warp into two repos: `warp` (code) + `warp-design` (thinking)

- **Date:** 2026-04-29
- **Status:** Accepted

## Context

We need a home for warp's code, and a home for the design exploration,
research notes, ADRs, and evolution log. Three options:

1. **Single repo** — code + design colocated, organized as `warp/` (code)
   and `warp/docs/design/` (thinking).
2. **Two repos** — `getheddle/warp` for code, `getheddle/warp-design` for
   thinking.
3. **Three repos** — `getheddle/warp` for code, `getheddle/warp-design` for
   in-progress thinking, with finalized docs absorbed back into heddle's
   docs over time.

Option 1 keeps everything in one place but conflates two different audiences
(code consumers vs. design contributors) and two different velocity profiles
(production releases vs. exploratory note-taking that may be wrong).

Option 3 adds a flow boundary that's not yet justified. We can always
promote finalized docs back to heddle later — the choice between 2 and 3
is essentially "lazy promotion" vs. "eager promotion," and lazy is cheaper.

## Decision

**Two repos, public from day 1, both under `getheddle/`:**

- **`getheddle/warp`** — production Swift code. Released, versioned,
  code-signed + notarized binaries from CI. End-user docs live here. Issues
  about bugs and feature requests land here. Will be created when v0 code
  begins (after the research streams produce enough findings).
- **`getheddle/warp-design`** — this repo. Design exploration, ADRs,
  research notes, evolution log. No production code. Public so anyone can
  see *how* and *why* design decisions were made. Cloned by contributors
  next to `heddle/`, `baft/`, etc. — same sibling-directory convention the
  ITP analyst already uses.

The vision document is canonicalized to **`heddle/docs/vision/AD_HOC_CLUSTERS.md`**
to signal heddle's project direction. The warp-design copy may run ahead
of heddle's during exploration; at phase boundaries the canonical heddle
copy is updated.

## Consequences

**Easier**

- Code repo stays focused on shipping (no clutter from speculative design
  work).
- Design repo can move at exploratory speed (commits with "this idea didn't
  pan out, here's why" are valuable here, embarrassing in a code repo).
- Different audiences see different default surfaces (code consumers land
  on `warp`; contributors and curious observers land on `warp-design`).
- Public from day 1 means the evolution log captures the *real* trajectory,
  not a sanitized post-hoc retelling.

**Harder**

- Two repos to keep in sync. Cross-references between them must be careful
  not to assume a common file tree. Use full GitHub URLs in cross-repo
  links; relative links only within a single repo.
- Searching across the project requires either GitHub search or `grep -r`
  across both clones.
- A design doc that a code contributor really needs may be in the other
  repo — we'll need to surface key links from `warp/README.md` to the
  relevant `warp-design/decisions/NNNN-*.md`.

**Trade-offs accepted**

- We accept the cross-repo navigation cost in exchange for clean separation
  of concerns.
- We accept that the design repo's quality will be uneven (notes-in-progress,
  abandoned exploration paths, etc.). That's the point — it shows the work.

## References

- [`0001-pursue-ad-hoc-cluster-orchestration.md`](0001-pursue-ad-hoc-cluster-orchestration.md)
- [`../README.md`](../README.md) — repo-relationship diagram

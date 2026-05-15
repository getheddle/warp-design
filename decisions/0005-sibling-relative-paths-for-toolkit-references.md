# 0005. Sibling-relative paths for cross-repo references within the toolkit-anchored agent layer

- **Date:** 2026-05-15
- **Status:** Accepted
- **Deciders:** Hooman (principal), Claude Code (sparring partner)

## Context

ADR [0003](0003-warp-and-warp-design-repo-split.md) established two
conventions for cross-repo links:

> Use full GitHub URLs in cross-repo links; relative links only within
> a single repo.

That rule was correct for user-facing documentation where a reader
might land on the rendered MkDocs site without a local clone of the
sibling repo. The full URL guarantees the link works regardless of
where the reader is.

Subsequent work introduced
[`heddle-agent-toolkit`](https://github.com/getheddle/heddle-agent-toolkit),
a shared repository of agent guidance (anchors, skills, subagents). The
toolkit is *designed* to be installed as a sibling of the consumer
repo via `install.sh`, which creates symlinks under the consumer's
`.claude/` directory back into the sibling toolkit checkout. The
sibling-clone convention is therefore a load-bearing assumption of the
toolkit's install model — not an accident of one developer's directory
layout.

Within that toolkit-anchored agent layer (`AGENTS.md`, `CLAUDE.md`,
and any `.claude/` content), references to `../heddle-agent-toolkit/`
or other sibling repos are *guaranteed* to resolve when the install
convention is followed. Forcing full GitHub URLs in those files would:

- Push readers off the local clone they already have to a GitHub web
  page that just paraphrases the same content.
- Break the offline development case (the toolkit anchors are meant to
  load into Claude's context from the local symlink, not over HTTPS).
- Create noise for AI agents whose context window already includes
  the sibling files.

## Decision

Within the **agent layer** — defined as:

- `AGENTS.md` and `CLAUDE.md` in any `getheddle/*` repository
- Any file under any repo's `.claude/` directory
- Any file in `heddle-agent-toolkit/` itself

— cross-repo references to sibling repos may use **sibling-relative
paths** (`../heddle/...`, `../heddle-agent-toolkit/...`, etc.). The
sibling-clone convention is assumed.

Outside the agent layer — user-facing documentation, READMEs aimed at
end users, MkDocs site content — the ADR 0003 rule still applies:
**use full GitHub URLs for cross-repo references**. Readers may be
browsing the rendered site without any local clone.

In short: agent-facing files assume sibling clones; user-facing files
do not.

## Consequences

**Easier**

- Local agent sessions don't bounce out to GitHub for content the
  reader already has on disk.
- The toolkit's anchor docs can be loaded by skills as local files,
  with confidence that referenced paths resolve.
- Diffs in agent-layer files stay readable (relative paths are shorter
  than `https://github.com/...` URLs).

**Harder**

- We now have two link conventions in the family. A contributor moving
  text from an agent-layer file to a user-facing doc must rewrite
  links; vice versa is the same problem in reverse.
- A new contributor who clones one repo without the sibling toolkit
  will see dead links in `AGENTS.md` and `CLAUDE.md` until they
  install the toolkit. Mitigated by the "Toolkit install" section now
  present in each `AGENTS.md`.

**Trade-offs accepted**

- We accept the rule-bifurcation cost in exchange for the
  install-model coherence. The alternative (full URLs everywhere)
  optimizes for the cold-from-GitHub reader at the cost of every
  warm-from-local-clone agent session, and agent sessions are the
  much more frequent use.

## References

- [`0003-warp-and-warp-design-repo-split.md`](0003-warp-and-warp-design-repo-split.md)
- [`../../heddle-agent-toolkit/README.md`](../../heddle-agent-toolkit/README.md)
- Audit finding G3a (heddle-agent-toolkit/audit-warp-design.md)

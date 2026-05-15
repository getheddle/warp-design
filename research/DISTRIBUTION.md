# DISTRIBUTION.md — Code signing, notarization, and distribution

**Status:** TODO (stub).

**Gates:** Release pipeline, CI signing strategy.

Findings here pin the release machinery for warp binaries so the v0
deliverable can ship as a notarized, Gatekeeper-friendly binary
without scaring off non-technical SMB users.

## Topics to cover

- **Developer ID** — Apple Developer Program enrollment, certificate
  management, automating CI access to signing identities.
- **Hardened runtime** — required entitlements, which capabilities to
  declare, runtime restrictions (no JIT, restricted dyld, etc.).
- **Notarization** — `notarytool submit`, stapling, ticket revocation
  flow, common rejection reasons.
- **Gatekeeper UX** — first-launch warnings on unsigned vs. signed vs.
  notarized binaries; quarantine attribute behavior; Sequoia-and-later
  changes around "always allow" workflows.
- **CI signing** — secrets handling (encrypted keychain vs. fastlane
  match vs. raw .p12), reproducible builds, supply-chain hygiene
  (SBOMs, dependency provenance).
- **Distribution channels** — direct download (signed pkg), Mac App
  Store (sandbox requirements, IAP irrelevant), Homebrew tap, GitHub
  Releases. Trade-offs per channel for daemon-class software.
- **Future-proofing** — sandbox profile, App Store submission
  readiness even if v0 ships off-store.

## Implications for v0 to capture

- A concrete CI signing+notarization pipeline (likely GitHub Actions
  with encrypted secrets, `xcrun notarytool submit --wait`).
- Required entitlements minimal set.
- Distribution channel for the v0 release.

## Open questions

- Mac App Store path: keep it open, or commit to off-store distribution
  and avoid sandboxing constraints from day 1?
- Is Homebrew tap a viable first channel for SMB operators, or do we
  need a one-click `.pkg` installer with a UI?
- How do we make first-launch trust friction-free without compromising
  on Developer ID hardening?

## References

- (To add as research lands.)

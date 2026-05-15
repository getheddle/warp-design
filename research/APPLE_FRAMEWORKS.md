# APPLE_FRAMEWORKS.md — Apple platform frameworks warp depends on

**Status:** TODO (stub).

**Gates:** Service registration approach, mDNS implementation, future
capacity-probe primitives.

Findings here pin which Apple frameworks warp uses, which APIs are
public vs. SPI, and how the trust/permissions story fits TCC and
hardened-runtime requirements.

## Topics to cover

- **SMAppService** — modern daemon registration; how it differs from
  hand-rolled launchd plists; the `SMAppService.daemon(plistName:)`
  registration model; uninstall semantics; TCC interactions.
- **Network.framework** — `NWConnection`, `NWListener`, `NWBrowser`
  for discovery and connectivity, including over Thunderbolt
  interfaces.
- **mDNS / Bonjour** — `NWBrowser`/`NWListener` Bonjour APIs vs.
  `dnssd` C APIs; service registration formats; the heddle existing
  mDNS primitive we can mirror.
- **EndpointSecurity** — auditing / observability of OS events; what
  capability warp might want to observe (process lifecycle of the
  supervised Python service).
- **IOKit power/thermal** — `IOPMCopyCPUPowerStatus`,
  `IOPSCopyPowerSourcesInfo`, thermal-state APIs; what is public vs.
  needs entitlements.
- **Virtualization.framework** — `VZVirtualMachine` for workload
  isolation in later phases.
- **Keychain Services** — secret storage for pairing tokens, mTLS keys.
- **TCC + Info.plist** — `NSPrivacy*` keys for camera/mic/screen/files;
  what warp needs *not* to trigger inadvertently; how to declare for
  Mac App Store submission later.

## Implications for v0 to capture

- The minimum entitlements warp needs.
- Which frameworks are required at link time vs. dlopen-on-demand
  (matters for code-signing entitlements bundle).
- How daemon registration plays with Developer ID notarization
  (see `DISTRIBUTION.md`).

## Open questions

- Does SMAppService work cleanly when the supervised binary is a
  user-installed `uv`/`pipx`-resolved Python? Is there a sane
  bootstrap order?
- Does `NWBrowser` correctly enumerate peers across Thunderbolt-IP
  links, or do we need to use lower-level mDNS APIs?
- What is the minimum-friction TCC posture for warp v0 (initially
  full-disk-access-free, asking only for what's needed)?

## References

- (To add as research lands.)

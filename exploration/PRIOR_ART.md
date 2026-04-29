# Prior Art Survey

**Status:** Exploration. Snapshot as of 2026-04-29; landscape changes
fast and this needs periodic refresh.

Surveys adjacent projects and identifies what warp is converging with vs.
competing against. The goal is to know where existing tools end so warp
can start there, and to avoid reinventing wheels that someone else has
already polished.

## Closest analog: Exo

- **What:** Peer-to-peer LLM inference across home devices.
  [github.com/exo-explore/exo](https://github.com/exo-explore/exo)
- **Scope:** Inference only. Splits a single LLM across multiple devices
  for memory pooling. Apple-Silicon-friendly.
- **Overlap:** Multi-device coordination on a home/SMB-scale fleet.
- **Differences:** Inference-distribution focus, no orchestration of
  arbitrary workloads, no scheduler, no privacy-tag routing, no cost
  arbitrage, no hardware advisor. They've solved a sub-problem warp could
  use as one tactic among many.
- **What we'd learn from them:** Memory-pooling protocol, peer discovery
  patterns, what works/doesn't in real-world home networks.

## Lightweight Kubernetes alternatives

### k3s

- **What:** Production-grade single-binary K8s by Rancher/SUSE. Small
  footprint, designed for edge/IoT/CI.
- **Overlap:** Edge orchestration positioning.
- **Differences:** Still K8s under the hood — yaml manifests, CRDs,
  kubectl. Wrong shape for non-ops-team SMB. No AI-workflow primitives,
  no privacy tags, no cost arbitrage, no hardware advisor.
- **What we'd learn:** How to package a single-binary daemon for many
  platforms. Their distribution story is excellent.

### KubeEdge

- **What:** Cloud-native edge computing for K8s. Operates at the
  cloud-edge boundary.
- **Overlap:** Edge concept.
- **Differences:** Same as k3s — K8s-shaped. Designed for industrial-IoT
  fleets with a central cloud control plane, not for SMB peer-to-peer.

### MicroK8s, k0s, etc.

- Same K8s-shaped problem at smaller scale. Useful for engineers; wrong
  for SMB.

## GPU scheduling

### NVIDIA Run:ai

- **What:** GPU scheduler for ML clusters. Acquired by NVIDIA. Enterprise.
- **Overlap:** Resource scheduling for AI workloads.
- **Differences:** Cluster-side only (no edge). Apple-blind. Expensive.
  Assumes K8s or Slurm.
- **What we'd learn:** Workload-aware scheduling primitives, fairness
  policies, gang scheduling for distributed training.

### NVIDIA DCGM + GPU Operator

- **What:** GPU monitoring + scheduling primitives for K8s.
- **Overlap:** Capacity reporting for accelerators.
- **Differences:** NVIDIA-only. K8s-specific.
- **What we'd learn:** Capacity vocabulary for accelerators. Their
  metrics taxonomy is mature.

## Local AI inference stacks

### Ollama

- **What:** Run LLMs locally with a clean CLI/REST API. Single-binary.
- **Overlap:** Local LLM execution. Has very-recent multi-machine support.
- **Differences:** Inference-only, no workflow orchestration, no cluster
  scheduling beyond the new (and limited) clustering. Excellent
  packaging + UX.
- **What we'd learn:** Single-binary distribution, model management UX,
  REST API ergonomics for local LLMs.

### LM Studio

- **What:** GUI app for running local LLMs. OpenAI-compatible REST API.
- **Overlap:** Local LLM execution + serving.
- **Differences:** Single-machine only. GUI-first (warp wants invisible).
  No clustering.
- **What we'd learn:** Apple Silicon performance tuning patterns
  (used heavily in heddle today).

### llama.cpp

- **What:** The C++ runtime that everything else builds on.
- **Overlap:** None directly — it's a building block.
- **What we'd learn:** Performance characteristics, backend abstraction
  (Metal/CUDA/Vulkan/CPU).

## Apple-specific primitives

### Apple Container Sandbox

- **What:** Apple's lightweight container runtime using
  Virtualization.framework. New as of 2025.
- **Overlap:** Per-workload isolation primitive.
- **Differences:** It's a building block, not a competitor. Warp would
  *use* it for workload isolation in later phases.
- **What we'd learn:** Native macOS isolation patterns, not
  Docker-shaped retrofits.

### NetworkExtension framework

- **What:** Apple's framework for VPNs, content filters, and DNS proxies.
- **Overlap:** Could be used for cluster-traffic routing and isolation.
- **Differences:** It's a primitive, not a product.

### EndpointSecurity framework

- **What:** Apple's framework for monitoring system events (process
  exec, file access, network).
- **Overlap:** Could be used for warp's audit + privacy enforcement.
- **Differences:** Primitive, not a product.

### SMAppService

- **What:** The modern Service Management framework, replacing
  SMJobBless and hand-managed launchd plists. Includes proper TCC
  integration.
- **Overlap:** Direct — this is *how* warp registers as a daemon on
  macOS.

## Distributed compute (academic / research)

### Ray, Dask, Spark on small clusters

- **What:** Distributed computing frameworks designed primarily for
  data-center clusters.
- **Overlap:** Cluster compute concepts.
- **Differences:** Ops-team-shaped. Resource models assume homogeneous
  fleets. Network assumptions are data-center, not Thunderbolt + Wi-Fi.

### BOINC

- **What:** Distributed compute for volunteer grids (SETI@Home, etc.).
- **Overlap:** Volunteer-style "donate idle cycles" idea.
- **Differences:** No interactive workloads, no privacy tags, no
  cluster-of-trust model. But the donation idea has aged well.

## What warp's distinctive lane is

Cross-referencing the above, no existing project covers all of:

1. **AI-workflow shape** (workers + pipelines + multi-stage orchestration,
   not just inference)
2. **Heterogeneity-native** (Apple Silicon + NVIDIA + AWS Inferentia
   first-class, not "we have a GPU plugin")
3. **Privacy-aware routing** (tags constrain where workloads run)
4. **Cost arbitrage** (local sunk-cost vs. cloud variable cost)
5. **Hardware advisor** (longitudinal pattern analysis → ROI advice)
6. **SMB-friendly UX** (AirDrop-shaped, not kubectl-shaped)
7. **Apple-native** (SMAppService, EndpointSecurity, Network.framework,
   Virtualization.framework — first-class)
8. **Cross-platform** (Linux + Windows agents follow, sharing protocol)

Most projects do 1-2 of these well; warp aims for all 8 over phases 0-6.

## Open research items

To be filled in by upcoming research:

- [ ] Survey Mac App Store / macOS-native cluster apps for adjacent UX
      patterns
- [ ] Survey Apple Silicon ML benchmarks (MLPerf etc.) for capacity
      vocabulary
- [ ] Talk to / look at Anthropic's published per-token energy data
- [ ] Look at Apple's published per-chip ANE TOPS specs
- [ ] Find and read whatever Run:ai's customers have published about
      pain points (likely useful negative data)

## Reference table — when to revisit

| Project | Why we'd recheck | Trigger |
|---|---|---|
| Exo | If they expand beyond inference | Their changelog |
| k3s / KubeEdge | If they ship something dramatically simpler | Quarterly |
| Ollama | They're moving fastest in the local space | Monthly |
| Apple Container Sandbox | We'll likely *use* it | Each macOS release |
| SMAppService | Same | Each macOS release |
| Run:ai (NVIDIA) | If they expand to non-K8s | Annually |

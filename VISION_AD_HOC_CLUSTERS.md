# The Future of AI Compute: Ad-Hoc Personal & SMB Clusters

**Status:** Vision (committed direction, implementation underway via warp).
**Date:** 2026-04-29.
**Audience:** Anyone considering heddle's direction, contributing to warp,
or evaluating whether this approach fits their environment.

## Thesis

**Apple Silicon and hyperscaler chips (AWS Graviton/Trainium/Inferentia, NVIDIA
Hopper/Blackwell, etc.) are not competing — they optimize for orthogonal axes,
and the AI future needs both layers working together.**

- **Apple Silicon** optimizes for *best computer per user*: unified on-package
  memory, heterogeneous cores (Performance + Efficiency CPU + GPU + Neural
  Engine + Media + Secure Enclave), tight HW/SW integration, deployed across
  tens of millions of personal devices. Built for **mixed, bursty, personal**
  workloads with privacy as a default property of the platform.
- **Hyperscaler chips** optimize for *cheapest compute per workload*:
  disaggregated infrastructure, chips purpose-built per task (general
  compute / training / inference), deployed across millions of servers. Built
  for **continuous, specialized, global** workloads where data already lives
  in the cloud or where the scale of the task exceeds what any single
  endpoint could deliver.

The personal-vs-global axis is more durable than the edge-vs-cloud axis.
Edge/cloud is a deployment topology that could collapse if networks get fast
enough; personal/global is about *who the compute serves*, which is permanent.
Apple is on the personal axis; hyperscalers are on the global axis. They
literally can't substitute for each other.

## What's missing

Today's AI tooling treats either axis in isolation:

- **Local-only stacks** (Ollama, LM Studio, llama.cpp) are excellent on a
  single machine but have no story for: pooling capacity across the four Macs
  in your house or office; shedding load to cloud when local compute is
  insufficient; routing private workloads to local while letting non-private
  workloads use the cheapest substrate; or telling you when buying a Mac
  Studio would pay back vs. continuing to rent cloud time.
- **Cloud-only stacks** (Bedrock, Vertex, OpenRouter, etc.) are excellent at
  scale but force every workload through a metered third party — no
  privacy story, no cost arbitrage with local capacity, no concept of "this
  question can be answered by your laptop in 200ms."
- **Heavyweight orchestrators** (Kubernetes + KubeFlow + Run:ai) provide
  scheduling and resource management but assume a homogeneous fleet, an ops
  team, and a uniform OS. They're catastrophically wrong-shaped for a small
  business with four Macs and an analyst who wants to query their own data
  privately.

The gap is **a control plane that treats personal/SMB clusters as
first-class**: heterogeneous, privacy-aware, cost-conscious, ad-hoc, and
operable by someone whose job is not "Kubernetes administrator."

## What warp becomes

Warp is the daemon agent + control plane that:

- **Discovers peers** via mDNS (already a heddle primitive) and Thunderbolt
  topology (Apple's 40-80 Gbps wired link makes machine-to-machine
  coordination essentially free in latency terms within ~3m).
- **Reports capacity** in dimensions current orchestrators ignore: free
  unified memory, available Neural Engine seconds, thermal headroom (is
  the M-series throttling?), power state (battery vs. plugged in for
  laptops, where running heavy work on battery is rude to the user),
  network bandwidth to each peer (Thunderbolt vs. Wi-Fi).
- **Schedules workloads** by matching requirements + privacy tags + deadlines
  + budget against the live capacity map. A workload tagged "private" never
  leaves the local cluster. A workload with "deadline: 30s" routes to
  whichever node has free Neural Engine. A workload tagged "training,
  large" might choose cloud burst if local capacity would block the user's
  Mac Studio for 8 hours.
- **Migrates work** when capacity shifts — a Mac Studio joins, a MacBook
  unplugs, the user starts a video render. Stateless work moves trivially
  via heddle's existing NATS queue groups; stateful work (long inference,
  training jobs) gets graceful drain and eventually checkpoint+resume.
- **Bursts to cloud** on policy: when local backlog exceeds a threshold AND
  cloud cost stays under budget AND the workload's privacy tag permits.
- **Recommends hardware** — observes actual workload patterns over weeks
  and projects ROI. "Your nightly inference backlog is costing $400/mo on
  AWS Inferentia; a Mac Studio M5 Ultra pays back in 9 months at your rate.
  Or: a Jetson AGX Orin would handle your throughput for $2K up-front and
  ~30W steady-state, with these caveats..."
- **Stays out of the user's way** — feels like AirDrop or HomeKit, not
  like `kubectl`. The whole point is that a small business with three Macs
  and an iMac can install warp in five minutes and have an ad-hoc compute
  pool without learning anything about clustering.

## Why macOS is the right starting environment

Not "macOS forever." Warp will eventually have a Linux agent (likely Rust)
for Jetsons, DGX boxes, and traditional servers, plus a Windows agent. But
macOS is the right *starting* environment because:

| Property | Why it matters for v0 |
|---|---|
| **Apple Silicon mixed-load performance** | A single M-series chip handles inference + agents + the user's actual work concurrently, which is the workload shape warp targets |
| **Unified memory** | Capacity model can assume "total memory pool" without per-device fragmentation gymnastics |
| **Neural Engine + Media engines + GPU types** | Heterogeneous capacity reporting is forced from day 1, which is right for the long-term design |
| **Thunderbolt 4/5 ubiquity** | 40-80 Gbps wired M2M links between common consumer machines — the topology assumption "fast wired link between cluster members" is realistic |
| **Apple's Virtualization.framework** | Low-overhead VMs for workload isolation are a built-in primitive, not a docker-shaped retrofit |
| **mDNS / Bonjour built in** | Discovery is free; we don't need to ship a discovery protocol |
| **Privacy + security primitives** | TCC, Keychain Services, EndpointSecurity, hardened runtime, code signing, notarization — we get a structured trust model from the OS instead of bolting one on |
| **Service Management framework (SMAppService)** | Modern, principled daemon registration with proper TCC interaction, replacing hand-rolled launchd plists |
| **Single-vendor coherent story** | macOS + Apple Silicon + Apple frameworks let v0 target a small surface and prove the architecture; cross-platform comes after |
| **Operator's familiarity** | The initial implementer (Hooman) has long Swift experience |

## Why heddle is positioned to own this

Heddle already has the substrate:

| Heddle primitive | Maps to in warp |
|---|---|
| NATS bus + queue groups | Cluster control plane + load balancing |
| BaseActor / LLMWorker / ProcessorWorker | Unit of compute that ships across nodes |
| PipelineOrchestrator with parallel-stage detection | Workload graph that already understands "this stage can run anywhere" |
| OllamaBackend / LMStudioBackend / OpenAICompatibleBackend / AnthropicBackend | Multi-runtime abstraction is partway there |
| mDNS service advertisement | Node discovery primitive |
| `HEDDLE_LOCAL_BACKEND` selector | Single-node version of "pick where to run this" |
| Workshop's eval + impact analysis | Half of what a hardware advisor needs |
| `baft.sessions` markers + scheduler `expand_from` | Multi-tenant awareness |

The net-new pieces — the warp-specific work — are the **node agent**, the
**capacity model**, **topology awareness**, the **scheduler-with-policies**,
the **budget controller**, the **migration engine**, the **hardware advisor**,
**trust + admission control**, and the **end-user UX**.

Of those, **the node agent** is the foundational primitive that everything
else builds on. That's where v0 begins. See
[`daemon-v0/SCOPE.md`](daemon-v0/SCOPE.md).

## What this is not

- **Not a rewrite of Kubernetes for laptops.** K8s is wrong-shaped for SMB:
  too much yak-shaving for the operator, too little awareness of personal
  device concerns (privacy, battery, thermal, user attention). Warp borrows
  K8s *concepts* (node/scheduler/resource model) but rejects K8s *ergonomics*.
- **Not a hyperscaler replacement.** Cloud is the right answer for training
  large models, processing massive datasets, and cross-organization
  collaboration. Warp's job is to use cloud only when local can't.
- **Not Apple-only forever.** macOS is the starting environment. Linux and
  Windows agents follow. The protocol between agents is platform-neutral.
- **Not a single-tenant tool.** Multiple users on the same cluster is in
  scope. Per-user RBAC, per-workload privacy tags, per-tenant budgets.
- **Not a research project.** This is a production direction with a phased
  delivery plan. The exploration is in service of shipping.

## Adjacent prior art

Worth knowing what we're converging with or competing against. See
[`exploration/PRIOR_ART.md`](exploration/PRIOR_ART.md) for the survey.
Short version:

- **Exo** — peer-to-peer LLM inference across home devices. Closest analog;
  much narrower scope (inference only).
- **k3s / KubeEdge** — lightweight Kubernetes for edge. Still K8s-shaped.
- **NVIDIA Run:ai** — GPU scheduling, cluster-side, expensive, ignores Apple.
- **Apple Container Sandbox** — Apple's own lightweight container runtime
  using Virtualization.framework. Useful primitive for warp; not a competitor.
- **Ollama clustering** — narrow inference distribution; not workflow-aware.

Heddle's distinctive lane is **AI-workflow shape + heterogeneity + privacy +
cost arbitrage + hardware advisory** in one coherent product. None of the
above does all five.

## Phased delivery

| Phase | Deliverable | Ambition |
|---|---|---|
| **0 — daemon core** | Swift daemon (warp v0) that supervises the existing Python heddle service, registers via SMAppService, announces via mDNS, lays foundation modules (swift-service-lifecycle, swift-log, swift-metrics, swift-distributed-tracing) without committing to scheduling | Replaces launchd's role; sets the ground for everything else; ships on one machine |
| **1 — capacity reporter** | Agent reports its live capacity over NATS; Workshop renders a fleet view across discovered peers | First multi-node visibility |
| **2 — capacity-aware routing** | Pipeline stages route to the agent with best fit (memory, accelerator availability, latency to peer); stateless graceful drain | First real ad-hoc clustering |
| **3 — budget + cost arbitrage** | Cloud-burst policy engine; per-tenant + per-user budgets; cost dashboards | Local + cloud hybrid in production shape |
| **4 — hardware advisor** | Longitudinal workload analysis; ROI projections; spec recommendations | The unique product feature |
| **5 — Linux + Windows agents** | Rust agent for non-macOS platforms; cross-platform protocol formalized | True heterogeneous fleet |
| **6 — UX polish** | First-class macOS app for cluster setup, status, and policy editing; SMB onboarding flow | Usable by people who aren't Hooman |

Each phase is independently useful — none depends on the next phase shipping.

## Open questions

Captured in `decisions/` as they get resolved. Currently open:

- License (likely Apache-2.0; ADR pending)
- IPC protocol between Swift daemon and Python service (UDS + JSON-NDJSON
  vs. msgpack vs. cap'n proto)
- How much of v0's foundation should be Swift-Service-Lifecycle-shaped vs.
  hand-rolled (depends on research findings)
- Trust/admission model for cluster joining (mTLS? shared secret? PIN
  pairing?)
- Capacity vocabulary — which dimensions are first-class in v1 (memory,
  CPU, GPU TFLOPS, Neural Engine TOPS, thermal, power, bandwidth-to-peer)
- Workload-class taxonomy (inference / training / agent / batch / ...)

See also `EVOLUTION_LOG.md` for what's been decided + the reasoning trail.

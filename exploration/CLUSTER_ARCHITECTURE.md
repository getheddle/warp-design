# Cluster Architecture — Initial Sketch

**Status:** Exploration. Not committed. Captures the architectural shape
discussed on 2026-04-29 before serious research; expect this to evolve
substantially as research streams complete.

## Shape

```text
                                ┌──────────────────────────────────────┐
                                │           Operator UX                │
                                │  (Workshop web, native macOS app,    │
                                │   CLI — eventual)                    │
                                └──────────────┬───────────────────────┘
                                               │
                                               ▼
                       ┌────────────────────────────────────────────────┐
                       │           Cluster Control Plane                │
                       │   (NATS bus + scheduler + budget controller)   │
                       └─────┬──────────────────────────────────────┬───┘
                             │                                      │
                ┌────────────┴────────────┐         ┌───────────────┴──────────────┐
                ▼                         ▼         ▼                              ▼
         ┌──────────────┐         ┌──────────────┐ ┌──────────────┐         ┌────────────┐
         │ warp agent   │  ...    │ warp agent   │ │ warp agent   │   ...   │ warp agent │
         │ (Mac Studio) │         │ (MacBook Pro)│ │ (Mac mini)   │         │ (Jetson —  │
         │              │         │              │ │              │         │ phase 5)   │
         └──────┬───────┘         └──────┬───────┘ └──────┬───────┘         └─────┬──────┘
                │                        │                │                       │
                ▼                        ▼                ▼                       ▼
         ┌──────────────┐         ┌──────────────┐ ┌──────────────┐         ┌────────────┐
         │ Heddle       │         │ Heddle       │ │ Heddle       │         │ Heddle     │
         │ workers      │         │ workers      │ │ workers      │         │ workers    │
         │ (Python +    │         │ (subset —    │ │ (subset)     │         │ (Linux/    │
         │ LM Studio)   │         │ battery!)    │ │              │         │ TensorRT)  │
         └──────────────┘         └──────────────┘ └──────────────┘         └────────────┘

                                                    ▲
                                                    │ (cloud burst)
                                                    │
                       ┌────────────────────────────┴───────────────────────────────┐
                       │     Cloud substrates (AWS, GCP, Azure, etc.) — phase 3     │
                       │      Trainium / Inferentia / Bedrock / etc.                │
                       └─────────────────────────────────────────────────────────────┘
```

## Components

### Per-node: warp agent

The cluster-member daemon. One per machine. Runs as the user (or a
service account on Linux servers).

**Responsibilities:**

| Responsibility | v0 | v1 (capacity) | v2 (routing) | v3 (budget) | v4 (advisor) |
|---|---|---|---|---|---|
| Process supervision (spawn workers, monitor, restart) | ✓ | ✓ | ✓ | ✓ | ✓ |
| OS-native service registration (SMAppService on macOS) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Structured logging, health endpoint, mDNS announce | ✓ | ✓ | ✓ | ✓ | ✓ |
| Foundation modules wired (lifecycle, log, metrics, tracing) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Capacity probes (memory, CPU, GPU, NE, thermal, power) |  | ✓ | ✓ | ✓ | ✓ |
| Capacity reports published over NATS |  | ✓ | ✓ | ✓ | ✓ |
| Local workload execution under scheduler direction |  |  | ✓ | ✓ | ✓ |
| Stateless graceful drain on shutdown / pressure |  |  | ✓ | ✓ | ✓ |
| Cost/electricity sampling for budget controller |  |  |  | ✓ | ✓ |
| Workload pattern reporting for advisor |  |  |  |  | ✓ |

### Cluster control plane: scheduler + budget controller

Lives in the heddle process (Python) — at least initially. The scheduler
matches workload requirements to capacity reports; the budget controller
gates cloud-burst decisions on cash + local-compute thresholds.

**Open question:** does the scheduler stay Python-side forever, or does it
eventually move into a coordinator role inside warp itself? Answering this
is downstream of the protocol design. For now: scheduler stays in heddle,
agents are dumb-but-capable executors.

### Workload classes

Initial taxonomy (will refine):

- **Inference (single-shot)** — short, latency-sensitive, fits on one node
- **Inference (batch)** — throughput-sensitive, parallelizable, can shard
- **Training (small)** — fits on a few GPU-class nodes for hours
- **Training (large)** — only viable on cloud burst
- **Agent (long-running)** — interactive, stateful, conversational
- **Pipeline (multi-stage)** — heddle's pipeline orchestrator semantics
- **Background batch** — non-urgent, opportunistic, runs on idle capacity

Privacy tags orthogonal to class:

- **Public** — can run anywhere including cloud
- **Sensitive** — local cluster only
- **Personal-data** — local only AND must satisfy per-user isolation

### Capacity vocabulary (initial)

Each agent reports a snapshot like:

```yaml
agent_id: warp-mac-studio-01
hostname: hooman-studio.local
os: macos-15.0
arch: arm64
chip: apple-m5-ultra
memory:
  total_gb: 128
  free_gb: 92
  unified: true
cpu:
  performance_cores: 16
  efficiency_cores: 8
  utilization: 0.12
gpu:
  type: apple-m5-ultra-gpu
  cores: 76
  memory_gb: 128  # unified
  utilization: 0.05
neural_engine:
  cores: 32
  estimated_tops: 80
  available: true
thermal:
  state: nominal  # nominal | fair | serious | critical
  throttling: false
power:
  source: ac  # ac | battery
  battery_level: null
network:
  peers:
    - id: warp-mac-mini-02
      bandwidth_gbps: 80   # thunderbolt 5
      rtt_ms: 0.12
    - id: warp-imac-03
      bandwidth_gbps: 1    # wifi
      rtt_ms: 3.4
labels:
  - role: compute
  - location: hooman-office
  - admin-grants:
      - cloud-burst-enabled
```

### Topology awareness

We distinguish:

- **Thunderbolt ring/star** — 40-80 Gbps, sub-ms RTT, within physical
  reach. The scheduler treats these as "essentially free to coordinate."
- **Local network (Ethernet / Wi-Fi 6e+)** — 1-10 Gbps, single-digit ms RTT.
  Fine for normal cluster work, expensive for tightly-coupled inference.
- **Internet** — variable. Cloud burst territory.

Agents probe peer-link bandwidth periodically and publish in their
capacity report. Scheduler uses topology in its placement decisions
(e.g., "this multi-stage pipeline fits cleanly on the Thunderbolt ring;
that one needs to stay on a single Mac").

### Trust + admission

How does a new machine join a cluster? Open question, options:

- **PIN pairing** — operator runs `warp join <cluster-name>` on the new
  machine, types a 6-digit PIN displayed on an existing cluster member.
  Simple, AirDrop-like.
- **mTLS with CA bootstrap** — first cluster member generates a CA, signs
  its own cert, ships the CA fingerprint via QR or PIN to new joiners.
- **Shared secret in a config file** — works for ops-managed deployments,
  bad UX for SMB.

Initial lean: PIN pairing for SMB, mTLS-with-CA for advanced/multi-tenant.
Both produce mTLS at the network layer; the difference is bootstrap UX.

### State migration

Most workloads are stateless from the cluster's POV (heddle workers reset
after each task). The ones that aren't:

- **LLM KV cache** — present during a long inference. Migrating means
  serializing + transferring the cache. Likely deferred until phase 4+.
- **Telethon session** — bound to one machine via its session file.
  Migration means transferring the file (plus possibly re-auth from the
  Telegram side). Probably never auto-migrated; stays on the original
  agent.
- **Embedding indices, vector stores** — large; migrating is expensive.
  Probably stay put; replication for HA is a separate later concern.

For v2 routing, accept that **stateless work moves trivially, stateful
work runs in place or stops + restarts**. Real migration is phase 4+.

## How heddle's existing pieces map

| Heddle today | Role in warp's cluster |
|---|---|
| `BaseActor._process_one` | The unit of work; warp ships these between agents |
| `LLMWorker.execute_with_tools()` | LLM-backed worker; capacity-aware placement |
| `PipelineOrchestrator` parallel-stage detection | Already understands "this stage can run anywhere" |
| NATS queue groups | Load-balancing primitive — agents subscribe to `heddle.tasks.{worker}.{tier}` and the bus distributes |
| `OllamaBackend` / `LMStudioBackend` / `OpenAICompatibleBackend` / `AnthropicBackend` | Multi-runtime — agent picks the right backend for its host |
| mDNS service advertisement | Discovery primitive — already mostly there |
| `~/.heddle/config.yaml` resolution chain | Config layering survives the cluster expansion |
| Workshop UI | Eventual fleet view — already half the UI exists |
| Distributed tracing (W3C traceparent through NATS) | Cross-agent traces work already, just need to be exercised |

The net-new pieces are the agent itself, the capacity model, the
topology-aware scheduler additions, and the budget controller. Everything
else is "use what's already there" or "extend it modestly."

## Hard problems (collected, not solved)

1. **Heterogeneous capacity is non-trivial.** Apple Silicon's unified
   memory + dynamic CPU/GPU split + Neural Engine make "available compute"
   not a number — it's a vector indexed by workload class. Worth doing
   right; it's the moat.
2. **Workload portability across runtimes.** Same logical workload, three
   different runtimes (LM Studio MLX, TensorRT-LLM, NeuronSDK). Need an
   abstraction that doesn't reduce everything to lowest-common-denominator.
3. **Latency-aware routing.** Topology probing must not flood the network.
   How often, what protocol, and how to invalidate quickly when a
   Thunderbolt cable is unplugged.
4. **Laptop courtesy.** Running heavy work on a MacBook Pro on battery is
   rude. Policy needs to be richer than "use idle cycles."
5. **The "K8s for SMB" UX problem.** Big design challenge orthogonal to
   the systems work.
6. **Cloud cost modeling is hard.** Comparing local sunk-cost compute to
   cloud variable cost requires a coherent model. Anthropic and Apple have
   each published per-token energy data; we'd standardize ours.
7. **Hardware advisor cold-start.** Need months of data to give good
   advice. Bootstrap with rules-of-thumb tables.

## What this sketch does not yet specify

- Exact NATS subject hierarchy for warp messages
- Capacity report wire format (JSON vs. msgpack vs. cap'n proto)
- Admission protocol details
- Telemetry sampling rates
- Budget controller policy DSL
- Migration trigger thresholds
- Per-platform agent feature parity matrix

These get pinned down as the research streams complete and v0 ships.

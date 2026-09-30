# Curriculum Index

Module IDs, titles, and paths match the directories in this folder. Effort is each module's `estimated_effort.total`; outline modules show a planning placeholder until authored.

Status: **authored** modules follow the [instructional schema](INSTRUCTIONAL_SCHEMA.md) with a research registry and knowledge model; **outline** modules are placeholders that are not yet curriculum content.

## Phase 1 — Scientific and Model Foundations

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [00](00-scientific-ai-engineering/) | Scientific AI Engineering | authored (7 lessons, 4 labs) | 23h | Estimands, experimental units, randomization and blocking, warmup/state control, tails and outliers, confidence intervals, coordinated omission, reproducibility, O/D/H evidence discipline |
| [01](01-foundation-model-internals/) | Foundation Model Internals | authored (7 lessons, 4 labs) | 26h | Decoder tensor contracts, MHA/MQA/GQA, RoPE, RMSNorm, SwiGLU, MoE and latent attention scope, parameter and FLOP accounting, numerical failures |
| [02](02-inference-gpu-fundamentals/) | Inference GPU Fundamentals | authored (7 lessons, 5 labs) | 29h | Execution and memory hierarchy, asynchronous timing, coalescing, qualified Roofline, occupancy, shape-dependent regimes, profiler scope |

## Phase 2 — Inference Systems

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [03](03-kv-cache-engineering/) | KV Cache Engineering | authored (10 lessons, 5 labs) | 25h | KV payload accounting, fragmentation, paged allocation, block tables, sharing/CoW, prefix identity, cache lifecycle, KV quantization and tiering |
| [04](04-serving-scheduling-capacity/) | Serving, Scheduling, & Capacity | authored (7 lessons, 4 labs) | 25.5h | Request lifecycle, continuous batching, chunked prefill, scheduling objectives, admission and backpressure, queueing, TTFT/ITL/TPOT/goodput, capacity planning |
| [05](05-inference-optimization/) | Inference Optimization | authored (7 lessons, 4 labs) | 28h | IO-aware attention, fusion/Triton, quantization contracts (FP8/INT8/INT4, SmoothQuant/GPTQ/AWQ), exact speculative decoding, Medusa/EAGLE |
| [06](06-reasoning-test-time-compute/) | Reasoning & Test-Time Compute | authored (6 lessons, 4 labs) | 31h | Chain-of-thought scope, self-consistency/best-of-N, search, outcome/process verification, pass@k assumptions, compute ledgers, budgets and stopping |

## Phase 3 — Behavior and Data

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [07](07-model-behavior-uncertainty/) | Model Behavior & Uncertainty | authored (6 lessons, 4 labs) | 32h | Behavior contracts, factuality, calibration and proper scores, selective prediction, semantic uncertainty, conformal scope, behavior regression |
| [08](08-ai-data-engineering/) | AI Data Engineering | authored (6 lessons, 4 labs) | 31h | Data contracts, lineage, validation/quarantine, distribution shift, leakage, deduplication, contamination, synthetic data, active learning |

## Phase 4 — Retrieval and Context

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [09](09-retrieval-engineering/) | Retrieval Engineering | authored (6 lessons, 4 labs) | 30h | BM25, dense retrieval, exact vs ANN, HNSW and filtered search, hybrid/RRF, reranking, late interaction, IR metrics |
| [10](10-advanced-rag/) | Advanced RAG | authored (6 lessons, 4 labs) | 31h | RAG request contract, update/delete convergence, visibility lag, query transformation, context selection, conflict handling, citations, RAG vs long context |
| [11](11-context-memory-engineering/) | Context & Memory Engineering | authored (6 lessons, 4 labs) | 31h | Context accounting, effective context, typed memory, write/read/update/delete policy, forgetting, summaries and compression, stage-attributed memory evaluation |

## Phase 5 — Agents and Harnesses

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [12](12-agent-loop-engineering/) | Agent Loop Engineering | authored (6 lessons, 4 labs) | 24h | Bounded controllers, tool and observation contracts, budgets, terminal predicates, retries and replanning, progress detection, authority policy |
| [13](13-harness-engineering/) | Harness Engineering | authored (6 lessons, 4 labs) | 24h | Canonical vs effective requests, capability negotiation, JSON Schema and constrained decoding, validation ladder, streaming assembly, retries, replay |
| [14](14-durable-agent-runtime/) | Durable Agent Runtime | authored (6 lessons, 4 labs) | 24h | Durable history and replay, activity boundaries, idempotency, outbox/inbox, sagas, leases and fencing, durable timers, version-safe replay |

## Phase 6 — Evaluation, Falsification, Evolution, Security

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [15](15-evaluation-engineering/) | Evaluation Engineering | authored (6 lessons, 4 labs) | 24h | Evaluation contracts and units, dataset lineage and slices, grader validity, paired inference, release gates, offline-online validity |
| [16](16-falsification-engineering/) | Falsification Engineering | authored (6 lessons, 4 labs) | 24h | Property and stateful testing, shrinking, metamorphic and differential testing, mutation adequacy, zero-failure bounds, fault injection |
| [17](17-harness-evolution/) | Harness Evolution | authored (6 lessons, 4 labs) | 24h | Harness manifests, change classification, paired migration evaluation, prompt optimization as search, selection bias, shadow/canary/ramp, rollback |
| [18](18-ai-security/) | AI Security | authored (6 lessons, 4 labs) | 25h | Source–sink threat models, prompt injection and retrieval poisoning, adaptive attacks, authority and control/data separation, output handling, security evaluation |

## Phase 7 — Adaptation and Scale

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [19](19-model-adaptation/) | Model Adaptation | authored (6 lessons, 4 labs) | 24.75h | Adapt vs retrieve/prompt/route, training memory, LoRA/QLoRA mechanics, SFT data and poisoning, RLHF/DPO/GRPO, forgetting, release gates |
| [20](20-distributed-inference/) | Distributed Inference | outline | ≈25h (planned) | TP/PP/DP/CP/EP scope for inference, collectives and interconnect cost, placement, P/D disaggregation and KV transfer, failure domains |
| [21](21-multimodal-ai-systems/) | Multimodal AI Systems | outline | ≈25h (planned) | Modality preprocessing and token accounting, encoder/projector paths, multimodal batching and caching, cross-stream timing |
| [22](22-ai-economics/) | AI Economics | outline | ≈25h (planned) | Cost per offered/successful task, fixed vs marginal cost, utilization, routing and cascades, caching trade-offs, pricing snapshots and sensitivity |

## Phase 8 — Integration

| Module | Title | Status | Effort | Key Topics |
|---|---|---|---:|---|
| [23](23-observability-reliability/) | Observability & Reliability | outline | ≈25h (planned) | Trace/metric/event boundaries, offered-outcome SLIs, error budgets and burn rates, delayed quality labels, incident discrimination, rollback |
| [24](24-model-system-codesign/) | Model-System Co-design | outline | ≈25h (planned) | Architecture → memory/compute/communication → scheduler → quality/cost coupling; dense vs MoE and attention trade-offs under workload |

## Capstone and Graduation

| Item | Description |
|---|---|
| [Capstone](capstone/) | AI Runtime Platform — progressive integration of module artifacts |
| [Graduation](graduation/) | Architecture review under ambiguous requirements and constraints |

## Effort Summary

- Authored modules 00–19: **536.25h** of declared module effort.
- Outline modules 20–24: **≈125h** planned (25h each until authored).
- Module total: **≈661.25h**, before capstone and graduation. See [ROADMAP.md](ROADMAP.md) for the week plan.

## Cross-Cutting Concerns

These topics are not confined to a single module:

- **Evidence discipline** — O/D/H separation and measurement design from Module 00 apply in every lab.
- **Observability** — timing and profiling from Module 02, request metrics from Module 04, and the dedicated Module 23.
- **Security** — authority policy in Module 12, validation in Module 13, the dedicated Module 18, and training-time poisoning in Module 19.
- **Economics** — cost terms appear from Module 04 onward; the dedicated Module 22.
- **Failure analysis** — every module includes break/falsify labs and an incident scenario.

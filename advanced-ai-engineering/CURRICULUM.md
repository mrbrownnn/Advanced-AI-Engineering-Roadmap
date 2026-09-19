# Curriculum Index

## Module Map

### Phase 1 — Engineering Foundations (Week 1–3)

| Module | Title | Key Topics |
|--------|-------|------------|
| [00](00-foundations/) | Foundations | Measurement, reproducibility, hypothesis formation, experimental design, ablation, distributions, confidence intervals, evaluation strategies |
| [01](01-llm-internals/) | LLM Internals | Decoder architecture, RMSNorm, SwiGLU, RoPE, MHA/MQA/GQA/MLA, MoE, parameter and FLOP accounting |
| [02](02-inference-fundamentals/) | Inference Fundamentals | GPU execution model, memory hierarchy, HBM, arithmetic intensity, Roofline model, profiling, TTFT/TPOT/throughput/goodput |

### Phase 2 — Inference Systems (Week 4–9)

| Module | Title | Key Topics |
|--------|-------|------------|
| [03](03-kv-cache-engineering/) | KV Cache Engineering | KV lifecycle, PagedAttention, block tables, CoW, preemption, prefix/radix caching, eviction, KV quantization/offloading |
| [04](04-serving-scheduling-capacity/) | Serving, Scheduling, Capacity | Static/dynamic/continuous batching, chunked prefill, Little's Law, saturation knee, backpressure, load shedding, SLO-aware scheduling |
| [05](05-inference-optimization/) | Inference Optimization | FlashAttention, kernel fusion, FP8/INT8/INT4 quantization, AWQ/GPTQ, speculative decoding, Medusa/EAGLE, Triton fundamentals |

### Phase 3 — Behavior and Data (Week 10–11)

| Module | Title | Key Topics |
|--------|-------|------------|
| [06](06-model-behavior-uncertainty/) | Model Behavior & Uncertainty | Hallucination taxonomy, calibration, overconfidence, selective prediction, abstention, escalation, distribution shift |
| [07](07-ai-data-engineering/) | AI Data Engineering | Provenance, lineage, versioning, data contracts, deduplication, leakage, contamination, drift, synthetic data, active learning |

### Phase 4 — Retrieval and Context (Week 12–14)

| Module | Title | Key Topics |
|--------|-------|------------|
| [08](08-retrieval-engineering/) | Retrieval Engineering | BM25, HNSW, IVF, PQ, hybrid retrieval, RRF, reranking, cross encoders, late interaction, Recall@K/MRR/NDCG |
| [09](09-advanced-rag/) | Advanced RAG | Incremental indexing, freshness, stale embeddings, contradiction handling, provenance, attribution, RAG vs long context |
| [10](10-context-memory-engineering/) | Context & Memory Engineering | Context budgeting, compression, working/episodic/semantic memory, long-context degradation, position effects, cache reuse |

### Phase 5 — Agents and Evaluation (Week 15–18)

| Module | Title | Key Topics |
|--------|-------|------------|
| [11](11-durable-agent-runtime/) | Durable Agent Runtime | Execution graphs, state machines, checkpoint/resume, idempotency, fan-out/join, transactional outbox, HITL, side-effect boundaries |
| [12](12-evaluation-engineering/) | Evaluation Engineering | Deterministic/semantic/pairwise/trajectory evaluation, judge calibration, inter-rater agreement, dataset versioning, leakage |
| [13](13-falsification-engineering/) | Falsification Engineering | Property-based testing, metamorphic testing, fault injection, adversarial evaluation, grader gaming, Goodhart's Law |
| [14](14-ai-security/) | AI Security | Threat modeling, trust boundaries, prompt injection, retrieval poisoning, tool abuse, confused deputy, sandboxing, audit trails |

### Phase 6 — Adaptation and Scale (Week 19–22)

| Module | Title | Key Topics |
|--------|-------|------------|
| [15](15-model-adaptation/) | Model Adaptation | SFT, LoRA/QLoRA, DPO, RLHF concepts, failure mining, synthetic generation, data flywheel, adapt vs prompt/retrieve/route |
| [16](16-distributed-inference/) | Distributed Inference | TP, PP, DP, CP/SP, EP, all-reduce/gather/scatter, NVLink, RDMA, MoE placement, P/D disaggregation |
| [17](17-ai-economics/) | AI Economics | Cost/token, cost/task, GPU utilization economics, model routing, cascades, semantic caching, Pareto frontier, build vs API |

### Phase 7 — Integration (Week 23–26)

| Module | Title | Key Topics |
|--------|-------|------------|
| [18](18-observability-reliability/) | Observability & Reliability | Distributed tracing, scheduler/retrieval/agent telemetry, SLO, alerting, cost attribution, incident response, runbooks |
| [19](19-model-system-codesign/) | Model-System Co-design | MHA→GQA→MLA→KV→HBM→scheduler→economics; Dense→MoE→placement→communication→topology→economics |
| — | [Capstone](capstone/) | AI Runtime Platform — progressive integration of all module artifacts |
| — | [Graduation](graduation/) | Architecture review under ambiguous requirements and constraints |

## Cross-Cutting Concerns

These topics are **not** confined to a single module:

- **Observability** — introduced Module 02, deepened every module thereafter
- **Security** — surfaces in retrieval (08), agents (11), evaluation (12) before dedicated Module 14
- **Economics** — cost awareness from Module 02 onward
- **Benchmarking** — methodology from Module 00 applied everywhere
- **Failure analysis** — every module includes "break it" exercises

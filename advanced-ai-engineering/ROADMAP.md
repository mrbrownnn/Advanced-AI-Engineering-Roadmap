# Roadmap — 26 Weeks

Capstone development runs continuously from Week 6 onward. Each module produces artifacts that feed into the capstone.

| Week | Module | Focus |
|------|--------|-------|
| 1 | [00 — Foundations](00-foundations/) | Scientific engineering methodology, measurement, experimental design, placement assessment |
| 2 | [01 — LLM Internals](01-llm-internals/) | Decoder architecture, attention variants, parameter and FLOP accounting |
| 3 | [02 — Inference Fundamentals](02-inference-fundamentals/) | GPU execution model, memory hierarchy, Roofline, profiling, latency metrics |
| 4–5 | [03 — KV Cache Engineering](03-kv-cache-engineering/) | KV lifecycle, PagedAttention, fragmentation, eviction, prefix/radix caching |
| 6–7 | [04 — Serving, Scheduling, Capacity](04-serving-scheduling-capacity/) | Batching strategies, continuous batching, queueing, SLO-aware scheduling, capacity planning |
| 8–9 | [05 — Inference Optimization](05-inference-optimization/) | FlashAttention, quantization, speculative decoding, kernel optimization (bottleneck-driven) |
| 10 | [06 — Model Behavior & Uncertainty](06-model-behavior-uncertainty/) | Hallucination taxonomy, calibration, selective prediction, behavior regression |
| 11 | [07 — AI Data Engineering](07-ai-data-engineering/) | Provenance, validation, drift, synthetic data, active learning |
| 12 | [08 — Retrieval Engineering](08-retrieval-engineering/) | BM25, HNSW, hybrid retrieval, reranking, latency-quality trade-offs |
| 13 | [09 — Advanced RAG](09-advanced-rag/) | Incremental indexing, freshness, contradiction handling, RAG vs long context |
| 14 | [10 — Context & Memory](10-context-memory-engineering/) | Context budgeting, memory types, long-context degradation, cache reuse |
| 15 | [11 — Durable Agent Runtime](11-durable-agent-runtime/) | Execution graphs, checkpoint/resume, idempotency, HITL, side-effect boundaries |
| 16 | [12 — Evaluation Engineering](12-evaluation-engineering/) | Evaluation taxonomy, judge calibration, regression testing, offline-online mismatch |
| 17 | [13 — Falsification Engineering](13-falsification-engineering/) | Property-based testing, metamorphic testing, adversarial evaluation, Goodhart's Law |
| 18 | [14 — AI Security](14-ai-security/) | Threat modeling, prompt injection, retrieval poisoning, capability scoping |
| 19–20 | [15 — Model Adaptation](15-model-adaptation/) | SFT, LoRA, DPO, data flywheel, when to fine-tune vs prompt/retrieve/route |
| 21 | [16 — Distributed Inference](16-distributed-inference/) | TP, PP, DP, EP, interconnects, MoE placement, P/D disaggregation |
| 22 | [17 — AI Economics](17-ai-economics/) | Cost modeling, model routing, cascades, quality-latency-cost Pareto frontier |
| 23 | [18 — Observability & Reliability](18-observability-reliability/) | Distributed tracing, SLO, alerting, cost attribution, incident response |
| 24 | [19 — Model-System Co-design](19-model-system-codesign/) | Architecture-model coupling: attention → KV → HBM → scheduler → economics |
| 25 | [Capstone](capstone/) | War week — integration, load testing, failure injection, defense |
| 26 | [Graduation](graduation/) | Architecture review under ambiguous requirements and constraints |

## Progressive Integration

Modules are not isolated. Key cross-cutting concerns accumulate:

- **Observability** — introduced in Module 02, progressively deepened through every subsequent module
- **Benchmarking** — methodology from Module 00 applied in every lab
- **Security** — threat awareness surfaces in retrieval, agents, and evaluation before the dedicated module
- **Economics** — cost thinking introduced with inference and reinforced through every architectural decision
- **Capstone** — absorbs artifacts from Module 04 onward

## Pacing Notes

- Weeks 4–5 and 6–7 are doubled because KV cache and serving are foundational to everything downstream
- Weeks 8–9 cover optimization breadth; depth comes from choosing specific bottlenecks
- Weeks 19–20 cover both training techniques and data engineering for adaptation
- Week 25 is intentionally intense — the capstone should already be substantially built

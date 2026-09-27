# Curriculum Content Audit Summary — Module 04 Alignment

**Audit date:** 2026-09-27  
**Task phase:** Task 1 — existing authored modules only  
**Golden reference:** `04-serving-scheduling-capacity/README.md`

## Scope and Decision Rule

This audit covers Modules 00–16, the modules that already contained reviewed instructional content when Task 1 began. Module 04 is the structural reference and was not rewritten. Pre-existing generated placeholders for Modules 17–24 are excluded from Task 1; they belong to Task 2.

Alignment means a shared instructional and evidence contract, not identical subject matter. Each module keeps its own lesson count, technical mechanisms, equations, diagrams, O/D/H distinctions, currentness classifications, and pinned source trace. The common contract is:

1. `00 Why This Module Exists`, including Module Orientation.
2. `01 Baseline Assumptions`.
3. `02 Target Mastery`, with a common depth contract and effort estimate.
4. Layer 1: Knowledge Map, Lessons, and Literature & Production Source Map.
5. Layer 2: Engineering Labs and Break / Incident Scenarios.
6. Layer 3: Mastery Assessment, Required Evidence & Rubric, traceability, exit criteria, wrap-up, and competency targets.

Lesson, lab, incident, and assessment substructure follows Module 04 where the field applies. A module is not forced to include a meaningless equation or identical number of lessons.

## Why Content Was Added

### Orientation and target mastery

Module Orientation was made explicit so the learner can see the engineering problem, the work they will perform, and the required environment before beginning. The full depth contract prevents a concept-only module from appearing equivalent to one that also requires implementation, instrumentation, production reasoning, failure analysis, and falsification. Effort estimates make the expected depth operational rather than aspirational.

### Lesson scaffolding

Existing compressed prose was preserved and organized around an engineering question, definitions, mechanism or quantitative model, worked example, knowledge check, guided or independent practice, feedback contract, learning outcome, and effort. This was added because a technically correct survey does not by itself demonstrate that a learner can calculate, implement, break, diagnose, or transfer the mechanism.

### Experimental labs

Labs were expanded with a pre-registered hypothesis, controlled inputs, measured outputs, a break/falsify condition, lesson alignment, and effort. These fields prevent a build-only exercise from being reported as causal evidence and make negative or counterexample results first-class outcomes.

### Incident diagnosis

Incident sections now separate symptoms, competing hypotheses, missing evidence, discriminating measurements, ranked explanations, immediate and long-term interventions, and same-boundary remeasurement. This enforces diagnosis instead of guessing one bottleneck or root cause from an ambiguous dashboard.

### Transfer assessment, rubric, and exit criteria

Named transfer problems and numbered deliverables were added so mastery must survive a new workload rather than repeat lesson vocabulary. Required artifacts and leveled rubrics make source tracing, quantitative reasoning, experimental rigor, diagnosis, and architecture defense assessable. Separate exit criteria and final mental-model reconstruction make the pass condition and the durable concept distinct.

## Module-Specific Rationale

| Module | Additions and reason |
|---|---|
| 00 — Scientific AI Engineering | Added per-lesson checks and feedback around estimands, timestamp boundaries, pseudoreplication, quantiles, practical significance, provenance, and competing hypotheses so the evidence discipline is executable in every later module. |
| 01 — Foundation Model Internals | Added shape/numerical invariants, GQA/RoPE/RMSNorm/MoE worked checks, and backend-dispatch verification so architecture vocabulary connects to executable tensor and source-code behavior. |
| 02 — Inference GPU Fundamentals | Normalized the mastery schema and added timing, traffic, Roofline, shape-regime, profiler-intrusion, and source-trace exercises so bottleneck claims retain units, synchronization boundaries, and falsifiers. |
| 03 — KV Cache Engineering | Strengthened allocation, fragmentation, sharing, routing, compression, tiering, stale metadata, and copy-on-write experiments; expanded the memory-pressure incident so cache symptoms are not assumed to prove one cause. |
| 04 — Serving, Scheduling, & Capacity | Unchanged golden structural reference. Its topic content was not copied into unrelated modules. |
| 05 — Inference Optimization | Added explicit attention, fusion, quantization, speculative-decoding, composition, and counterexample exercises so isolated speedups must pass correctness, quality, memory, latency, and goodput gates. |
| 06 — Reasoning & Test-Time Compute | Added oracle-versus-selected accuracy, correlated pass@k, verifier error, branch budgets, critical-path latency, and stopping checks so “more reasoning” is evaluated as a controlled compute policy. |
| 07 — Model Behavior & Uncertainty | Added label/adjudication quality, subgroup calibration, proper-score, risk–coverage, and conformal-scope examples so aggregate confidence does not become a per-request guarantee. |
| 08 — AI Data Engineering | Added replay identity, join/validation failure, drift and detector quality, contamination residual risk, synthetic-mixture ablation, label-system QA, retention/deletion, and active-labeling economics so a dataset release is an auditable lifecycle. |
| 09 — Retrieval Engineering | Added lexical/dense/ANN/fusion/reranking calculations, filtered-ANN diagnosis, index update/delete convergence, loaded-service measurements, and authorization invariants so relevance and approximation loss remain distinguishable. |
| 10 — Advanced RAG | Added visibility and caching semantics, query/context transformations, temporal conflict, citation-stage attribution, partial failure, and route-utility exercises so answer quality can be traced to the earliest failed evidence boundary. |
| 11 — Context & Memory Engineering | Added context accounting, effective-context tests, multi-writer correction, deletion/TTL convergence, compaction loss, and stored→valid→selected→placed→used metrics so application memory is not confused with a large context window. |
| 12 — Agent Loop Engineering | Added controller transition tables, unknown-effect handling, multidimensional budgets, stall-versus-polling tests, reflection baselines, authority/postcondition checks, and offered-episode goodput so an agent is treated as a bounded controller. |
| 13 — Harness Engineering | Added canonical→effective request diffs, capability decisions, validation-ladder examples, stream-state traces, retry/deadline amplification, and replay checks so protocol compatibility is demonstrated rather than assumed. |
| 14 — Durable Agent Runtime | Added crash-window, delivery/idempotency, outbox, saga, fencing, fan-out, history-capacity, and replay-safe upgrade exercises so durable orchestration does not imply exactly-once external effects. |
| 15 — Evaluation Engineering | Added paired/block-aware examples, slice weighting, judge order and disagreement checks, evaluation-budget planning, online experiment integrity, and offered-request accounting so a score becomes decision evidence rather than benchmark theater. |
| 16 — Falsification Engineering | Added complete property contracts, valid generation/shrinking, sound and unsound metamorphic examples, differential/mutation adequacy, IID-qualified bounds, adaptive-search counterexamples, and portfolio economics so test volume or coverage is not mistaken for correctness. |

## Evidence and Currentness Policy

- New empirical facts require a Tier 1 or Tier 2 source and scoped applicability.
- New numerical exercises use declared hypothetical inputs unless reproduced measurements already exist with full context.
- Runtime behavior remains pinned to an exact repository revision, path, and symbol; static inspection is not described as execution.
- O — source observations, D — derivations, and H — engineering hypotheses remain separate.
- Module-specific REFERENCE / CURRENT DEFAULT / WORKLOAD-DEPENDENT / FRONTIER / LEGACY classifications are retained where they add information beyond the golden format.

## Verification Record

The final Task 1 verification must record:

- exact heading/order compliance for Modules 00–16;
- per-lesson and per-lab scaffold coverage;
- evidence-registry and knowledge-model schema validity;
- claim, evidence, and dependency cross-reference integrity;
- retained source URL and pinned source-symbol checks;
- Markdown/diff hygiene and absence of generated placeholders;
- explicit confirmation that Module 04 was not modified;
- explicit confirmation that Modules 17–24 were not accepted as Task 1 content.

Final counts and any scoped exceptions are appended only after these checks pass.

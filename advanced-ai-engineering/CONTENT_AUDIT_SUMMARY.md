# Curriculum Content Audit Summary — Module 04 Alignment

**Audit date:** 2026-09-27 (Task 1); correction and authoring pass 2026-09-30 to 2026-10-03\
**Task phase:** Task 1 and Task 2 complete — Modules 00–24\
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

The checklist above was written on 2026-09-27 for Task 1 (Modules 00–16). The record below states what was actually verified in the correction and authoring pass of 2026-09-30 to 2026-10-03, which followed the root audit (`../audit.md`) and replaces the earlier plan. Earlier "audited" labels for Modules 00–16 did not mean the later audit findings were already fixed.

### Scope completed

- **Baseline (AUD-B01–B04):** Module 04 was corrected rather than left unchanged: absolute parameter units with explicit GB/GiB, an expectation-form latency identity with a covariance counterexample, Little's Law conditions, reconciled effort, a solved swap/recompute example, and a solvable mastery fixture.
- **Authored modules (AUD-00 to AUD-19):** every listed finding for Modules 00–19 was addressed by one owner per module. P1 correctness items were fixed before depth items.
- **Outline modules (AUD-N20–N24):** Modules 20–24 were authored from their outlines to the same contract, each with an evidence registry, knowledge model, pinned production source trace, six lessons, four labs, an incident, and a mastery fixture.
- **Shared contracts (AUD-G01–G02):** `INSTRUCTIONAL_SCHEMA.md` now matches the Module 04 spine; `CURRICULUM.md`, `ROADMAP.md`, and `README.md` use the real 00–24 map with no broken local links; the evidence and knowledge-model schemas reject VERIFIED_FACT claims without evidence, unpinned source-code evidence, and empty published knowledge models, enforced by negative fixtures and `tools/validate_curriculum.py`.

### Static results (validator, 2026-10-03)

- 25 modules, 160 lessons, 102 labs, 665h of declared module effort; program total ≈755–770h against the 730–780h budget.
- 559 registry claims, 718 knowledge-model entries, 1,910 knowledge-model evidence references; all resolve.
- Every module: schema-valid registry and knowledge model, identical H2 spine to Module 04, lesson/lab targets resolved, O/D/H labels matching registry provenance, effort components summing to the declared total, no empty Concepts subsections. All negative fixtures are rejected.

Passing the validator does not certify technical correctness; semantic review was done per module by its owner and spot-checked during integration (recomputed worked numbers, pinned commits and symbols opened at their revisions).

### Known limits and open items

- **Static inspection only.** No lab, benchmark, or pinned runtime was executed. Every source trace is a static read at a pinned commit; worked-example and fixture numbers are synthetic and labeled.
- **TODO_VERIFY items remain** in most modules, recorded in each registry: industry-prevalence surveys were not done, some papers were read at abstract level only, and some 2025–2026 frontier areas were searched but not surveyed exhaustively. None supports a required outcome.
- **Verification dates.** Some pinned sources were re-read on 2026-10-01, one day after the 2026-09-30 research cutoff; the dated sources themselves fall on or before the cutoff.
- **Citation coverage.** Modules 04, 10, 11, and 12 have registry claims that are not cited inline in the README; this is reported as a validator warning, not an error.
- **Per-package reports** are in `audit-reports/` (gitignored): `WP-B`, `WP-F1`, `WP-F2`, `WP-D1`, `WP-D2`, `WP-A1a`, `WP-A1b`, `WP-A2a`, `WP-A2b`, and `module19-review` through `module24-audit`.

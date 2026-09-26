# Module 10 — Advanced RAG

## 00 Why This Module Exists

RAG is not “retrieve top-k and paste it into a prompt.” It is a versioned evidence system whose answer depends on source state, index visibility, query transformations, candidate retrieval, context selection and order, generator behavior, citation binding, and conflict policy.

```text
source change -> parse/chunk/embed -> candidate index -> query-visible revision
                                                       |
query -> preserve/rewrite/decompose/route -> retrieve/rerank
      -> select/deduplicate/order context -> generate claims
      -> bind citations -> verify/support/abstain -> response
```

A wrong answer can therefore arise because evidence did not exist, was not yet visible, was not retrieved, was omitted from context, was displaced by order or noise, contradicted another version, was ignored by the model, or was cited incorrectly. Those failures require different fixes.

This module owns query orchestration, incremental-index freshness and version consistency, context assembly, contradiction handling, citations/grounding, staged RAG diagnosis, and RAG-versus-long-context decisions. Module 08 owns upstream data lineage, Module 09 owns retrieval/index/fusion/reranking mechanics, Module 11 owns general context and memory engineering, and Module 15 owns the organization-wide evaluation program.

**Research cutoff:** 2026-09-26.

- **Engineering problem:** deliver evidence-backed answers from a mutable corpus under quality, freshness, consistency, latency, cost, access, and abstention constraints.
- **Evidence rule:** keep source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**) distinct. A paper result remains scoped to its model, corpus, retriever, prompt, task, and metric.

## 01 Baseline Assumptions

- Module 00: measurement boundaries, paired tests, uncertainty, and falsification.
- Module 07: behavioral failures, uncertainty, and selective prediction.
- Module 08: stable identity, provenance, dataset versions, and deletion/rollback.
- Module 09: lexical/dense retrieval, exact/ANN separation, fusion, reranking, and retrieval metrics.
- Module 04: latency distributions, queueing, goodput, overload, and capacity.
- Mathematical prerequisites: sets, conditional probability intuition, constrained optimization, critical paths, timestamps, and empirical distributions.

## 02 Target Mastery

```yaml
depth_contract:
  conceptual: REQUIRED
  mechanistic: REQUIRED
  mathematical: REQUIRED
  quantitative: REQUIRED
  implementation: REQUIRED
  source_code: REQUIRED
  instrumentation: REQUIRED
  falsification: REQUIRED
```

The learner must be able to define an end-to-end RAG contract; build idempotent incremental updates and deletes; prove query visibility and snapshot lineage; test query rewrites and context order; resolve or expose contradictions; validate citations at claim level; attribute failures by stage; and choose RAG, long context, or a hybrid route from matched evidence.

## 03 Knowledge Map

```text
                    mutable source-of-truth
                             |
             stable ID + version + event time
                             v
      parse -> chunk -> embed -> write -> refresh/commit
                             |
                   visible high-watermark
                             |
request contract -> query transform -> retrieve/rerank
                             |
            evidence eligibility + context budget
                 /           |            \
          authority       conflict       ordering
                 \           |            /
                    assembled context
                             |
                  generator claims
                             |
            citation binding + verification
                             |
                  answer or abstention
```

Every request must preserve enough lineage to answer: which source versions were eligible, which were actually retrieved, what entered the prompt and in what order, which claim cites which immutable evidence span, and which policy resolved a conflict.

## 04 Lessons

### Lesson 10.1 — The RAG Request and Evidence Contract

Pin the request's query intent and time, tenant/access scope, corpus or index revision, original and transformed queries, retrieval/context budgets, model and prompt versions, answer format, citation semantics, and abstention policy. “Latest” is not a version. A response built from unrecorded mutable state cannot be reproduced or audited.

The original RAG architecture is a reference: generation is conditioned on retrieved non-parametric evidence. Production systems add many lossy boundaries. Keep three values separate:

- **available:** the required evidence exists in the authorized source snapshot;
- **opportunity:** the required evidence reaches candidates or assembled context;
- **use/support:** the generator uses it correctly and citations entail the claims.

**Outcome:** capture a complete request trace rather than only the final prompt and answer.

### Lesson 10.2 — Incremental Indexing, Visibility, and Consistency

An update lifecycle is not complete when a worker reports success:

```text
source event -> detected -> parsed -> chunked -> embedded -> written
             -> refreshed/committed -> replicated/routed -> cache-visible
             -> retrieved by a production-equivalent probe
```

Define the freshness measurand:

$$
L_{visible}=t_{query\ visible}-t_{source\ event}.
$$

The relation is exact for declared endpoints; decomposing it into stage times requires the actual dependency graph because work may overlap. Record event time semantics and clock skew. Report a distribution and stale-read fraction by source/slice, not only a mean.

Stable IDs and monotonic versions make retries idempotent. Updates and deletes must converge across text, chunks, embeddings, metadata, caches, replicas, and citation targets. A tombstone that reaches lexical search but not vector search is not a completed delete. A high-watermark or read-snapshot contract prevents mixed-version answers; the required strength is product-specific.

Elasticsearch provides one concrete example: a write can be acknowledged before it becomes searchable; refresh controls visibility and costs work, while sequence numbers and primary terms support conditional updates. This is implementation evidence, not a universal store definition.

**Outcome:** prove update/delete visibility and rollback end to end under reordered and duplicated events.

### Lesson 10.3 — Query Orchestration and Context Assembly

Preserving the original query is a valid route. Rewriting, expansion, decomposition, and multi-step retrieval are interventions that can improve evidence opportunity or silently alter identifiers, negation, scope, entity, and time. Log every transformation and compare it with the original using paired slices.

Context assembly is constrained selection, not concatenation. Candidate evidence differs in relevance, authority, freshness, length, redundancy, conflict, and access eligibility. Deduplicate by stable identity/version, retain source boundaries, budget tokens, and vary order deliberately.

Fusion-in-Decoder shows that a model can aggregate multiple retrieved passages in a scoped QA setup. Lost in the Middle and FreshLLMs show why “more context” is not a universal rule: evidence position, amount, and order can matter. Test depth/order matrices, relevant-plus-distractor mixtures, duplicate amplification, and truncation for the target model and prompt.

**Outcome:** demonstrate which query and packing policy improves answer/citation quality at matched latency and cost, including slices where it fails.

### Lesson 10.4 — Contradiction Is a Data-and-Policy Problem

Contradiction can occur among source documents, revisions of one source, retrieved context and model memory, or the answer and its cited passage. Normalize the contested claim before choosing a winner:

```text
(subject, predicate, object, valid_time, transaction/version,
 source_identity, authority, scope)
```

Two passages may be temporally different rather than contradictory. Ten mirrors of a stale article do not outrank one authoritative update. “Newest” is unsafe when publication time differs from effective time. Model confidence does not establish source authority.

Real-document experiments report that models can retain incorrect parametric answers despite corrective context. Therefore test both directions: trusted context correcting model memory, and untrusted context attempting to override a valid prior. If authority/time/scope cannot resolve the evidence, surface the conflict or abstain.

**Outcome:** implement an auditable authority/temporal policy and a deliberate unresolved-conflict state.

### Lesson 10.5 — Citations, Grounding, and Stage Attribution

A citation has at least two independent properties:

$$
CitationCorrectness=\frac{supported\ emitted\ citations}{checked\ emitted\ citations}
$$

$$
CitationCompleteness=\frac{support\text{-}required\ claims\ with\ adequate\ citation}
{support\text{-}required\ claims}.
$$

Declare claim segmentation, support threshold, multi-source requirements, citation granularity, and zero-denominator policy. A syntactically valid `[3]` can point to a real but irrelevant document. ALCE separates answer correctness and citation quality; current source inspection of Haystack likewise shows numeric reference parsing/binding, not semantic entailment verification.

Diagnose an incorrect claim in order:

```text
source absent?
  -> version invisible?
  -> not retrieved?
  -> retrieved but not selected?
  -> selected but ignored/misread?
  -> correct claim but wrong/incomplete citation?
```

Record evaluator identity and uncertainty. Automated NLI or LLM judges are measurements with errors, not ground truth.

**Outcome:** produce a claim-evidence table and assign failures to the earliest discriminated stage.

### Lesson 10.6 — RAG, Long Context, and Adaptive Routes

Compare architectures with the same corpus snapshot, eligible evidence, base model where possible, prompt objective, answer set, and load. Include retrieval/index work, input tokens, cached prefixes, latency tails, failures, and evaluator costs. A large advertised context window does not prove robust evidence use; retrieval may miss evidence that full context contains; full context may dilute or misorder evidence that retrieval isolates.

A useful engineering decision is constrained rather than ideological:

$$
U(route)=Q(route)-\lambda_L L(route)-\lambda_C C(route),
$$

subject to hard access, safety, freshness, and consistency constraints. This is a product-specific heuristic: quality definition and weights are not universal. Published comparisons through 2025 report different task-dependent outcomes, which is evidence against a universal winner.

Self-RAG, corrective RAG, and hybrid routing demonstrate adaptive mechanism families. They add evaluator/router errors, correlated self-judgment, extra model or search calls, new source-quality risks, latency, and cost. Treat them as workload-dependent/frontier until calibrated on the target service.

**Outcome:** build a measured route policy with fallback and a falsifiable decision log.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) — Lewis et al. (NeurIPS 2020).
- [Leveraging Passage Retrieval with Generative Models for Open Domain Question Answering](https://arxiv.org/abs/2007.01282) — Izacard and Grave (2021).
- [Lost in the Middle](https://arxiv.org/abs/2307.03172) — Liu et al. (TACL 2023).
- [Enabling Large Language Models to Generate Text with Citations](https://arxiv.org/abs/2305.14627) — Gao et al. (EMNLP 2023).
- [FreshLLMs](https://arxiv.org/abs/2310.03214) — Vu et al. (2023).

**CURRENT DEFAULT:** stable document/version identity, query-visible freshness probes, idempotent update/delete paths, request/context/citation lineage, explicit conflict policy, claim-level support checks, staged diagnosis, and matched end-to-end evaluation.

**WORKLOAD-DEPENDENT:** query rewrite/decomposition, context depth and order, compression, authority rules, citation granularity, long-context use, retrieval frequency, and route thresholds.

**FRONTIER:** learned or self-reflective retrieval decisions, corrective retrieval/evidence evaluators, adaptive RAG/long-context routing, and 2025–2026 task-specific context orchestration. A published mechanism is not an industry default.

**LEGACY / INSUFFICIENT:** one-shot static top-k stuffing; “indexed” equated with query-visible; newest/frequent source automatically trusted; citation marker treated as support; answer score used as the only RAG metric; RAG or long context declared universally superior.

**PRODUCTION SOURCE TRACE**

- Repository: `deepset-ai/haystack`
- Revision: `8a5406eea71a0fc19e94c4b9a5cd96df2158a45a`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `Pipeline.run/_run_component`, `PromptBuilder.run`, `DocumentWriter.run`, and `AnswerBuilder.run` in the registry-recorded paths.
- Execution: pipeline dispatch invokes component `run`; prompt variables are rendered; document writes delegate to the configured store; answer building parses reference indices and attaches document copies. No semantic citation-entailment validation was observed in that path.
- Scope: current pinned Haystack behavior, not a general definition of RAG orchestration or consistency.

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Incremental Index Visibility and Delete Convergence

- Build a versioned source/change ledger and two query representations with a production-equivalent probe.
- Inject duplicated, reordered, delayed, failed, and replayed upserts/deletes plus cache and replica lag.
- Measure source-to-query visibility distributions, stale-read fraction, version monotonicity, partial representation state, access violations, and rollback time.
- Falsify “worker success means fresh” and prove idempotence with replay.
- Artifact: event ledger, high-watermarks, visibility trace, invariant checks, and recovery report.

### LAB B — Query and Context Phase Diagram

- Compare original, rewritten, expanded, and decomposed queries at matched retrieval budgets.
- Sweep evidence depth, order, duplicate rate, distractor ratio, conflict ratio, compression, and token budget.
- Measure evidence opportunity, intent fidelity, answer quality, claim support, latency, tokens, and cost by identifiers, negation, temporal, multi-hop, and no-answer slices.
- Falsify “more context is better” and “rewrite always improves recall.”

### LAB C — Contradiction and Citation Audit

- Construct versioned claim sets with true updates, scope differences, authority conflicts, stale mirrors, model-memory conflicts, partial support, and uncited claims.
- Implement authority/time/version policy, unresolved-conflict output, claim segmentation, citation binding, correctness, and completeness.
- Compare automatic entailment judgments with a blinded human adjudication sample and record disagreement.
- Break numeric references, source order, deletes, and multi-source claims; verify that syntax never substitutes for support.

### LAB D — Matched RAG/Long-Context/Hybrid Routing

- Run retrieval, full/long context, and one adaptive hybrid on identical source snapshots, query sets, answer contracts, and model families.
- Include answerable-without-context controls, shuffled evidence, no-answer cases, temporal updates, long noisy documents, and load.
- Measure quality/support, input/output tokens, stage and end-to-end latency distributions, cost, freshness, abstention, and SLO-goodput.
- Calibrate the router, analyze false routes, include router/evaluator work, and define safe fallback and rollback.

## 07 Break / Incident Scenarios

### Incident 10.1 — Fresh Answer Metric, Stale and Unsupported Production Answers

After an incremental-index release, average answer accuracy improves, but some tenants receive deleted policies, recent updates are absent for minutes, contradictory versions are blended, and citations open valid documents that do not support the attached claims.

Competing explanations include source polling lag, parse/embed failures, acknowledged-but-unrefreshed writes, replica or alias skew, stale query/result caches, out-of-order retries, incomplete tombstone fanout, rewrite drift, retriever loss, context order/dilution, parametric bias, generator misuse, reference-index bugs, or citation evaluator error.

Recover source event/version logs, representation status, query-visible high-watermarks, route/cache/replica identity, original and transformed queries, candidates, exact assembled context/order, prompt/model version, generated claims, citation bindings, and stage timings. Replay matched requests at pinned snapshots; probe each representation directly; disable transformations/caches one at a time; compare opportunity, use, and support; apply the smallest discriminated intervention; then remeasure freshness and citation distributions plus rejected/abstained utility.

## 08 Mastery Assessment

Design Advanced RAG for mutable technical and policy documentation with tenant ACLs, frequent updates/deletes, contradictory versions, citations, and a latency/cost SLO. Deliver the request/read contract; incremental event ledger and convergence invariants; freshness probe; query/context ablations; contradiction/authority policy; claim-citation evaluator with human calibration; matched RAG/long-context/hybrid study; current source trace; canary/rollback plan; and an incident diagnosis that separates source, visibility, retrieval, selection, use, and support failure.

## 09 Required Evidence & Rubric

- **Lineage:** stable IDs, versions, source/index revisions, transformations, context order, model/prompt, claims, and citations are reproducible.
- **Freshness:** endpoints, event-time semantics, clocks, high-watermarks, distributions, stale reads, deletes, retries, and rollback are tested.
- **Context:** opportunity, depth, order, duplication, distractors, conflict, truncation, latency, and cost are measured by slice.
- **Contradiction:** entity/time/version/authority semantics and unresolved-conflict behavior are explicit.
- **Citations:** correctness and completeness are separate; automatic support judgments are calibrated.
- **Architecture:** RAG/long-context/hybrid paths use matched evidence and include all material work and failures.
- **Diagnosis:** claims follow `symptom → hypotheses → missing evidence → discriminating measure → intervention → remeasurement`.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Versioned incremental RAG | 10.1–10.2 | LAB A | Incident / Mastery | Ledger, watermark, convergence tests |
| Query and context orchestration | 10.3 | LAB B | Mastery | Paired phase diagram and traces |
| Contradiction and citations | 10.4–10.5 | LAB C | Incident / Mastery | Claim-evidence audit and adjudication |
| RAG vs long context | 10.6 | LAB D | Mastery | Matched frontier and routing errors |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to demonstrate query-visible freshness rather than ingestion success; survive duplicate/reordered updates and deletes; reproduce an answer from immutable lineage; identify rewrite and context-order failures; preserve unresolved contradictions; distinguish citation syntax, correctness, and completeness; trace a current implementation without generalizing it; and defend a workload-specific RAG/long-context route under quality, latency, cost, freshness, access, and abstention constraints.

**Final mental model:** RAG is a versioned evidence-control loop. Preserve identity across every transformation, measure visibility at the query boundary, treat context as a constrained selection problem, keep contradictions explicit, require claim-level support, and choose architecture from matched evidence rather than fashion.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

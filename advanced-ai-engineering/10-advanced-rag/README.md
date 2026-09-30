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

**Research cutoff:** 2026-09-30. Sources marked as re-opened on that date are listed in the registry; other entries keep their original access dates.

**Module Orientation**

- **Engineering problem:** deliver evidence-backed answers from a mutable corpus under quality, freshness, consistency, latency, cost, access, and abstention constraints.
- **What you will do:** define a versioned RAG request contract; implement replay-safe updates and deletes; measure query-visible freshness; compare query and context policies; resolve temporal/authority conflicts; audit claim-level citations; test RAG against long-context and hybrid routes; and diagnose stale unsupported answers.
- **Environment:** Python 3.10+ with a versioned document store/search fixture and reproducible generation endpoint; optional distributed index and load generator for replica/cache/failure tests. Pin source, index, query transformation, prompt, model, evaluator, and cache revisions.
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
  experimental: REQUIRED
  statistical: REQUIRED
  production_reasoning: REQUIRED
  failure_analysis: REQUIRED
  falsification: REQUIRED
  security: REQUIRED
  economics: REQUIRED
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 5h        # lesson instruction: 35+65+45+50+55+50 min = 300 min
  guided_practice: 3h    # lesson practice: 30+35+30+25+30+30 min = 180 min
  labs: 18h              # LAB A 4.5h + LAB B 4.5h + LAB C 4.5h + LAB D 4.5h
  assessment: 3h         # Mastery transfer problem 2.5h + Incident 10.1 0.5h
  source_trace: 2h       # Section 05 Haystack trace and Mastery Deliverable 8, counted once
  total: 31h
```
Each category is counted once. Lab analysis is not also counted as guided practice, and the source trace is not also counted as assessment time.

The learner must be able to define an end-to-end RAG contract; build idempotent incremental updates and deletes; prove query visibility and snapshot lineage; test query rewrites and context order; resolve or expose contradictions; validate citations at claim level; attribute failures by stage; and choose RAG, long context, or a hybrid route from matched evidence.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

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

**Engineering Question:**
Can a delivered answer be reconstructed from an immutable request and evidence boundary?

**Concepts & Definitions:**

Pin the request's query intent and time, tenant/access scope, corpus or index revision, original and transformed queries, retrieval/context budgets, model and prompt versions, answer format, citation semantics, and abstention policy. “Latest” is not a version. A response built from unrecorded mutable state cannot be reproduced or audited.

The original RAG architecture is a reference: generation is conditioned on retrieved non-parametric evidence. Production systems add many lossy boundaries. Keep three values separate:

- **available:** the required evidence exists in the authorized source snapshot;
- **opportunity:** the required evidence reaches candidates or assembled context;
- **use/support:** the generator uses it correctly and citations entail the claims.

**Worked Example:** Replay one request against its recorded snapshot and against “latest.” Different answers under latest show why a mutable alias is not sufficient lineage.

**Knowledge Check:** Distinguish available evidence, candidate/context opportunity, and correct use/support.

**Guided Practice:** Specify and validate a request trace containing all revisions, transformations, candidates, context order, claims, citations, and finish state.

**Feedback Contract:**
- *Expected Output*: a trace record whose replay against the recorded snapshot reproduces the same eligible set, candidates, context order, and citation targets. Any dependency that cannot be replayed (for example, a hosted model revision that is no longer served) is listed explicitly as unavailable.
- *Common Failure*: storing only the final prompt and answer. The prompt shows what the model saw. It does not show which evidence was eligible but not retrieved, or which query transformation ran.
- *Diagnostic Hint*: remove one field from your trace. Can you still tell whether a wrong answer came from a missing source, an invisible version, or a retrieval miss? If you can, that field was not doing any work.
- *Concept to Revisit*: available versus opportunity versus use/support.

**Learning Outcome:** Capture a complete request trace rather than only the final prompt and answer.

*(Effort: 35m instruction, 30m practice)*

### Lesson 10.2 — Incremental Indexing, Visibility, and Consistency

**Engineering Question:**
When is a source update or deletion actually visible to production-equivalent reads?

**Concepts & Definitions:**

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

**Stable IDs and versions are inputs to idempotency; they do not create it.** A retry is safe only if the *store* applies an acceptance rule to every write, and applies it atomically. Each logical source item has a stable `doc_id`. The source assigns each change a monotonically increasing `version` (an event sequence number or source revision, never a worker's wall clock). The store keeps one record per `doc_id`: `(version_s, content_hash_s, state_s ∈ {live, tombstone})`. An incoming event `(op, doc_id, version_e, hash_e)` is decided as follows (**D**, CLM-018):

| Stored state | Incoming event | Decision | Store after | Acknowledge source as |
|---|---|---|---|---|
| no record | upsert / create | ACCEPT | live, `version_e` | applied |
| no record | delete | ACCEPT as tombstone | tombstone, `version_e` | applied |
| any | `version_e > version_s` | ACCEPT (upsert → live, delete → tombstone) | new state, `version_e` | applied |
| any | `version_e = version_s`, same op, same hash | DUPLICATE (no write) | unchanged | applied (return the stored outcome) |
| any | `version_e = version_s`, different op or hash | CONFLICT (no write, alert) | unchanged | rejected; the same key was reused with a different intent |
| any | `version_e < version_s` | STALE (no write) | unchanged | superseded |

Four conditions make this rule an idempotency guarantee rather than a hope:

1. **Atomicity.** Compare and write must be one atomic step per `doc_id`: a conditional write, a compare-and-set, or a single-writer partition. "Read the stored version, then write" in two steps lets two workers both pass the check. Elasticsearch implements such conditions: `version_type=external` fails the write with a 409 conflict when the supplied version is less than or equal to the stored one, `if_seq_no`/`if_primary_term` make a write conditional on the last modification, and `op_type=create` fails if the ID already exists (**O**, CLM-019). Those are one store's semantics, not a definition for every vector store.
2. **Same rule in every representation.** Text, chunks, embeddings, metadata, and citation-target stores each apply the rule with the *source* version, not a local counter. A tombstone that reaches lexical search but not vector search is not a completed delete.
3. **Tombstones outlive the replay window.** A delete must leave a versioned tombstone. If it is physically removed (or the delete was unversioned) before every delayed or retried event for that `doc_id` has arrived, a late upsert meets "no record" and is accepted. The deleted document comes back.
4. **Acknowledge after the decision is durable.** The ingestion worker acknowledges a source event only after the ACCEPT/DUPLICATE/STALE/CONFLICT outcome is durably recorded. That acknowledgement means "decided", not "query-visible".

This is the same contract Module 14 (Lesson 14.2) develops for external effects: the receiver stores identity, canonical intent (here, version plus content hash), and outcome atomically, and rejects reuse of a key with different intent. Module 10 applies it to index writes. Durable workflow recovery stays in Module 14.

**Solved traces for one `doc_id`** (synthetic events; hashes `hA ≠ hB`):

| Trace | Events in arrival order | Decisions | Final state | What last-writer-wins would do |
|---|---|---|---|---|
| Reordered retry | upsert v2 hA; upsert v1 hB; upsert v2 hA | ACCEPT; STALE; DUPLICATE | live v2 hA | ends at v2, but v1 content is served between the second and third event |
| Same order, different tail | upsert v2 hA; upsert v2 hA; upsert v1 hB | ACCEPT; DUPLICATE; STALE | live v2 hA | ends at **v1**: an older version wins permanently |
| Duplicate create | create v1 hA; create v1 hA; create v1 hB | ACCEPT; DUPLICATE; CONFLICT | live v1 hA | second create overwrites or errors; a changed body under the same version is silently accepted |
| Delete then delayed upsert | upsert v3; delete v4; *(retry of)* upsert v3 | ACCEPT; ACCEPT (tombstone v4); STALE | tombstone v4 | the deleted document is live again |
| Same, tombstone collected before the retry arrives | upsert v3; delete v4; *GC*; upsert v3 | ACCEPT; ACCEPT; —; ACCEPT (no record) | **live v3: resurrected** | same failure |

The last row is the reason for condition 3. The fix is a tombstone retention period longer than the maximum event delay plus replay window, and a convergence check across all representations before garbage collection.

**Progress watermark versus pinned read snapshot.** These are different objects:

- A **progress watermark** $H_r$ for representation $r$ is the largest source sequence number $s$ such that every event $\le s$ has been decided and is visible in $r$. It is a monotone progress report. By itself it guarantees nothing about what one request reads.
- A **pinned read snapshot** $S$ is chosen once per request, and *every* read for that request is served at $S$: lexical, vector, metadata, citation-target fetch, and cache. Enforcement needs one of: a store-level point-in-time or generation/alias read; or an application filter that drops any hit with version newer than $S$ and refuses (or waits for) representations with $H_r<S$. If a representation cannot serve $S$, the response is marked partial or the request abstains.

Deletions obligated by policy or law are a separate hard constraint. A snapshot older than a delete must still exclude the deleted item, typically through a tombstone overlay checked at read time.

**Mixed-version read fixture** (synthetic). Event 101 updates `A` from v1 to v2; event 102 deletes `B`. At request time $H_{lex}=102$, $H_{vec}=100$, and a cached candidate list was built at 100.

| Read policy | What the request sees | Decision |
|---|---|---|
| No pinning | lexical returns `A` v2; vector and cache return `A` v1 chunks and `B` | **Mixed-version answer.** Context contains both versions of `A` and a deleted document. Not acceptable. |
| Pin $S=\min_r H_r=100$, no overlay | `A` v1 and `B` everywhere | consistent but stale, *and* serves a deleted document. Fails the delete constraint. |
| Pin $S=100$ plus tombstone overlay through 102 | `A` v1, `B` excluded | consistent; answer marked stale by the lag of event 101. Acceptable if the freshness SLO allows it. |
| Pin $S=102$ | lexical ready; vector must catch up | wait up to the deadline for $H_{vec}\ge102$; otherwise answer from lexical only, marked partial, or abstain. |

**Cache keys.** Including $S$ (or the index generation), the transformation/prompt versions, and the access scope in the cache key is one strategy. It stops a response built at one snapshot from being served to a request pinned at another. It does not remove the need for delete overlays and for validating the version of each cached citation target, because a key built at an old $S$ can still be requested legitimately. Timeout and retry handling must expose whether a response is stale, partial, failed, or a safe abstention. A fallback that silently serves an older snapshot changes the consistency contract.

**Worked Example — freshness ledger with two replicas and a cache** (synthetic values; times in seconds after the source event at $t=0$):

| Stage | Time | Note |
|---|---:|---|
| source event (document `P` updated v7→v8) | 0.0 | declared endpoint $t_{event}$ |
| change detected, parsed, chunked, embedded | 9.0 | |
| write acknowledged by primary | 11.0 | $t_w$ |
| refresh makes v8 searchable on replica R1 | 12.5 | |
| replica R2 applies and refreshes | 19.0 | R2 is lagging |
| result cache entry for the popular query, built at | 5.0 | TTL 60 s, holds v7 |
| delete of document `Q` (same event batch): lexical tombstone applied | 12.0 | |
| delete of `Q`: vector delete fails, retried and applied | 40.0 | branch failure, not lag |

Steps:

1. Per-path visibility: $L_{visible,R1}=12.5$ s and $L_{visible,R2}=19.0$ s. The post-write portion $t_v-t_w$ is 1.5 s on R1 and 8.0 s on R2. Reporting only $t_v-t_w$ would hide the 11 s before the write.
2. Probes run once per second from $t=0$ to $t=30$ (31 probes), alternating R1 (even seconds) and R2 (odd seconds), bypassing the cache. Stale probes: 7 on R1 ($t=0,2,\dots,12$) and 9 on R2 ($t=1,3,\dots,17$), so $16/31\approx0.516$ of probes in the window are stale.
3. The cache is independent of both replicas. With no invalidation, the cached v7 answer is served until $t=65$. With invalidation at $t=12.0$ and a cache miss refilled from R2 at $t=12.2$, the refill stores v7 again and serves it until $t=72.2$. Invalidation without a version check made the cache *later* than either replica. A cache key or entry check that requires version ≥ v8 prevents that refill.
4. For `Q`, the lexical path stops returning it at 12 s, but the vector path still returns it until 40 s. That is an incomplete delete, not refresh lag.

Interpretation: an "index refresh lag" hypothesis predicts that direct replica probes and cached answers become fresh at the same time. At $t=30$ both direct replica probes return v8 while the cached path returns v7, so index lag is **refuted** as the cause of staleness after 19 s. The remaining causes are the cache and, for `Q`, the failed vector delete. Limits: the numbers are synthetic, the probe schedule is deterministic, and a real measurement needs clock-skew bounds and a distribution over many events, not one document.

**Knowledge Check:**
1. Why does worker success not establish cache or replica visibility?
2. In the trace `upsert v2; delete v3; upsert v2 (retry)`, what is the final state, and what extra condition makes it safe?
3. Why does a progress watermark not stop a request from reading two versions of one document?

**Guided Practice:** Inject duplicate, reordered, delayed, failed, and timed-out events across text/vector/cache representations; probe high-watermarks and deletion convergence. Then decide these two traces with the acceptance table: (a) `create v1 hA; upsert v3 hB; upsert v2 hC; delete v3`; (b) `upsert v5; delete v6; GC after 30 s; upsert v5 retry arriving at 45 s`.

**Feedback Contract:**
- *Expected Output*: (a) ACCEPT, ACCEPT, STALE, CONFLICT: the delete reuses version 3 with a different op, so it is rejected and alerted, and the item stays live at v3 hB. The source must issue the delete as v4. (b) The retry meets "no record" and is accepted, resurrecting the document; the design fails unless tombstone retention exceeds 45 s. Your ledger should also report visibility as a distribution per representation, the stale/partial rate, and a rollback proof.
- *Common Failure*: averaging only successful updates; or treating stable IDs plus versions as idempotent without an atomic conditional write.
- *Diagnostic Hint*: for each event ask "who compares the version, and can two workers both pass that comparison?" For each stale read ask "was the replica behind, or did the cache refill from a replica that was behind?"
- *Concept to Revisit*: conditional acceptance rule; progress watermark versus pinned read snapshot; Module 14 Lesson 14.2 (atomic idempotency).

**Learning Outcome:** Prove update/delete visibility and rollback end to end under reordered and duplicated events.

*(Effort: 65m instruction, 35m practice)*

### Lesson 10.3 — Query Orchestration and Context Assembly

**Engineering Question:**
Which query transformation and context packing choices improve evidence opportunity without changing intent?

**Concepts & Definitions:**

Preserving the original query is a valid route. Rewriting, expansion, decomposition, and multi-step retrieval are interventions that can improve evidence opportunity or silently alter identifiers, negation, scope, entity, and time. Log every transformation and compare it with the original using paired slices.

Context assembly is constrained selection, not concatenation. Candidate evidence differs in relevance, authority, freshness, length, redundancy, conflict, and access eligibility. Deduplicate by stable identity/version, retain source boundaries, budget tokens, and vary order deliberately.

Fusion-in-Decoder shows that a model can aggregate multiple retrieved passages in a scoped QA setup. Lost in the Middle and FreshLLMs show why “more context” is not a universal rule: evidence position, amount, and order can matter. Test depth/order matrices, relevant-plus-distractor mixtures, duplicate amplification, and truncation for the target model and prompt.

**Worked Example:** Hold candidates fixed while moving the only supporting passage from first to middle to last and adding distractors. A quality change isolates placement sensitivity more directly than changing retrieval and order together.

**Knowledge Check:** Why can rewriting improve recall while corrupting an identifier or negation?

**Guided Practice:** Cross original/rewrite/decompose with depth, order, duplicates, distractors, and token budget; preserve candidate/context lineage.

**Feedback Contract:** Expected evidence is a paired phase surface at matched budgets, including intent fidelity, opportunity, use/support, latency, and cost. A common failure is changing multiple stages without checkpoints.

**Learning Outcome:** Demonstrate which query and packing policy improves answer/citation quality at matched latency and cost, including slices where it fails.

*(Effort: 45m instruction, 30m practice)*

### Lesson 10.4 — Contradiction Is a Data-and-Policy Problem

**Engineering Question:**
Which evidence applies when sources disagree across authority, scope, and time?

**Concepts & Definitions:**

Contradiction can occur among source documents, revisions of one source, retrieved context and model memory, or the answer and its cited passage. Normalize the contested claim before choosing a winner:

```text
(subject, predicate, object, valid_time, transaction/version,
 source_identity, authority, scope)
```

Two passages may be temporally different rather than contradictory. Ten mirrors of a stale article do not outrank one authoritative update. “Newest” is unsafe when publication time differs from effective time. Model confidence does not establish source authority.

Real-document experiments report that models can retain incorrect parametric answers despite corrective context. Therefore test both directions: trusted context correcting model memory, and untrusted context attempting to override a valid prior. If authority/time/scope cannot resolve the evidence, surface the conflict or abstain.

**Worked Example:** A policy published later can still have an earlier effective date; an article mirrored ten times does not become ten independent authorities. Compare valid time, transaction/index visibility time, identity, and authority before declaring contradiction.

**Knowledge Check:** When are two different values temporal versions rather than contradictions?

**Guided Practice:** Resolve a set containing stale mirrors, future-effective updates, scope-specific rules, and equal-authority conflict; preserve an unresolved state.

**Feedback Contract:** Expected evidence is a normalized claim table and policy trace. A common failure is majority vote by document count or “newest publication wins.”

**Learning Outcome:** Implement an auditable authority/temporal policy and a deliberate unresolved-conflict state.

*(Effort: 40m instruction, 25m practice)*

### Lesson 10.5 — Citations, Grounding, and Stage Attribution

**Engineering Question:**
Does each support-required claim bind to adequate evidence, and where was opportunity lost?

**Concepts & Definitions:**

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

**Worked Example:** An answer with four support-required claims and three adequate citations has completeness $3/4$; if one of those three citations does not entail its claim, citation correctness among three checked citations is $2/3$. State zero-denominator policy.

**Knowledge Check:** Why does a valid reference index not establish semantic support?

**Guided Practice:** Label claims and evidence spans, then replay with oracle availability, candidates, context, and citation binding to isolate the earliest failing stage.

**Feedback Contract:** Expected evidence is a claim-evidence table, evaluator/human agreement, stage trace, and uncertainty. A common failure is one aggregate “groundedness” score.

**Learning Outcome:** Produce a claim-evidence table and assign failures to the earliest discriminated stage.

*(Effort: 50m instruction, 30m practice)*

### Lesson 10.6 — RAG, Long Context, and Adaptive Routes

**Engineering Question:**
Which route maximizes declared utility while preserving hard evidence and access constraints?

**Concepts & Definitions:**

Compare architectures with the same corpus snapshot, eligible evidence, base model where possible, prompt objective, answer set, and load. Include retrieval/index work, input tokens, cached prefixes, latency tails, failures, and evaluator costs. A large advertised context window does not prove robust evidence use; retrieval may miss evidence that full context contains; full context may dilute or misorder evidence that retrieval isolates.

A useful engineering decision is constrained rather than ideological:

$$
U(route)=Q(route)-\lambda_L L(route)-\lambda_C C(route),
$$

subject to hard access, safety, freshness, and consistency constraints. This is a product-specific heuristic: quality definition and weights are not universal. Published comparisons through 2025 report different task-dependent outcomes, which is evidence against a universal winner.

Self-RAG, corrective RAG, and hybrid routing demonstrate adaptive mechanism families. They add evaluator/router errors, correlated self-judgment, extra model or search calls, new source-quality risks, latency, and cost. Treat them as workload-dependent/frontier until calibrated on the target service.

**Worked Example:** Compare routes on the same source snapshot and answer set. Retrieval misses weaken RAG; evidence dilution or position sensitivity can weaken full context. Include router calls, indexes, input tokens, cache state, failures, and load before selecting a frontier.

**Knowledge Check:** Why is advertised context length insufficient evidence for long-context quality?

**Guided Practice:** Run matched RAG, full-context, and hybrid routes under equal eligibility and load; test no-answer, noisy-long-document, temporal, and access slices.

**Feedback Contract:** Expected evidence is a matched frontier, route calibration/errors, full cost/latency, hard-constraint checks, fallback, and rollback. A common failure is comparing unequal corpora or models.

**Learning Outcome:** Build a measured route policy with fallback and a falsifiable decision log.

*(Effort: 45m instruction, 30m practice)*

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

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Incremental Index Visibility and Delete Convergence

- **Objective:** build a versioned change ledger and prove query-visible update/delete convergence across representations, replicas, and caches.
- **Pre-Registered Hypothesis:** worker acknowledgement will precede full read visibility in at least one injected delay/failure case; replay-safe identity/version rules will preserve convergence.
- **Independent Variables:** event order/duplication/delay/failure, representation, refresh, replica/cache state, and rollback.
- **Dependent Variables:** visibility distributions, stale/partial reads, version monotonicity, access violations, idempotence, and recovery time.
- **Break & Falsify:** replay upserts/deletes, lag caches/replicas, time out one branch, and verify fallback state is explicit.
- **Alignment:** Lessons 10.1 and 10.2.
- **Effort Estimate:** 4.5h.

### LAB B — Query and Context Phase Diagram

- **Objective:** compare query transformations and context packing at matched evidence and resource budgets.
- **Pre-Registered Hypothesis:** transformation/packing gains will vary by slice, with identifier/negation and distractor/order counterexamples.
- **Independent Variables:** query route, depth, order, duplicates, distractors, conflicts, compression, and token budget.
- **Dependent Variables:** intent fidelity, evidence opportunity, use/support, answer/citation quality, latency, tokens, and cost.
- **Break & Falsify:** test identifiers, negation, temporal, multi-hop, no-answer, middle-position, and noisy-context cases.
- **Alignment:** Lesson 10.3.
- **Effort Estimate:** 4.5h.

### LAB C — Contradiction and Citation Audit

- **Objective:** implement conflict policy and claim-level citation correctness/completeness with independent adjudication.
- **Pre-Registered Hypothesis:** document-count/newest-only policies and citation syntax will fail at least one authority/time/support case.
- **Independent Variables:** authority, valid/publication/transaction time, scope, mirrors, context order, reference binding, and evaluator.
- **Dependent Variables:** resolved/unresolved accuracy, citation correctness/completeness, evaluator agreement, severity, and abstention.
- **Break & Falsify:** use stale mirrors, future-effective rules, equal-authority conflicts, deletes, partial/multi-source support, and broken numeric references.
- **Alignment:** Lessons 10.4 and 10.5.
- **Effort Estimate:** 4.5h.

### LAB D — Matched RAG/Long-Context/Hybrid Routing

- **Objective:** compare RAG, long-context, and hybrid routing on identical evidence eligibility and loaded conditions.
- **Pre-Registered Hypothesis:** no route will dominate every prespecified slice after full work, freshness, failure, and SLO constraints are counted.
- **Independent Variables:** route, evidence length/order/noise, query slice, temporal state, router threshold, load, and cache.
- **Dependent Variables:** answer/support quality, tokens, full work, stage/end-to-end tails, cost, freshness, abstention, route error, and SLO-goodput.
- **Break & Falsify:** include answerable-without-context, shuffled/no evidence, temporal updates, long noisy documents, router shift, and bursts.
- **Alignment:** Lesson 10.6.
- **Effort Estimate:** 4.5h.

## 07 Break / Incident Scenarios

### Incident 10.1 — Fresh Answer Metric, Stale and Unsupported Production Answers

**Incident Symptoms:**
After an incremental-index release, average answer accuracy improves, but some tenants receive deleted policies, recent updates are absent for minutes, contradictory versions are blended, and citations open valid documents that do not support the attached claims.

**Diagnostic Protocol (Task):**
1. **Form Competing Hypotheses:** source polling lag, parse/embed failure, unrefreshed write, replica/alias/cache skew, retry reordering, incomplete tombstone fanout, rewrite drift, retrieval loss, context dilution, parametric bias, generator misuse, reference binding, or evaluator error.
2. **Identify Missing Evidence:** event/version logs, representation state/high-watermarks, route/cache/replica identity, original/transformed queries, candidates, exact context/order, prompt/model, claims/citations, timeout/fallback state, and stage timing.
3. **Design Discriminating Measurements:** probe each representation and replica, replay pinned snapshots, disable transformations/caches one at a time, and substitute oracle candidates/context/bindings.
4. **Rank Explanations:** locate the earliest available → visible → retrieved → selected → used → supported divergence and preserve interacting causes.
5. **Intervene:** gate/rollback the smallest supported stage, converge deletes/replicas/caches, repair transformation/context/binding, or abstain on unresolved partial state.
6. **Remeasure:** repeat freshness/stale/partial distributions, access/deletion invariants, claim support/citation completeness, latency, abstention, and user utility.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem — Versioned Evidence Platform

Design Advanced RAG for mutable technical and policy documentation with tenant ACLs, frequent updates/deletes, contradictory versions, citations, and a latency/cost SLO.

**Required Deliverables:**
1. request/read contract with eligible snapshot, transformations, budgets, and abstention;
2. incremental event ledger, idempotence, high-watermarks, cache/replica and update/delete convergence;
3. query-visible freshness and stale/partial/failure measurements;
4. query/context depth/order/noise ablations with complete lineage;
5. temporal/scope/authority contradiction policy and unresolved state;
6. claim-citation correctness/completeness evaluator calibrated against blinded adjudication;
7. matched RAG/long-context/hybrid frontier including all work and loaded failures;
8. current source trace with generalizability boundary;
9. canary, fallback, rollback, and incident diagnosis separating source, visibility, retrieval, selection, use, and support.

## 09 Required Evidence & Rubric

### Required Artifact: End-to-End Evidence Lineage Trace

Submit one replayable source-event-to-answer trace, visibility/deletion tests, query/context checkpoints, claim-evidence table, evaluator audit, matched architecture results, source trace, and release/rollback decision.

### Rubric Dimensions

- **Lineage:** *Insufficient* stores final prompt only. *Competent* preserves stable IDs/versions and transformations. *Strong* reproduces eligibility, order, claims, citations, caches, and failures.
- **Freshness:** *Insufficient* equates write success with visibility. *Competent* measures declared endpoints. *Strong* tests clocks, high-watermarks, retries, partial state, deletes, and rollback.
- **Context:** *Insufficient* changes top-k only. *Competent* measures opportunity/order/noise/truncation. *Strong* uses matched stage ablations and workload slices.
- **Contradiction/Citations:** *Insufficient* trusts newest or reference syntax. *Competent* defines time/authority and support. *Strong* preserves unresolved conflicts and calibrates correctness/completeness judgments.
- **Architecture:** *Insufficient* declares a universal winner. *Competent* matches evidence/model/objective. *Strong* includes all work, router error, load, freshness, and hard constraints.
- **Diagnosis:** *Insufficient* guesses. *Competent* follows the diagnostic sequence. *Strong* identifies earliest divergence, interacting causes, intervention, and remeasurement.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Versioned incremental RAG | 10.1–10.2 | LAB A | Incident / Mastery | Ledger, watermark, convergence tests |
| Query and context orchestration | 10.3 | LAB B | Mastery | Paired phase diagram and traces |
| Contradiction and citations | 10.4–10.5 | LAB C | Incident / Mastery | Claim-evidence audit and adjudication |
| RAG vs long context | 10.6 | LAB D | Mastery | Matched frontier and routing errors |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner passes when they can demonstrate query-visible freshness rather than ingestion success; survive duplicate/reordered updates, partial failure, cache/replica lag, and deletes; reproduce an answer from immutable lineage; identify rewrite and context-order failures; preserve unresolved contradictions; distinguish citation syntax, correctness, and completeness; trace a current implementation without generalizing it; and defend a workload-specific route under quality, latency, cost, freshness, access, and abstention constraints.

### Module Wrap-Up (Final Mental Model Reconstruction)

RAG is a versioned evidence-control loop. Preserve identity across every transformation, measure visibility at the query boundary, treat cache/fallback state as part of the read contract, treat context as constrained selection, keep contradictions explicit, require claim-level support, and choose architecture from matched evidence rather than fashion.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

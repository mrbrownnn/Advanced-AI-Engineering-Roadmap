# Module 09 — Retrieval Engineering

## 00 Why This Module Exists

Retrieval is not “vector search.” It is a staged decision system that converts a query into a bounded candidate set under corpus, relevance, filter, latency, and capacity constraints. A relevant document can be lost by analysis, representation, approximate search, filtering, fusion, truncation, or reranking; the same missing result can therefore have several competing causes.

$$
\text{query}\to\text{analyze/embed}\to\text{candidate branches}
\to\text{filter/fuse}\to\text{rerank}\to\text{top-}k.
$$

This module covers lexical and dense retrieval, exact and approximate nearest neighbors, HNSW, hybrid fusion, reranking, late interaction, metrics, and latency–quality trade-offs. Document ingestion and lineage belong to Module 08. Query rewriting, generation, citations, freshness orchestration, and RAG failure handling belong to Module 10.

**Research cutoff:** 2026-09-27.

**Module Orientation**

- **Engineering problem:** maximize useful relevance under filter correctness, tail latency, memory, update, and capacity constraints.
- **What you will do:** implement lexical and exact-dense baselines; quantify representation and approximation loss separately; tune HNSW under selective filters; build hybrid fusion and bounded reranking; measure judgment and service trade-offs; trace Lucene source; and diagnose a loaded retrieval regression.
- **Environment:** Python 3.10+ for experiment orchestration and analysis, plus a pinned search engine or ANN implementation and a versioned corpus/query/judgment set. Record index hardware, build/search parameters, concurrency, filters, and cache state.
- **Evidence rule:** separate source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**). Paper gains remain scoped to model, corpus, queries, judgments, index, hardware, and baseline.

## 01 Baseline Assumptions

- Module 00: experimental units, paired comparisons, uncertainty, and causal diagnosis.
- Module 02/04: performance measurement, latency distributions, queueing, throughput, and goodput.
- Module 08: immutable corpus/chunk lineage, validation, contamination, and release rollback.
- Mathematical prerequisites: vectors, dot product, cosine/L2 distance, logarithms, sets, ranks, and empirical distributions.

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
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 17h
  assessment: 3h
  source_trace: 2h
  total: 30h
```

The learner must be able to implement strong lexical/dense baselines, isolate encoder from ANN loss, tune HNSW under filters and load, construct hybrid fusion, rerank bounded candidates, select metrics from a relevance contract, and diagnose offline-to-production regressions.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
query + corpus snapshot + relevance/filter contract
        |                         |
        | lexical analyzer       | query encoder
        v                         v
   BM25 candidate list      vector candidate list
                                  |
                         exact kNN or ANN/HNSW
        \_________________________/
                    |
              identity + fusion
                    |
             reranker/late interaction
                    |
        ranked results + complete stage trace
```

Keep three loss boundaries separate:

1. **Representation/relevance loss:** even exact search ranks the wrong items.
2. **Approximation/filter loss:** ANN differs from exact search under the same vectors and filters.
3. **Selection/system loss:** fusion, reranking, truncation, latency, or capacity changes the delivered list.

## 04 Lessons

### Lesson 9.1 — Lexical Retrieval and BM25

**Engineering Question:**
Which lexical analysis and scoring choices make an exact term retrievable and reproducible?

**Concepts & Definitions:**

BM25 is a strong, interpretable baseline for exact identifiers, names, rare terms, and lexical intent. A common term contribution has the shape:

$$
IDF(t)\cdot\frac{f(t,d)(k_1+1)}
{f(t,d)+k_1(1-b+b|d|/avgdl)}.
$$

**Mechanism Explanation:**

The formula is incomplete without analyzer, token positions/overlaps, fields/boosts, corpus statistics, query construction, $k_1$, and $b$. Stemming, stopwords, Unicode normalization, synonyms, and index/query analyzer mismatch can dominate tuning.

**Break cases:** product codes split incorrectly; language-specific tokenization; a corpus refresh changes IDF; synonyms expand only one side; long templated documents receive unintended length penalties.

**Worked Example:**
Index a product code once as one token and once split on punctuation. The same BM25 parameters can return different candidates because the analyzer changed the term inventory and corpus statistics.

**Knowledge Check:** Why is $k_1,b$ tuning unable to repair a term removed by analysis?

**Guided Practice:** Reproduce one score from term frequency, document length, corpus statistics, fields, and analyzer output; then change one factor.

**Feedback Contract:** Expected evidence is a pinned analyzer/index manifest and score decomposition. A common failure is comparing BM25 parameters across different analyzers.

**Learning Outcome:** Reproduce scores from the pinned index contract and explain which input changed.

*(Effort: 40m instruction, 25m practice)*

### Lesson 9.2 — Dense Dual Encoders and Similarity Semantics

**Engineering Question:**
Did the representation fail, or did approximate search lose a useful vector result?

**Concepts & Definitions:**

Dense retrieval encodes query $q$ and document $d$ separately, then scores inner product, cosine, or distance. DPR establishes this mechanism for open-domain QA, not universal superiority over BM25. Training positives/negatives, encoder version, pooling, normalization, dimension, chunking, domain, and similarity must match index and query paths.

**Mechanism Explanation:**

Cosine and inner product are equivalent in ranking only under compatible unit normalization. L2 ordering has another relationship under unit vectors; do not switch metrics by name. First run exact vector search on a representative subset: if exact search misses relevance, increasing ANN effort cannot repair the encoder.

**Worked Example:**
Run exact search over the same normalized vectors and metric. If the relevant item is absent from exact top-$k$, raising HNSW search effort cannot repair that representation ranking.

**Knowledge Check:** Under which normalization condition do cosine and inner-product rankings coincide?

**Guided Practice:** Cross normalized/unnormalized embeddings with cosine, inner product, and L2; record ranking changes and invalid combinations.

**Feedback Contract:** Expected evidence is a pinned encoder/tokenizer/pooling/metric manifest plus exact ranks. A common failure is blaming ANN before exact comparison.

**Learning Outcome:** Distinguish semantic representation failure from index approximation.

*(Effort: 40m instruction, 25m practice)*

### Lesson 9.3 — Exact kNN, HNSW, and Filtered ANN

**Engineering Question:**
How do approximation, filters, updates, and graph-search budgets change delivered candidates?

**Concepts & Definitions:**

HNSW builds randomized hierarchical proximity graphs and searches from sparse upper layers toward a denser base. Construction connectivity/effort, search effort, vector distribution, distance, deletion/update policy, memory layout, and implementation produce a recall–latency–memory surface.

**Quantitative Model / Derivation:**

For exact top-$k$ set $E_k$ and approximate set $A_k$ under identical vectors, metric, filter, snapshot, and ties:

$$
Recall^{ANN}@k=\frac{|A_k\cap E_k|}{|E_k|}.
$$

This measures approximation, not relevance. A system can have perfect ANN recall and poor retrieval relevance.

Filters complicate search. Prefiltering, integrated graph filtering, oversampling then filtering, and exact fallback behave differently with selectivity and vector/filter correlation. Measure exact filtered ground truth, visited nodes/candidates, fallback, tail latency, and concurrency. ACL correctness is a hard invariant, not a recall trade.

Index lifecycle is part of correctness: insert, update, delete/tombstone, segment merge or graph rebuild, replica visibility, and rollback can temporarily diverge. Track query-visible revision and resource cost; “write accepted” does not establish that every serving replica uses the same candidate universe.

**Worked Example:**
If exact filtered top-10 contains ten eligible targets and ANN returns eight of those plus two other eligible items, ANN recall@10 is 0.8. This says nothing about relevance recall unless the exact set is itself judged relevant; returning an unauthorized item is a correctness failure regardless of 0.8.

**Knowledge Check:** Why must filter, vectors, metric, snapshot, and ties match when computing ANN recall?

**Guided Practice:** Sweep search effort and filter selectivity/correlation, then inject update/delete and replica lag. Compare against exact filtered ground truth.

**Feedback Contract:** Expected evidence includes exact/ANN sets, visited work, tail latency, memory, revision, fallback, and zero ACL violations. A common failure is calling ANN recall relevance recall.

**Learning Outcome:** Tune ANN against exact search and break it with selective, correlated filters and lifecycle churn.

*(Effort: 55m instruction, 35m practice)*

### Lesson 9.4 — Hybrid Retrieval and Rank Fusion

**Engineering Question:**
Do retrieval branches add complementary opportunity, and does fusion preserve it?

**Concepts & Definitions:**

Lexical and dense branches can be complementary: one captures explicit terms, the other learned semantic similarity. They can also be redundant. Normalize document identity before union so aliases/chunks do not fragment votes.

**Quantitative Model / Derivation:**

Reciprocal rank fusion uses:

$$
RRF(d)=\sum_r\frac{w_r}{c+rank_r(d)}.
$$

It avoids treating incomparable raw score scales as comparable, but $c$, weights, list depths, missing items, and duplicate policy remain choices. Fusion cannot recover an item missing from all branches. Compare each branch, union oracle opportunity, fusion, and latency at equal budgets.

**Worked Example:**
If lexical and dense each retrieve six of ten relevant items but their union contains nine, the union exposes complementarity. A fusion result below that opportunity can then be attributed to identity, depth, weighting, or rank aggregation.

**Knowledge Check:** Can fusion recover an item absent from every branch?

**Guided Practice:** Compare branch results, normalized-identity union opportunity, RRF, and a single-branch baseline at matched depth and latency budget.

**Feedback Contract:** Expected evidence is branch lineage, duplicate policy, union oracle, fused ranks, and critical-path/total work. A common failure is adding incomparable raw scores.

**Learning Outcome:** Prove complementarity rather than assume “hybrid is better.”

*(Effort: 40m instruction, 25m practice)*

### Lesson 9.5 — Reranking and Late Interaction

**Engineering Question:**
How much candidate opportunity should be purchased before reranking cost and truncation dominate?

**Concepts & Definitions:**

A cross-encoder jointly scores query–document pairs and can reorder only the candidate pool it receives. Its cost grows with candidate count and sequence shapes; batching and hardware change the actual curve. Truncation can remove the only relevant span.

**Mechanism Explanation:**

ColBERT-style late interaction precomputes contextual document-token vectors and performs finer query-token matching than a single-vector dual encoder. It occupies a different storage/compute point; paper speedups do not transfer without index, hardware, and corpus context.

For each depth, record candidate recall before reranking, reranker nDCG/MRR after reranking, truncation, model/batch time, queue time, and total work. If candidate opportunity saturates early but reranking cost rises, deeper is not automatically better.

**Worked Example:**
If candidate recall saturates by depth 50 but P99 rerank latency rises through depth 200 with no judged gain, the deeper setting is dominated for that workload. A different corpus or batch shape requires remeasurement.

**Knowledge Check:** Why can a perfect reranker not recover a relevant document missing from its candidate pool?

**Guided Practice:** Sweep candidate depth, truncation, model, and batch size; record pre-rerank opportunity, post-rerank quality, queue/compute time, and memory.

**Feedback Contract:** Expected evidence is a depth frontier with identical queries/candidates and full cost. A common failure is reporting reranker-only latency.

**Learning Outcome:** Select candidate and rerank depth jointly under capacity and latency constraints.

*(Effort: 40m instruction, 25m practice)*

### Lesson 9.6 — Metrics, Judgments, and the Quality–Latency Frontier

**Engineering Question:**
Which relevance and service measurements support a production retrieval decision?

**Concepts & Definitions:**

- $Recall@k=|Rel\cap Top_k|/|Rel|$ measures known relevant-set coverage.
- $RR=1/r$ uses the first relevant rank $r$; MRR averages queries.
- $DCG@k=\sum_{i=1}^k g(rel_i)/\log_2(i+1)$; nDCG divides by ideal DCG.

**Mechanism Explanation:**

Declare binary/graded relevance, cutoff, gain, discount, query unit, ties, duplicates, zero-relevant queries, and unjudged policy. Incomplete pools can favor systems similar to the pooling methods. BEIR's heterogeneous results are evidence against a universal retriever ranking.

End-to-end latency includes query processing/embedding, branch searches, filters, fusion, fetch, reranking, network, and queueing. Parallel branch latency follows the join critical path; both branches consume capacity. Report per-stage and end-to-end distributions, quality under load, timeout/cancellation, and SLO-goodput.

**Worked Example:**
Two systems can have the same nDCG while one times out on selective-filter queries. Count timed-out, cancelled, and incomplete requests in the declared offered/admitted population rather than computing quality only on successes.

**Knowledge Check:** Why can incomplete judgment pools favor systems similar to the pool constructors?

**Guided Practice:** Evaluate a stable query set at increasing open-loop load. Join judgments, stage traces, timeouts, and resource use; report quality/latency/goodput by slice.

**Feedback Contract:** Expected evidence is a pinned judgment policy, unjudged handling, uncertainty, stage and end-to-end tails, and complete denominators. A common failure is selecting on one average metric.

**Learning Outcome:** Choose a Pareto point rather than a metric-only winner.

*(Effort: 50m instruction, 30m practice)*

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [The Probabilistic Relevance Framework: BM25 and Beyond](https://www.staff.city.ac.uk/~sbrp622/papers/foundations_bm25_review.pdf) — Robertson and Zaragoza (2009).
- [Efficient and robust approximate nearest neighbor search using HNSW graphs](https://arxiv.org/abs/1603.09320) — Malkov and Yashunin (2018).
- [Dense Passage Retrieval](https://arxiv.org/abs/2004.04906) — Karpukhin et al. (EMNLP 2020).
- [Reciprocal Rank Fusion](https://dl.acm.org/doi/10.1145/1571941.1572114) — Cormack, Clarke, and Büttcher (SIGIR 2009).
- [ColBERT](https://arxiv.org/abs/2004.12832) — Khattab and Zaharia (SIGIR 2020).
- [BEIR](https://arxiv.org/abs/2104.08663) — Thakur et al. (2021).

**CURRENT DEFAULT:** a measured BM25 baseline, embedding-version pinning, ANN-versus-exact audit, hybrid candidate lineage where justified, bounded reranking, slice metrics, and end-to-end timing.

**WORKLOAD-DEPENDENT:** analyzer, dense model, distance, HNSW parameters, filter strategy, fusion method, reranker, depths, and metric weights.

**FRONTIER:** learned sparse retrieval, multi-vector compression, LLM reranking/query representations, and 2025–2026 retrieval agents. None is a universal replacement for workload-grounded baselines.

**LEGACY / INSUFFICIENT:** dense-only by default; ANN recall called retrieval recall; a single average metric; unpinned embeddings; raw lexical/vector score addition without calibration; microbenchmark latency used as service latency.

**PRODUCTION SOURCE TRACE**

- Repository: `apache/lucene`
- Revision: `e357029271b3de560a6ff13c9cab4d4c8103b53b`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `BM25Similarity.idf/scorer`, `KnnFloatVectorQuery.approximateSearch`, and `HnswGraphSearcher.search/searchLevel` in the registry-recorded paths.
- Scope: pinned Lucene behavior, not a universal BM25/HNSW or filtered-search definition.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Lexical vs Exact Dense Retrieval

- **Objective:** build versioned BM25 and exact-dense baselines and isolate analysis/representation failures.
- **Pre-Registered Hypothesis:** lexical and dense branches will show complementary strengths on prespecified identifier and semantic slices, not universal dominance.
- **Independent Variables:** analyzer/BM25 parameters, encoder/normalization/metric, corpus snapshot, and query slice.
- **Dependent Variables:** raw ranks, Recall/MRR/nDCG by slice, overlap/opportunity, latency, and errors.
- **Break & Falsify:** use IDs, misspellings, multilingual terms, paraphrases, corpus-stat changes, and analyzer mismatch.
- **Alignment:** Lessons 9.1 and 9.2.
- **Effort Estimate:** 4h.

### LAB B — HNSW and Filter Phase Diagram

- **Objective:** map exact-to-ANN loss under build/search, filter, load, and lifecycle conditions.
- **Pre-Registered Hypothesis:** higher search effort will improve ANN recall in the tested range but latency/memory/update cost and selective filters will prevent a universal optimum.
- **Independent Variables:** build/search parameters, size, concurrency, filter selectivity/correlation, and update/delete churn.
- **Dependent Variables:** ANN and relevant recall@k, visited work, fallback, build/update cost, memory, stage/tail latency, revision convergence, and ACL violations.
- **Break & Falsify:** inject deletion churn, rare ACL groups, correlated metadata, replica lag, and adversarial filter selectivity.
- **Alignment:** Lesson 9.3.
- **Effort Estimate:** 4.5h.

### LAB C — Hybrid Fusion and Rerank Depth

- **Objective:** compare branch opportunity, fusion, and bounded reranking on a quality–latency–capacity frontier.
- **Pre-Registered Hypothesis:** fusion/reranking will help only where branch complementarity and candidate opportunity justify their extra work.
- **Independent Variables:** branch depth, RRF weights/constant, candidate depth, truncation, model, and batch size.
- **Dependent Variables:** union opportunity, duplicate fragmentation, pre/post-rerank metrics, total work, tail latency, memory, and SLO-goodput.
- **Break & Falsify:** use redundant branches, missing-from-all items, truncated relevant spans, and loaded queues.
- **Alignment:** Lessons 9.4 and 9.5.
- **Effort Estimate:** 4.5h.

### LAB D — Offline-to-Loaded Retrieval Release

- **Objective:** evaluate a candidate index across representative slices and production-like open-loop load before release.
- **Pre-Registered Hypothesis:** at least one offline winner will lose its advantage or violate a guardrail after filter, freshness, or loaded-tail effects are included.
- **Independent Variables:** index release, slice, offered load, filter, cache, update state, timeout, and concurrency.
- **Dependent Variables:** IR metrics with uncertainty, exact/ANN gap, freshness, stage/end-to-end tails, queueing, failures, capacity, and goodput.
- **Break & Falsify:** include no-relevant queries, bursts, timeouts, cancellations, stale replicas, and changed traffic mix.
- **Alignment:** Lesson 9.6 and Incident 09.1.
- **Effort Estimate:** 4h.

## 07 Break / Incident Scenarios

### Incident 09.1 — Better nDCG, Missing Authorized Results

**Incident Symptoms:**
A release changes the encoder, HNSW parameters, hybrid weights, and reranker. Offline average nDCG rises, but selective tenant queries miss recent authorized documents, P99 rises, exact identifiers regress, and one replica reports an older index revision.

**Diagnostic Protocol (Task):**
1. **Form Competing Hypotheses:** corpus/index lag, analyzer mismatch, encoder shift, ANN/filter interaction, lifecycle/replica divergence, identity fusion bug, reranker truncation, deeper candidate cost, ACL error, judgment bias, or traffic change.
2. **Identify Missing Evidence:** query/corpus/judgment and component manifests, replica revisions, analyzer tokens, exact and ANN candidates, filter/fallback work, fusion identity, reranker input/output, queue/load, timeout, and authorization decisions.
3. **Design Discriminating Measurements:** replay matched queries through lexical, exact dense, filtered ANN, fusion, and reranker checkpoints; cross old/new one factor at a time and probe every replica.
4. **Rank Explanations:** use earliest candidate divergence, matched deltas, stage timing, and authorization invariants; do not infer one cause from aggregate nDCG.
5. **Intervene:** roll back or gate the discriminated component, converge the index, repair analysis/filter/identity/truncation, or reduce loaded depth as evidence supports.
6. **Remeasure:** repeat slice quality, exact–ANN gap, freshness, authorization, stage/tail latency, timeout, capacity, and canary rollback checks.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem — Multilingual Tenant-Aware Technical Search

Design retrieval for multilingual technical search with exact identifiers, semantic questions, tenant ACLs, updates, and a latency SLO.

**Required Deliverables:**
1. relevance/filter contract and versioned corpus, embedding, and index manifests;
2. BM25 and exact-dense baselines with analyzer/similarity semantics;
3. exact-versus-ANN/filter phase map including lifecycle churn and memory/capacity;
4. branch complementarity, identity-normalized fusion, and rerank-depth frontier;
5. metric/judgment/unjudged protocol with uncertainty and slices;
6. stage and end-to-end loaded tests with timeouts, failures, and goodput;
7. pinned production source trace and scoped generalization;
8. authorization/update/delete canaries and rollback;
9. an incident diagnosis separating representation, approximation, selection, lifecycle, and system loss.

## 09 Required Evidence & Rubric

### Required Artifact: Retrieval Stage Trace and Release Report

Submit manifests, raw branch/stage rankings, exact–ANN/filter comparisons, judgment and uncertainty records, loaded latency/capacity results, lifecycle probes, source trace, and canary/rollback decision.

### Rubric Dimensions

- **Mechanism:** *Insufficient* says “vector search.” *Competent* traces analyzer, encoder, metric, ANN, filter, fusion, and reranker. *Strong* links candidate losses and lifecycle state to observable stages.
- **Mathematics:** *Insufficient* quotes formulas. *Competent* states assumptions for BM25, ANN recall, RRF, and IR metrics. *Strong* validates predictions against raw ranks and counterexamples.
- **Evaluation:** *Insufficient* reports one mean. *Competent* pins population/judgments/slices. *Strong* audits incomplete judgments, uncertainty, temporal and authorization cases.
- **Performance:** *Insufficient* gives microbenchmark averages. *Competent* measures stage/tails/load. *Strong* separates critical path, total work, memory, queueing, failures, and goodput.
- **Diagnosis:** *Insufficient* guesses. *Competent* uses exact search and checkpoints. *Strong* performs matched one-factor replays and ranks interacting causes.
- **Operations:** *Insufficient* omits updates and ACLs. *Competent* tests them. *Strong* proves replica/delete convergence, canary, and rollback.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| BM25 and dense baselines | 9.1–9.2 | LAB A | Mastery | Manifests, raw ranks, slice report |
| HNSW and filters | 9.3 | LAB B | Incident / Mastery | Exact-ANN/filter phase map |
| Hybrid and reranking | 9.4–9.5 | LAB C | Mastery | Candidate lineage and frontier |
| Metrics and system trade-offs | 9.6 | LAB D | Incident / Mastery | Judgments and loaded results |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner passes when they can reproduce BM25 behavior, validate embedding/metric semantics, isolate exact-versus-ANN loss, break filtered HNSW, verify update/delete visibility, justify fusion by complementarity, prove reranker candidate bounds, choose correct metrics, trace current source, and defend a quality–latency–capacity release with rollback.

### Module Wrap-Up (Final Mental Model Reconstruction)

Retrieval is a chain of lossy transformations. Instrument every candidate and lifecycle boundary, compare approximation with exact search, compare rankings with relevance judgments, and accept complexity only where it improves the declared production frontier.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

# Module 09 — Retrieval Engineering

## 00 Why This Module Exists

Retrieval is not “vector search.” It is a staged decision system that converts a query into a bounded candidate set under corpus, relevance, filter, latency, and capacity constraints. A relevant document can be lost by analysis, representation, approximate search, filtering, fusion, truncation, or reranking; the same missing result can therefore have several competing causes.

$$
\text{query}\to\text{analyze/embed}\to\text{candidate branches}
\to\text{filter/fuse}\to\text{rerank}\to\text{top-}k.
$$

This module covers lexical and dense retrieval, exact and approximate nearest neighbors, HNSW, hybrid fusion, reranking, late interaction, metrics, and latency–quality trade-offs. Document ingestion and lineage belong to Module 08. Query rewriting, generation, citations, freshness orchestration, and RAG failure handling belong to Module 10.

**Research cutoff:** 2026-09-26.

- **Engineering problem:** maximize useful relevance under filter correctness, tail latency, memory, update, and capacity constraints.
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
  falsification: REQUIRED
```

The learner must be able to implement strong lexical/dense baselines, isolate encoder from ANN loss, tune HNSW under filters and load, construct hybrid fusion, rerank bounded candidates, select metrics from a relevance contract, and diagnose offline-to-production regressions.

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

BM25 is a strong, interpretable baseline for exact identifiers, names, rare terms, and lexical intent. A common term contribution has the shape:

$$
IDF(t)\cdot\frac{f(t,d)(k_1+1)}
{f(t,d)+k_1(1-b+b|d|/avgdl)}.
$$

The formula is incomplete without analyzer, token positions/overlaps, fields/boosts, corpus statistics, query construction, $k_1$, and $b$. Stemming, stopwords, Unicode normalization, synonyms, and index/query analyzer mismatch can dominate tuning.

**Break cases:** product codes split incorrectly; language-specific tokenization; a corpus refresh changes IDF; synonyms expand only one side; long templated documents receive unintended length penalties.

**Outcome:** reproduce scores from the pinned index contract and explain which input changed.

### Lesson 9.2 — Dense Dual Encoders and Similarity Semantics

Dense retrieval encodes query $q$ and document $d$ separately, then scores inner product, cosine, or distance. DPR establishes this mechanism for open-domain QA, not universal superiority over BM25. Training positives/negatives, encoder version, pooling, normalization, dimension, chunking, domain, and similarity must match index and query paths.

Cosine and inner product are equivalent in ranking only under compatible unit normalization. L2 ordering has another relationship under unit vectors; do not switch metrics by name. First run exact vector search on a representative subset: if exact search misses relevance, increasing ANN effort cannot repair the encoder.

**Outcome:** distinguish semantic representation failure from index approximation.

### Lesson 9.3 — Exact kNN, HNSW, and Filtered ANN

HNSW builds randomized hierarchical proximity graphs and searches from sparse upper layers toward a denser base. Construction connectivity/effort, search effort, vector distribution, distance, deletion/update policy, memory layout, and implementation produce a recall–latency–memory surface.

For exact top-$k$ set $E_k$ and approximate set $A_k$ under identical vectors, metric, filter, snapshot, and ties:

$$
Recall^{ANN}@k=\frac{|A_k\cap E_k|}{|E_k|}.
$$

This measures approximation, not relevance. A system can have perfect ANN recall and poor retrieval relevance.

Filters complicate search. Prefiltering, integrated graph filtering, oversampling then filtering, and exact fallback behave differently with selectivity and vector/filter correlation. Measure exact filtered ground truth, visited nodes/candidates, fallback, tail latency, and concurrency. ACL correctness is a hard invariant, not a recall trade.

**Outcome:** tune ANN against exact search and break it with selective, correlated filters.

### Lesson 9.4 — Hybrid Retrieval and Rank Fusion

Lexical and dense branches can be complementary: one captures explicit terms, the other learned semantic similarity. They can also be redundant. Normalize document identity before union so aliases/chunks do not fragment votes.

Reciprocal rank fusion uses:

$$
RRF(d)=\sum_r\frac{w_r}{c+rank_r(d)}.
$$

It avoids treating incomparable raw score scales as comparable, but $c$, weights, list depths, missing items, and duplicate policy remain choices. Fusion cannot recover an item missing from all branches. Compare each branch, union oracle opportunity, fusion, and latency at equal budgets.

**Outcome:** prove complementarity rather than assume “hybrid is better.”

### Lesson 9.5 — Reranking and Late Interaction

A cross-encoder jointly scores query–document pairs and can reorder only the candidate pool it receives. Its cost grows with candidate count and sequence shapes; batching and hardware change the actual curve. Truncation can remove the only relevant span.

ColBERT-style late interaction precomputes contextual document-token vectors and performs finer query-token matching than a single-vector dual encoder. It occupies a different storage/compute point; paper speedups do not transfer without index, hardware, and corpus context.

For each depth, record candidate recall before reranking, reranker nDCG/MRR after reranking, truncation, model/batch time, queue time, and total work. If candidate opportunity saturates early but reranking cost rises, deeper is not automatically better.

**Outcome:** select candidate and rerank depth jointly under capacity and latency constraints.

### Lesson 9.6 — Metrics, Judgments, and the Quality–Latency Frontier

- $Recall@k=|Rel\cap Top_k|/|Rel|$ measures known relevant-set coverage.
- $RR=1/r$ uses the first relevant rank $r$; MRR averages queries.
- $DCG@k=\sum_{i=1}^k g(rel_i)/\log_2(i+1)$; nDCG divides by ideal DCG.

Declare binary/graded relevance, cutoff, gain, discount, query unit, ties, duplicates, zero-relevant queries, and unjudged policy. Incomplete pools can favor systems similar to the pooling methods. BEIR's heterogeneous results are evidence against a universal retriever ranking.

End-to-end latency includes query processing/embedding, branch searches, filters, fusion, fetch, reranking, network, and queueing. Parallel branch latency follows the join critical path; both branches consume capacity. Report per-stage and end-to-end distributions, quality under load, timeout/cancellation, and SLO-goodput.

**Outcome:** choose a Pareto point rather than a metric-only winner.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Lexical vs Exact Dense Retrieval

- Build a versioned corpus/query/judgment set and strong BM25 plus exact dense baselines.
- Sweep analyzers, BM25 parameters, encoder/normalization/metric, and query slices.
- Break with IDs, misspellings, multilingual terms, semantic paraphrases, corpus-stat changes, and analyzer mismatch.
- Artifact: index manifests, source trace, raw rankings, slice metrics, latency, and error taxonomy.

### LAB B — HNSW and Filter Phase Diagram

- Compare exact and approximate vector search across build/search parameters, dataset size, concurrency, and filter selectivity/correlation.
- Measure ANN recall@k, relevant recall@k, visited nodes, fallback, build/update cost, memory, stage latency, and tails.
- Inject deletion churn, rare ACL groups, and correlated metadata. Falsify any universal parameter recommendation.

### LAB C — Hybrid Fusion and Rerank Depth

- Compare lexical, dense, union opportunity, RRF/another justified fusion, cross-encoder, and optional late interaction.
- Sweep branch depth, weights/constant, candidate depth, truncation, and batch size.
- Measure complementarity, duplicate fragmentation, pre/post-rerank metrics, total work, tail latency, and SLO-goodput.

### LAB D — Offline-to-Loaded Retrieval Release

- Evaluate a candidate index on stable, temporal, language, identifier, semantic, filter, and no-relevant slices; then replay representative loaded traffic.
- Measure Recall/MRR/nDCG with judgment uncertainty, exact/ANN gap, freshness, stage distributions, queueing, timeout, and capacity.
- Produce canary, rollback, and missing-evidence decisions rather than a universal winner.

## 07 Break / Incident Scenarios

### Incident 09.1 — Better nDCG, Missing Authorized Results

A release changes the encoder, HNSW parameters, hybrid weights, and reranker. Offline average nDCG rises, but selective tenant queries miss recent authorized documents, P99 rises, and exact identifiers regress.

Competing causes include corpus/index lag, analyzer mismatch, encoder shift, ANN/filter interaction, identity fusion bugs, reranker truncation, deeper candidate cost, ACL error, judgment bias, or traffic change. Recover component/index manifests and stage traces; replay matched queries through lexical, exact dense, ANN, fusion, and reranker checkpoints; compare old/new one factor at a time; audit authorization as a hard invariant; intervene and remeasure identical boundaries.

## 08 Mastery Assessment

Design retrieval for multilingual technical search with exact identifiers, semantic questions, tenant ACLs, updates, and a latency SLO. Deliver relevance contract, corpus/index manifests, BM25 and exact-dense baselines, ANN/filter phase map, hybrid complementarity, rerank frontier, metric/judgment protocol, source trace, load test, canary/rollback, and an incident diagnosis that separates representation, approximation, selection, and system loss.

## 09 Required Evidence & Rubric

- **Mechanism:** analyzer, encoder, distance, ANN, filter, fusion, and reranker paths are explicit.
- **Mathematics:** BM25, ANN recall, RRF, IR metrics, and latency boundaries state assumptions.
- **Evaluation:** query population, judgments, unjudged policy, slices, uncertainty, and corpus snapshot are pinned.
- **Performance:** stage/critical-path latency, total work, memory, concurrency, tails, and goodput are measured.
- **Diagnosis:** exact search, branch ablations, candidate opportunity, and matched replays discriminate causes.
- **Operations:** authorization, update/deletion, canary, and rollback are first-class.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| BM25 and dense baselines | 9.1–9.2 | LAB A | Mastery | Manifests, raw ranks, slice report |
| HNSW and filters | 9.3 | LAB B | Incident / Mastery | Exact-ANN/filter phase map |
| Hybrid and reranking | 9.4–9.5 | LAB C | Mastery | Candidate lineage and frontier |
| Metrics and system trade-offs | 9.6 | LAB D | Incident / Mastery | Judgments and loaded results |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to reproduce BM25 behavior, validate embedding/metric semantics, isolate exact-versus-ANN loss, break filtered HNSW, justify fusion by complementarity, prove reranker candidate bounds, choose correct metrics, trace current source, and defend a quality–latency–capacity release with rollback.

**Final mental model:** retrieval is a chain of lossy transformations. Instrument every candidate boundary, compare approximation with exact search, compare rankings with relevance judgments, and accept complexity only where it improves the declared production frontier.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

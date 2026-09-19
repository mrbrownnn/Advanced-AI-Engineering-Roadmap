# Module 08 — Retrieval Engineering

## Why This Module Exists

This is NOT an introductory embeddings or vector database tutorial. This module covers the engineering of retrieval systems at depth: index structures, hybrid retrieval, reranking architectures, and the latency-memory-quality trade-off space. The goal is to make engineering decisions about retrieval, not to call an API.

## Key Engineering Questions

- What are the recall-latency-memory trade-offs of different index structures?
- When does hybrid retrieval (BM25 + dense) outperform either alone?
- What reranking architecture is appropriate for my latency budget?
- How do I measure retrieval quality correctly (Recall@K, MRR, NDCG)?
- How do I diagnose retrieval failures: was the document not indexed, not retrieved, or not ranked high enough?

## Prerequisites

- Module 00 (measurement methodology)
- Module 02 (latency measurement)

## Topics

### Sparse Retrieval
- BM25: term frequency, inverse document frequency, scoring
- Inverted index: structure, construction, query processing

### Dense Retrieval — Index Structures
- HNSW: multi-layer graph, construction parameters, recall-latency trade-offs
- IVF: inverted file index, coarse quantization, nprobe
- PQ: product quantization, codebook design, memory-quality trade-offs
- Combinations: IVF-PQ, HNSW with PQ compression

### Hybrid Retrieval
- Combining sparse and dense scores
- RRF (Reciprocal Rank Fusion): merging ranked lists
- Learned hybrid scoring

### Reranking
- Cross encoders: high quality, high latency
- Late interaction (ColBERT): token-level similarity with precomputed representations
- Adaptive retrieval: deciding retrieval depth dynamically
- Query decomposition: breaking complex queries into sub-queries

### Evaluation
- Recall@K: what fraction of relevant documents are in the top K?
- MRR (Mean Reciprocal Rank): how high is the first relevant result?
- NDCG (Normalized Discounted Cumulative Gain): quality of the full ranking
- Latency-memory-quality trade-off visualization

## Expected Artifacts

1. **Index comparison** — benchmark HNSW vs IVF-PQ for recall, latency, and memory on a real dataset
2. **Hybrid retrieval experiment** — measure when BM25 + dense outperforms either alone
3. **Reranking pipeline** — build and evaluate a retrieve-then-rerank pipeline
4. **Engineering report** — retrieval architecture recommendation for a specific use case

## Exit Criteria

The learner can:
- Select an index structure based on dataset size, latency budget, and memory constraints
- Design and evaluate a hybrid retrieval pipeline
- Choose a reranking strategy appropriate for the latency budget
- Correctly compute and interpret Recall@K, MRR, and NDCG
- Diagnose retrieval failures at the index, retrieval, and ranking stages

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

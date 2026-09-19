# Module 08 — Retrieval Engineering

## Why This Module Exists

This is NOT an introductory embeddings or vector database tutorial. This module covers the engineering of retrieval systems at depth: index structures, hybrid retrieval, reranking architectures, and the latency-memory-quality trade-off space. The goal is to make engineering decisions about retrieval, not to call an API.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal HNSW (Hierarchical Navigable Small World) graph and a simple Inverted File (IVF) index from scratch. Do not use FAISS or Qdrant.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Calculate the memory footprint of the index vs the raw vectors. Measure Recall@K and p99 query latency.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Construct an adversarial dataset containing clustered points that forces the IVF index to scan a massive number of vectors, breaking its latency bounds.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the failure by analyzing the distribution of vectors across IVF centroids. Formulate a hypothesis on why the clustering degraded performance.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement Product Quantization (PQ) to compress the vectors, or implement hybrid retrieval (BM25 + dense) to fix recall issues on rare keywords.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a production architecture choice: when to use HNSW (high memory, low latency) vs IVF-PQ (low memory, higher latency).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace the FAISS IVF-PQ implementation or Lucene's HNSW graph traversal.

## Expected Artifacts
- **Engineering Report**: Document the entire BUILD → MEASURE → BREAK → DEFEND loop with empirical evidence.
- **Implementation Code**: The scratch code demonstrating the mechanism.

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

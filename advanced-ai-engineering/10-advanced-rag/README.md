# Module 09 — Advanced RAG

## Why This Module Exists

Most RAG tutorials stop at "embed → retrieve → generate." Production RAG is an engineering problem: incremental indexing, stale content, embedding migration, contradiction handling, and knowing when NOT to retrieve. This module covers the full retrieval lifecycle.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement an incremental indexing system that can update, tombstone, and refresh documents without a full index rebuild.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the latency of index updates vs query performance as the index fragments.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Inject contradictory documents into the index. Observe the generative model hallucinating by conflating the contradictions.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the contradiction failure. Explain the limit of standard RAG when synthesizing opposing facts.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement multi-hop reasoning or query decomposition to explicitly verify retrieved facts against each other.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice between long-context models vs RAG for a specific application.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the index segment merging logic in an open-source vector database.

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

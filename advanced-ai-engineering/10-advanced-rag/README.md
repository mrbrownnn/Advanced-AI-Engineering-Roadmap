# Module 09 — Advanced RAG

## Why This Module Exists

Most RAG tutorials stop at "embed → retrieve → generate." Production RAG is an engineering problem: incremental indexing, stale content, embedding migration, contradiction handling, and knowing when NOT to retrieve. This module covers the full retrieval lifecycle.

## Key Engineering Questions

- How do I handle incremental updates without full re-indexing?
- How do I detect and manage stale embeddings?
- How do I migrate between embedding models without downtime?
- How do I handle contradictions between retrieved documents?
- When should the system NOT retrieve (RAG vs long context)?

## Prerequisites

- Module 08 (retrieval fundamentals, evaluation metrics)
- Module 07 (data lifecycle, freshness, drift)

## Topics

### Index Lifecycle
- Incremental indexing: adding documents without full rebuild
- Deletion and tombstones: handling removed content
- Freshness: detecting and updating stale content
- ACL changes: handling permission changes on indexed content

### Embedding Lifecycle
- Stale embeddings: when the embedding model changes
- Embedding migration: strategies for transitioning between models
- Dual-index rollout: running old and new indexes in parallel during migration

### Advanced Retrieval Patterns
- Contextual retrieval: enriching chunks with document-level context before embedding
- Iterative retrieval: multi-hop retrieval for complex queries
- Query decomposition and fusion for multi-part questions

### Quality and Reliability
- Contradiction handling: what to do when retrieved documents disagree
- Provenance and attribution: tracing answers back to source documents
- Ablation: measuring the contribution of retrieval to answer quality
- RAG vs long context: when to put everything in context vs when to retrieve
- When NOT to retrieve: queries the model can answer reliably from parameters

## Expected Artifacts

1. **Incremental indexing experiment** — implement and test incremental index updates
2. **Freshness analysis** — measure the impact of stale content on retrieval quality
3. **RAG vs long context comparison** — when does each approach win?
4. **Engineering report** — retrieval lifecycle strategy for a production system

## Exit Criteria

The learner can:
- Design an incremental indexing strategy with freshness management
- Plan an embedding model migration with minimal quality disruption
- Handle contradictions in retrieved content with traceable attribution
- Make evidence-based decisions about RAG vs long context vs no retrieval

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational → Extended Abstract
  dreyfus: Competent
```

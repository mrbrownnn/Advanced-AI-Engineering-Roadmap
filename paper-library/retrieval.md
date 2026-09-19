# Retrieval — Paper Index

## Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs

Authors: Malkov, Yashunin
Year: 2020
URL: https://arxiv.org/abs/1603.09320
Category: Approximate Nearest Neighbor Search
Priority: MUST

Why read:
HNSW is the dominant ANN index structure in production vector databases. Understanding its multi-layer navigable small world graph structure is essential for reasoning about recall-latency trade-offs, memory consumption, and index build costs.

Read specifically:
- Section 3 — the multi-layer graph construction
- Search algorithm
- Analysis of construction parameters (M, efConstruction) and their impact on recall and speed

Questions:
1. How does the hierarchical structure speed up search compared to a flat graph?
2. What are the memory costs of HNSW? How do they scale with dataset size?
3. How do construction parameters (M, efConstruction) affect recall vs latency vs memory?

Related module: 08-retrieval-engineering

---

## ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT

Authors: Khattab, Zaharia
Year: 2020
URL: https://arxiv.org/abs/2004.12832
Category: Retrieval, Late Interaction
Priority: MUST

Why read:
Introduces late interaction: computing token-level similarity between queries and passages rather than single-vector similarity. Achieves much higher quality than bi-encoder retrieval while remaining more efficient than cross-encoder reranking. Key architecture for production retrieval systems.

Read specifically:
- Section 3 — the MaxSim late interaction mechanism
- How token-level representations are precomputed and stored
- Quality vs latency comparison with bi-encoders and cross-encoders

Questions:
1. What is the quality advantage of late interaction over single-vector similarity?
2. What is the storage overhead of storing per-token embeddings?
3. When would you choose ColBERT over a bi-encoder + cross-encoder reranking pipeline?

Related module: 08-retrieval-engineering

---

## Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

Authors: Lewis, Perez, Piktus, Petroni, Karpukhin, Goyal, Küttler, Lewis, Yih, Rocktäschel, Riedel, Kiela
Year: 2020
URL: https://arxiv.org/abs/2005.11401
Category: RAG
Priority: MUST

Why read:
The original RAG paper. Establishes the paradigm of combining retrieval with generation. Required context for understanding the entire retrieval-augmented generation literature and for reasoning about when RAG helps vs hurts.

Read specifically:
- Section 3 — RAG-Sequence and RAG-Token models
- How retrieval is integrated into the generation process
- Comparison with closed-book generation

Questions:
1. What is the difference between RAG-Sequence and RAG-Token?
2. When does retrieval-augmented generation outperform closed-book generation?
3. What are the failure modes of RAG?

Related module: 08-retrieval-engineering, 09-advanced-rag

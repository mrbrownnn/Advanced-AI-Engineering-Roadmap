# KV Cache — Paper Index

## Efficient Memory Management for Large Language Model Serving with PagedAttention

Authors: Kwon, Li, Zhuang, Sheng, Zheng, Yu, Gonzalez, Zhang, Stoica
Year: 2023
URL: https://arxiv.org/abs/2309.06180
Category: KV Cache, Serving
Priority: MUST

Why read:
Introduces PagedAttention and the vLLM system. The core insight — applying OS virtual memory concepts (paging, block tables, copy-on-write) to KV cache management — is foundational to modern inference systems. This paper changed how production serving systems manage memory.

Read specifically:
- Section 3 — KV cache memory waste analysis
- Section 4 — PagedAttention design (block tables, non-contiguous storage)
- Section 4.3 — copy-on-write for parallel sampling
- Figure 3 — block table mapping
- Section 5 — scheduling and preemption policies

Questions:
1. What are the three types of KV cache memory waste the paper identifies?
2. How do block tables enable non-contiguous KV storage? What is the analogy to OS page tables?
3. How does copy-on-write reduce memory for parallel sampling (e.g., beam search)?
4. What is the preemption policy when memory is exhausted? What are the trade-offs?

Related module: 03-kv-cache-engineering, 04-serving-scheduling-capacity

Related modules: 03-kv-cache-engineering

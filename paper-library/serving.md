# Serving — Paper Index

## Orca: A Distributed Serving System for Transformer-Based Generative Models

Authors: Yu, Jeong, Shin, Chun
Year: 2022
URL: https://www.usenix.org/conference/osdi22/presentation/yu
Category: Serving, Scheduling
Priority: MUST

Why read:
Introduces iteration-level scheduling (continuous batching), which eliminated the inefficiency of waiting for the longest sequence in a batch to complete. This is now the default in production serving systems. Understanding Orca is essential before studying any modern scheduler.

Read specifically:
- Section 3 — the problem with request-level scheduling
- Section 4 — iteration-level scheduling design
- How selective batching handles different compute patterns for prefill vs decode

Questions:
1. Why is request-level scheduling (static batching) inefficient for autoregressive generation?
2. How does iteration-level scheduling improve GPU utilization?
3. What additional complexity does iteration-level scheduling introduce for memory management?

Related module: 04-serving-scheduling-capacity

---

## Sarathi-Serve: On Tailing Latency in LLM Inference with Chunked Prefills

Authors: Agrawal, Kedia, Panwar, Mohan, Kwatra, Gulavani, Ramjee, Tumanov
Year: 2024
URL: https://arxiv.org/abs/2308.16369
Category: Serving, Scheduling
Priority: MUST

Why read:
Addresses the prefill-decode interference problem. Shows how chunked prefill prevents long prefills from stalling decode iterations, improving tail latency. Critical for understanding how modern schedulers handle mixed workloads.

Read specifically:
- Analysis of how prefill operations cause latency spikes for concurrent decode requests
- Chunked prefill mechanism
- Impact on tail latency (p99)

Questions:
1. Why do long prefills cause decode latency spikes in continuous batching?
2. How does chunking the prefill mitigate this? What is the trade-off?
3. How does chunk size affect TTFT vs TPOT?

Related module: 04-serving-scheduling-capacity

---

## DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving

Authors: Zhong, Liu, Sheng, Jin, Zhang, Li, Zheng, Gonzalez, Stoica
Year: 2024
URL: https://arxiv.org/abs/2401.09670
Category: Serving, Distributed
Priority: SHOULD

Why read:
Proposes disaggregating prefill and decode onto separate GPU pools to optimize goodput. Demonstrates that prefill (compute-bound) and decode (memory-bound) have fundamentally different resource profiles. A key paper for capacity planning and system co-design.

Read specifically:
- Section 2 — analysis of prefill vs decode resource profiles
- The disaggregation architecture
- Goodput improvements under SLO constraints

Questions:
1. Why do prefill and decode benefit from different hardware configurations?
2. What are the communication costs of disaggregation?
3. Under what workloads is disaggregation NOT beneficial?

Related module: 04-serving-scheduling-capacity, 16-distributed-inference

# Foundations — Paper Index

## Attention Is All You Need

Authors: Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, Polosukhin
Year: 2017
URL: https://arxiv.org/abs/1706.03762
Category: Architecture
Priority: MUST

Why read:
The foundational transformer paper. Defines multi-head attention, positional encoding, and the encoder-decoder architecture that all modern LLMs descend from. Required to understand every subsequent architectural innovation in this curriculum.

Read specifically:
- Section 3 (Model Architecture) — the full architecture description
- Section 3.2.2 (Multi-Head Attention) — attention mechanism details
- Figure 1 — architecture diagram
- Figure 2 — scaled dot-product and multi-head attention

Questions:
1. Why is the attention scaled by 1/√d_k? What happens without scaling?
2. What is the computational complexity of self-attention with respect to sequence length?
3. Why multiple heads rather than a single attention function with larger dimensionality?

Related module: 01-llm-internals

---

## RoFormer: Enhanced Transformer with Rotary Position Embedding

Authors: Su, Lu, Pan, Murtadha, Wen, Liu
Year: 2021
URL: https://arxiv.org/abs/2104.09864
Category: Architecture
Priority: MUST

Why read:
Introduces Rotary Position Embedding (RoPE), now the dominant positional encoding in modern LLMs (LLaMA, Mistral, etc.). Understanding RoPE is essential for reasoning about position-dependent behavior, context length extension, and cache reuse.

Read specifically:
- Section 3.4 — RoPE formulation
- The rotation matrix construction and why it encodes relative position
- How RoPE interacts with attention computation

Questions:
1. How does RoPE encode relative position without explicit position embeddings?
2. What happens to attention scores as relative distance increases?
3. Why is RoPE preferred over learned absolute position embeddings in modern architectures?

Related module: 01-llm-internals

---

## GQA: Training Generalized Multi-Query Transformers from Multi-Head Checkpoints

Authors: Ainslie, Lee-Thorp, de Jong, Zemlyanskiy, Lebrón, Sanghai
Year: 2023
URL: https://arxiv.org/abs/2305.13245
Category: Architecture
Priority: MUST

Why read:
Introduces Grouped-Query Attention (GQA), the middle ground between MHA and MQA used in LLaMA 2 70B and subsequent models. Critical for understanding KV cache memory trade-offs — GQA directly determines KV cache size per token.

Read specifically:
- Section 2 — the GQA mechanism
- Figure 1 — visual comparison of MHA, MQA, GQA
- Ablation results comparing quality vs KV memory reduction

Questions:
1. How does the number of KV heads affect KV cache memory per token?
2. What quality trade-off does GQA make relative to MHA?
3. Why not just use MQA everywhere if it saves the most memory?

Related module: 01-llm-internals, 03-kv-cache-engineering

---

## Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity

Authors: Fedus, Zoph, Shazeer
Year: 2022
URL: https://arxiv.org/abs/2101.03961
Category: Architecture
Priority: SHOULD

Why read:
Foundational MoE paper for transformers. Explains expert routing, load balancing, and the key insight that sparse models can scale parameters without proportionally scaling compute. Essential background for understanding Mixtral and DeepSeek architectures.

Read specifically:
- Section 2 — simplified routing strategy
- Section 2.2 — load balancing loss
- Comparison of dense vs sparse scaling

Questions:
1. What problem does the auxiliary load balancing loss solve?
2. How does sparsity change the relationship between parameter count and FLOP count?
3. What are the communication costs of MoE routing in distributed settings?

Related module: 01-llm-internals, 16-distributed-inference

---

## Mixtral of Experts

Authors: Jiang, Sablayrolles, Roux, Mensch, Savary, Bamford, Chaplot, Casas, Hanna, Bressand, Lenber, Renard, Lacroix, Sirdey, El Sayed, Tamajon, Denoyer, Lample, Sayed
Year: 2024
URL: https://arxiv.org/abs/2401.04088
Category: Architecture
Priority: SHOULD

Why read:
Practical MoE deployment at scale. Shows how sparse expert models work in production and the engineering implications for serving, memory, and compute. Direct relevance to understanding modern model architectures.

Read specifically:
- Section 2 — Mixture of Experts layer design
- Expert selection and routing mechanism
- Comparison with dense models of similar compute budget

Questions:
1. How many experts are active per token? What does this imply for compute vs memory?
2. What are the serving challenges specific to MoE models?
3. How does expert placement interact with tensor parallelism?

Related module: 01-llm-internals, 16-distributed-inference

---

## DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model

Authors: DeepSeek-AI
Year: 2024
URL: https://arxiv.org/abs/2405.04434
Category: Architecture
Priority: SHOULD

Why read:
Introduces Multi-head Latent Attention (MLA), which compresses KV cache through low-rank projection. Demonstrates how architectural choices directly affect system economics. A key case study for model-system co-design.

Read specifically:
- Section 3.1 — Multi-head Latent Attention mechanism
- How MLA reduces KV cache size compared to GQA
- The relationship between compression ratio and quality

Questions:
1. How does MLA compress KV representations? What is the compression ratio?
2. What are the trade-offs of MLA vs GQA in terms of quality, memory, and compute?
3. How does MLA change the economics of serving at scale?

Related module: 01-llm-internals, 03-kv-cache-engineering, 19-model-system-codesign

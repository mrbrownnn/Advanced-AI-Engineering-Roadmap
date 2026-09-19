# Optimization — Paper Index

## FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

Authors: Dao, Fu, Ermon, Rudra, Ré
Year: 2022
URL: https://arxiv.org/abs/2205.14135
Category: Attention Optimization
Priority: MUST

Why read:
Introduces IO-aware attention computation that reduces HBM reads/writes by tiling attention computation in SRAM. The key insight is that attention is memory-bound, and the algorithmic optimization targets memory traffic rather than FLOP count. Foundational for all subsequent attention optimization work.

Read specifically:
- Section 2 — background on GPU memory hierarchy
- Section 3.1 — tiling algorithm
- Algorithm 1 — the forward pass
- Analysis of IO complexity vs standard attention

Questions:
1. Why is standard attention memory-bound despite being O(N²) in FLOPs?
2. How does tiling reduce HBM accesses? What is stored in SRAM?
3. What is the IO complexity of FlashAttention vs standard attention?

Related module: 05-inference-optimization

---

## FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning

Authors: Dao
Year: 2023
URL: https://arxiv.org/abs/2307.08691
Category: Attention Optimization
Priority: MUST

Why read:
Improves on FlashAttention with better work partitioning across GPU thread blocks and warps. Achieves closer to theoretical peak throughput. Essential for understanding how kernel-level optimization interacts with GPU architecture.

Read specifically:
- Section 3 — parallelism across sequence length and batch dimensions
- Work partitioning between warps
- Benchmark comparisons showing occupancy improvements

Questions:
1. What limited FlashAttention-1's GPU utilization?
2. How does FlashAttention-2 improve parallelism across thread blocks?
3. What is the practical speedup and under what conditions?

Related module: 05-inference-optimization

---

## GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers

Authors: Frantar, Ashkboos, Hoefler, Alistarh
Year: 2023
URL: https://arxiv.org/abs/2210.17323
Category: Quantization
Priority: MUST

Why read:
One-shot weight quantization to 4-bit and 3-bit with minimal quality loss. Essential for understanding weight quantization trade-offs in production. Widely used in practice.

Read specifically:
- The layer-wise quantization approach
- How reconstruction error is minimized
- Quality vs compression trade-offs at different bit widths

Questions:
1. Why quantize weights but not activations?
2. How does GPTQ minimize reconstruction error during quantization?
3. At what bit width does quality degradation become significant for different model sizes?

Related module: 05-inference-optimization

---

## AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration

Authors: Lin, Tang, Tang, Yang, Xiao, Han
Year: 2024
URL: https://arxiv.org/abs/2306.00978
Category: Quantization
Priority: MUST

Why read:
Improves on round-to-nearest quantization by identifying salient weight channels (determined by activation magnitudes) and protecting them. Achieves better quality than GPTQ at the same bit width for many models.

Read specifically:
- Section 3 — the observation that 1% of weights are critical
- The per-channel scaling approach
- Comparison with GPTQ across models and tasks

Questions:
1. What makes some weight channels more important than others?
2. How does AWQ protect salient channels without mixed-precision storage?
3. When would you choose AWQ over GPTQ?

Related module: 05-inference-optimization

---

## SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models

Authors: Xiao, Lin, Seznec, Wu, Demouth, Han
Year: 2023
URL: https://arxiv.org/abs/2211.10438
Category: Quantization
Priority: SHOULD

Why read:
Addresses activation quantization by migrating quantization difficulty from activations to weights via a mathematically equivalent per-channel scaling. Enables W8A8 quantization. Important for understanding the full weight+activation quantization picture.

Read specifically:
- Section 3 — the smoothing transformation
- Why activations are harder to quantize than weights
- The migration strength hyperparameter α

Questions:
1. Why are activations harder to quantize than weights?
2. How does the smoothing transformation preserve mathematical equivalence?
3. What is the role of the migration strength α?

Related module: 05-inference-optimization

---

## Fast Inference from Transformers via Speculative Decoding

Authors: Leviathan, Kalman, Matias
Year: 2023
URL: https://arxiv.org/abs/2211.17192
Category: Speculative Decoding
Priority: MUST

Why read:
Introduces speculative decoding: use a small draft model to predict multiple tokens, then verify in parallel with the target model. The key insight is that verification is cheaper than generation because it can be batched. Guarantees identical output distribution.

Read specifically:
- Section 3 — the speculative sampling algorithm
- The proof that output distribution is preserved
- Analysis of when speculation helps vs hurts

Questions:
1. Why is verification cheaper than sequential generation?
2. How does the acceptance rate depend on draft-target model agreement?
3. Under what conditions does speculative decoding provide no speedup?

Related module: 05-inference-optimization

---

## Medusa: Simple LLM Inference Acceleration Framework with Multiple Decoding Heads

Authors: Cai, Li, Geng, Peng, Lee, Chen, Dao
Year: 2024
URL: https://arxiv.org/abs/2401.10774
Category: Speculative Decoding
Priority: SHOULD

Why read:
An alternative to separate draft models: adds lightweight prediction heads to the target model itself to predict future tokens. Avoids the need for a separate draft model, simplifying deployment.

Read specifically:
- The multi-head architecture
- Tree-based verification
- Training the Medusa heads

Questions:
1. What are the advantages of self-speculation over a separate draft model?
2. How does tree-based verification improve acceptance rates?
3. What is the training cost of the Medusa heads?

Related module: 05-inference-optimization

---

## EAGLE: Speculative Sampling Requires Rethinking Feature Uncertainty

Authors: Li, Fang, Li, Chen, Xiao, Dai
Year: 2024
URL: https://arxiv.org/abs/2401.15077
Category: Speculative Decoding
Priority: SHOULD

Why read:
Improves on Medusa by operating on feature-level rather than token-level prediction. Achieves higher acceptance rates through better draft quality. Part of the evolving speculative decoding landscape.

Read specifically:
- The feature-level drafting approach
- Comparison with Medusa acceptance rates
- Speedup analysis across different models

Questions:
1. Why does feature-level prediction yield better drafts than token-level?
2. How does EAGLE's acceptance rate compare to Medusa and separate draft models?
3. What are the deployment trade-offs (training cost, memory overhead)?

Related module: 05-inference-optimization

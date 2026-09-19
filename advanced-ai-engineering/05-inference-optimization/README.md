# Module 05 — Inference Optimization

## Why This Module Exists

Optimization without understanding the bottleneck is guessing. This module is organized by **bottleneck** rather than by tool, teaching the learner to diagnose first and optimize second. Every optimization has a cost — complexity, quality, compatibility — and this module trains the judgment to choose wisely.

## Key Engineering Questions

- Is this workload memory-bound or compute-bound? Which optimization class applies?
- What is the actual speedup and what did it cost (quality, complexity, compatibility)?
- When does quantization hurt quality enough to matter for my use case?
- When does speculative decoding help vs hurt? What determines the acceptance rate?
- How do I write and debug a Triton kernel?

## Prerequisites

- Module 02 (Roofline, arithmetic intensity, profiling)
- Module 03 (KV cache memory)
- Module 04 (batching, throughput, latency metrics)

## Topics

### Memory Traffic Optimization
- FlashAttention: IO-aware tiling to reduce HBM reads/writes
- FlashAttention-2: improved parallelism and work partitioning
- FlashInfer: specialized attention kernels for serving
- Kernel fusion: reducing intermediate HBM round-trips

### Quantization (organized by what you quantize)
- Weight quantization: FP8, INT8, INT4
- GPTQ: one-shot post-training weight quantization
- AWQ: activation-aware weight quantization
- SmoothQuant: migrating quantization difficulty from activations to weights
- KV cache quantization: reducing KV memory footprint
- Quality-compression trade-offs for each approach

### Speculative Decoding
- Core idea: draft-then-verify for faster autoregressive generation
- Separate draft models
- Medusa: self-speculative multi-head prediction
- EAGLE: feature-level speculation
- Acceptance rate analysis: when speculation helps vs hurts
- Integration with serving systems

### Kernel Development
- Triton fundamentals: writing GPU kernels in Python
- When custom kernels are warranted vs using existing implementations
- Benchmarking and profiling custom kernels

## Expected Artifacts

1. **Bottleneck diagnosis** — profile a real workload, identify the bottleneck, select the appropriate optimization class
2. **Quantization comparison** — benchmark quality and speed for GPTQ vs AWQ at different bit widths on a specific task
3. **Speculative decoding analysis** — measure acceptance rate and speedup for a specific model pair and workload
4. **Engineering report** — optimization recommendation with evidence, trade-offs, and rollback criteria

## Exit Criteria

The learner can:
- Diagnose whether a workload needs memory traffic reduction, quantization, or speculative decoding
- Explain how FlashAttention reduces memory traffic and when it helps
- Choose between quantization methods based on quality, speed, and deployment constraints
- Predict when speculative decoding will provide a speedup and verify experimentally
- Write a simple Triton kernel and benchmark it against a reference implementation

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

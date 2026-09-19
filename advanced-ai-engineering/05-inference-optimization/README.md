# Module 05 — Inference Optimization

## Why This Module Exists

Optimization without understanding the bottleneck is guessing. This module is organized by **bottleneck** rather than by tool, teaching the learner to diagnose first and optimize second. Every optimization has a cost — complexity, quality, compatibility — and this module trains the judgment to choose wisely.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a basic IO-aware attention kernel (e.g., in Triton) that uses tiling to minimize HBM round-trips, recreating the core mechanism of FlashAttention.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Use PyTorch Profiler or Nsight to measure the arithmetic intensity of standard attention vs your tiled attention. Prove that the workload shifted from memory-bound toward compute-bound.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Push the sequence length or batch size until the optimized kernel OOMs or hits register spilling limits. Break the assumption that tiling solves all memory scaling issues.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the register spilling or shared memory bottleneck using profiling traces. Explain exactly why the optimization hit a wall.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Apply speculative decoding (draft-then-verify) on top of the optimized kernel.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice between FlashAttention-2 vs Speculative Decoding for a given workload (e.g., high-batch vs low-batch). Which optimizes throughput? Which optimizes latency?
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the Triton FlashAttention implementation to understand block size tuning and hardware constraints.

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

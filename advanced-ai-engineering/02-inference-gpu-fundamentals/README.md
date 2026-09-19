# Module 02 — Inference Fundamentals

## Why This Module Exists

LLM inference is not "just a forward pass." Understanding GPU execution, memory hierarchy, and the compute-vs-memory-bound distinction is essential for diagnosing performance problems, choosing optimizations, and making capacity decisions. Without this module, optimization becomes cargo-culting.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Write a simple CUDA kernel (or Triton equivalent) for vector addition and matrix multiplication.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Profile the kernel using Nsight Systems. Measure TTFT and TPOT on a simple LLM inference run.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Create a workload with massive memory fragmentation by allocating and freeing tensors randomly before launching the kernel.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the performance collapse using the Roofline model. Show exactly where the workload fell off the compute bound.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Re-engineer the memory access pattern to achieve coalesced global memory access.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend whether a specific workload is compute-bound or memory-bound and what optimization strategy should apply.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the CUDA programming guide sections on memory coalescing and warp scheduling.

## Expected Artifacts
- **Engineering Report**: Document the entire BUILD → MEASURE → BREAK → DEFEND loop with empirical evidence.
- **Implementation Code**: The scratch code demonstrating the mechanism.

## Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

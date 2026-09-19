# Module 16 — Distributed Inference

## Why This Module Exists

Large models don't fit on a single GPU. This module covers the parallelism strategies and communication primitives needed to serve models across multiple GPUs and nodes. The goal is architectural understanding and capacity reasoning, NOT an NCCL implementation course.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal pipeline parallel (PP) execution across two simulated devices. Manage the micro-batch scheduling (e.g., 1F1B schedule).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Derive the communication cost equation. Measure the bubble time (idle time) in the pipeline as a function of micro-batch size.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Create a workload with highly variable generation lengths. Break the pipeline efficiency, causing massive bubble times and synchronization stalls.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the straggler effect. Explain why static pipeline schedules fail under variable decode workloads.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement a dynamic scheduling mechanism or continuous batching across the pipeline stages to minimize idle time.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice between Tensor Parallelism (TP) and Pipeline Parallelism (PP) for a given interconnect topology (e.g., NVLink vs PCIe).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace the communication primitives in Megatron-LM or vLLM's distributed executor.

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

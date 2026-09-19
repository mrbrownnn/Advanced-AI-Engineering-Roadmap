# Module 03 — KV Cache Engineering

## Why This Module Exists

The KV cache is the single largest source of memory consumption in LLM serving and the primary constraint on concurrency. How you manage KV memory determines how many requests you can serve simultaneously, how you handle memory pressure, and how you exploit sharing across requests. This is where OS-level systems thinking meets ML inference.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal PagedAttention block manager from scratch (e.g., Python/NumPy) that handles logical-to-physical block mapping, allocation, reference counting, and deallocation.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Derive the exact HBM consumption equation for varying sequence lengths and batch sizes. Instrument the harness to measure block table lookup overhead.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Create a workload (e.g., highly variable sequence lengths) that severely fragments naive contiguous memory allocation, forcing an OOM at 50% utilization.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the exact point of fragmentation. Formulate a hypothesis on why preemption via swapping vs recomputation yields different TTFT/TPOT latency spikes.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement prefix caching or Radix tree caching to share KV blocks across common system prompts.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Compare INT8 KV quantization against FP16. Defend the trade-off between memory capacity gains and quality degradation with empirical evidence.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace `vllm/core/block_manager.py` to map your minimal implementation to production.

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

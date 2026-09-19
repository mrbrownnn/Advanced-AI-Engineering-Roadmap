# Module 02 — Inference Fundamentals

## Why This Module Exists

LLM inference is not "just a forward pass." Understanding GPU execution, memory hierarchy, and the compute-vs-memory-bound distinction is essential for diagnosing performance problems, choosing optimizations, and making capacity decisions. Without this module, optimization becomes cargo-culting.

## Key Engineering Questions

- Is my workload compute-bound or memory-bound? How do I determine this?
- Where is time being spent during prefill vs decode?
- What does the Roofline model tell me about my kernel's efficiency?
- How do I measure TTFT, TPOT, throughput, and goodput correctly?
- What is the difference between throughput and goodput under SLO constraints?

## Prerequisites

- Module 00 (measurement methodology)
- Module 01 (LLM architecture, FLOP and memory accounting)

## Topics

### GPU Execution Model
- Kernel launches, warps, thread blocks
- Occupancy and latency hiding
- Synchronization points

### Memory Hierarchy
- Register → shared memory (SRAM) → HBM → system memory
- Bandwidth at each level
- Why HBM bandwidth is the primary bottleneck for decode

### Arithmetic Intensity and Roofline
- Arithmetic intensity: FLOPs / bytes transferred
- Roofline model: identifying compute-bound vs memory-bound regions
- Applying Roofline to prefill (compute-bound) and decode (memory-bound)

### Profiling
- GPU profiling tools and methodology
- Identifying bottlenecks from profiling traces
- Common profiling pitfalls

### Latency Metrics
- TTFT (Time to First Token): prefill latency
- TPOT (Time per Output Token): decode latency
- Throughput: tokens/second
- Goodput: throughput under SLO constraints (requests that meet latency targets)

## Expected Artifacts

1. **Roofline analysis** — plot and analyze a real inference workload against the Roofline model
2. **Prefill vs decode characterization** — measure and explain the different bottlenecks
3. **Latency measurement report** — correctly measure TTFT, TPOT, throughput with appropriate statistics

## Exit Criteria

The learner can:
- Determine whether a given workload is compute-bound or memory-bound
- Use profiling tools to identify the dominant bottleneck in an inference pipeline
- Correctly measure and report TTFT, TPOT, throughput, and goodput
- Explain why prefill is compute-bound and decode is memory-bound

## Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze
  solo: Relational
  dreyfus: Advanced Beginner → Competent
```

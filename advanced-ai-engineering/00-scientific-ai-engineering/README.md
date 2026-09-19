# Module 00 — Foundations: Scientific Engineering Methodology

## Why This Module Exists

Every subsequent module depends on the ability to form hypotheses, design experiments, collect measurements, analyze results, and make evidence-based engineering decisions. Without this foundation, labs become "make it work" exercises rather than engineering investigations.

This module establishes the scientific rigor that separates an advanced engineer from someone who can follow tutorials.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build a benchmark harness that measures a simple noisy function (e.g. matrix multiplication). Ensure proper warmup and caching isolation.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure latency. Calculate the required sample size to achieve 95% confidence with a 5% margin of error.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Introduce a hidden confounder (e.g., thermal throttling or CPU background task) that skews the p99 latency without affecting the mean.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the confounder by analyzing the latency distribution (e.g., bimodal distribution). Explain why the mean is misleading.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement strict environment pinning, CPU affinity, and statistical outlier rejection.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the methodology used to claim a 'statistically significant speedup' in an engineering report.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the measurement logic in the MLPerf inference benchmark.

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

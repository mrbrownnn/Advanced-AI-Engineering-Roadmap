# Module 17 — Harness Evolution

## Why This Module Exists

AI systems degrade over time not just from data drift, but from upstream model upgrades, provider API changes, and shifting user distributions. Upgrading a foundational model or modifying prompt harnesses in production is fraught with regressions. Harness evolution establishes the discipline of continuous migration, automated prompt optimization, model routing, and backward compatibility.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build an automated prompt optimization loop (e.g. using DSPy) that optimizes few-shot examples against a metric.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the improvement on the evaluation set. Measure the variance of the optimizer across different random seeds.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Allow the optimizer to overfit to the evaluation set. Observe the catastrophic collapse on a hold-out test set.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the overfitting. Explain how prompt optimization acts exactly like gradient descent in finding spurious correlations.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement cross-validation, strict data splits, and semantic similarity bounding for the optimizer.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the deployment of an optimized harness via a shadow routing / canary rollout strategy.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the implementation of the DSPy teleprompter.

## Expected Artifacts
- **Engineering Report**: Document the entire BUILD → MEASURE → BREAK → DEFEND loop with empirical evidence.
- **Implementation Code**: The scratch code demonstrating the mechanism.

## Competency Targets

```yaml
competency:
  sfia: 5-6
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

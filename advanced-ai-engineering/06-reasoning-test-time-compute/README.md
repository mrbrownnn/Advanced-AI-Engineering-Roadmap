# Module 06 — Reasoning and Test-Time Compute

## Why This Module Exists

Modern LLMs increasingly use test-time compute strategies — chain-of-thought, search, verification, and self-correction — to solve harder problems. Understanding how models allocate compute at inference time is essential for engineering systems that handle reasoning tasks, and for understanding the cost and latency implications of reasoning-heavy workloads.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a Tree-of-Thought search loop using a small model as a generator and evaluator.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Calculate the exact token cost multiplier of the reasoning search vs direct prompting. Measure the accuracy gain.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Provide a problem where the evaluator model is systematically biased (e.g., prefers longer answers regardless of correctness), causing the search to fail.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the failure by analyzing the search tree. Explain the failure mode of the evaluator.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement a process reward model (PRM) or a verified execution step (e.g., Python REPL) to ground the evaluation.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the trade-off between test-time compute scaling (more search) vs parameter scaling (using a bigger model).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the original Tree of Thoughts implementation.

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

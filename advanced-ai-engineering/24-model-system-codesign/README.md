# Module 19 — Model-System Co-design

## Why This Module Exists

Model architecture decisions propagate through the entire system stack. This module connects the dots: how attention mechanism choices cascade through KV cache sizing, HBM consumption, concurrency limits, scheduler design, and ultimately economics. This is the integration module — it ties the full curriculum together.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Model the full dependency chain: Attention choice (MHA/GQA/MLA) -> KV Cache footprint -> HBM consumption -> Max concurrency -> Scheduler throughput -> Cost per request.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Calculate the exact financial impact of switching a production workload from MHA to GQA.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Select a model architecture that is highly compute-efficient (e.g. extreme MoE) but demands a network topology that breaks the deployment budget.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the systemic mismatch. Explain why optimizing one variable destroyed the system viability.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Design a balanced co-design that matches the model architecture to the exact hardware topology available.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the final, integrated architecture recommendation to a hypothetical executive board.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read system-level papers on MoE deployment architectures.

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

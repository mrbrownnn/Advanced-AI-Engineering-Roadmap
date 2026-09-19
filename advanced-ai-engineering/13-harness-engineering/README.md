# Module 13 — Harness Engineering

## Why This Module Exists

In production, models are never invoked in isolation. They operate inside an execution harness that formats prompts, enforces schemas, parses structured outputs, manages retries, handles fallbacks, and isolates failures. Harness engineering treats prompts and invocation protocols as software engineering artifacts with rigorous versioning, typing, and validation.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a prompt execution harness that enforces structured JSON output using grammar-guided decoding (e.g., using a finite state machine).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the exact TTFT and TPOT overhead introduced by the grammar constraint mechanism compared to unconstrained generation.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Construct a prompt and schema combination that forces the grammar decoder into an invalid state or a generative loop (e.g., forcing a closing bracket prematurely).
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the truncation or looping failure. Explain how the constrained sampling logic interacted poorly with the model's logits.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement a fallback cascade: try constrained decoding first, fallback to unconstrained + client-side retry/repair on failure.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice between engine-level grammar constraints vs client-side validation based on empirical throughput and reliability metrics.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace the outlines or guidance library to see how regex is compiled into logit processors.

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

# Module 12 — Evaluation Engineering

## Why This Module Exists

Evaluation is the hardest unsolved problem in AI engineering. Bad evaluation leads to bad decisions. This module treats evaluation as an engineering discipline: measurement design, calibration, regression testing, and understanding the gap between offline metrics and production behavior.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build an LLM-as-a-judge evaluation pipeline. Define explicit grading rubrics and implement inter-rater agreement tracking (e.g., Cohen's Kappa).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the calibration of the judge: does a score of 4/5 correlate strongly with human preference? Measure the cost per evaluation.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Create an adversarial response that is completely incorrect but highly verbose, perfectly formatted, and confident. Fool the LLM judge into giving it a perfect score.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the judge's bias (e.g., verbosity bias, style bias, position bias). Formulate a hypothesis on why the judge failed.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement Chain-of-Thought reasoning for the judge before it outputs a score, and calibrate it against a golden human dataset.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the validity of the evaluation metric. Prove that an improvement in the offline metric will translate to an improvement in production.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the Prometheus or MT-Bench evaluation implementation.

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

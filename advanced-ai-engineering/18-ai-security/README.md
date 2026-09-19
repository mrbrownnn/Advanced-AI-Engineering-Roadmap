# Module 14 — AI Security

## Why This Module Exists

AI systems expand the attack surface in unique ways: the model processes untrusted input as instructions, retrieval systems can be poisoned, and agents can be tricked into taking harmful actions. This module covers threat modeling and defense for AI-specific attack vectors.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a comprehensive threat model (STRIDE) for a given LLM architecture.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the latency overhead of various input/output guardrails.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Execute a successful indirect prompt injection attack via a poisoned retrieved document that exfiltrates data via a tool call.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the confused deputy problem. Explain the failure of the trust boundaries.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement strict capability scoping, sandboxing, and output validation.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a security architecture that mitigates risk while maintaining system utility.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read standard AI red-teaming methodologies and vulnerability databases.

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

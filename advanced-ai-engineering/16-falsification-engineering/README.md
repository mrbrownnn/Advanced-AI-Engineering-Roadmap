# Module 13 — Falsification Engineering

## Why This Module Exists

Standard evaluation tells you the system works on your test cases. Falsification tries to prove the system is broken. This adversarial mindset — systematically seeking failure modes — is what separates robust systems from fragile ones. This module teaches structured approaches to breaking AI systems.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build a property-based testing harness for an AI system (e.g., testing invariants rather than specific input-output pairs).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the coverage of the invariant tests. Calculate the false positive and false negative rates of the test suite.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Design a prompt injection attack (direct or indirect) that bypasses the system's guardrails. Break the invariant that 'the system never outputs PII'.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose exactly how the attack bypassed the guardrail. Was it token splitting, role manipulation, or context window overflow?
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement input sanitization, instruction hierarchies, or a secondary validation model to defend against the discovered attack.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the trade-off between security strictness (which increases false positives/rejections) and usability.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the implementation of standard red-teaming frameworks like Garak or promptmap.

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

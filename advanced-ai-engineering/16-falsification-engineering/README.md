# Module 13 — Falsification Engineering

## Why This Module Exists

Standard evaluation tells you the system works on your test cases. Falsification tries to prove the system is broken. This adversarial mindset — systematically seeking failure modes — is what separates robust systems from fragile ones. This module teaches structured approaches to breaking AI systems.

## Key Engineering Questions

- How do I systematically generate inputs that expose failures?
- How do I test properties of the system rather than specific input-output pairs?
- How do I detect when the evaluator itself is fooled?
- How do I distinguish genuine improvement from gaming the metric?
- How does Goodhart's Law manifest in AI systems?

## Prerequisites

- Module 12 (evaluation methods, judge calibration)
- Module 06 (failure taxonomy)

## Topics

### Systematic Testing
- Property-based testing: testing invariants rather than specific examples
- Metamorphic testing: if input changes in way X, how should output change?
- Mutation testing: introducing small changes to verify detection

### Fault Injection
- Fault injection for AI: corrupting context, tools, retrieval results
- Stress testing: behavior under extreme inputs, loads, edge cases
- Degradation testing: graceful failure vs catastrophic failure

### Adversarial Evaluation
- Adversarial examples: inputs designed to cause failures
- Prompt attacks: injection, jailbreaks, role manipulation
- Red-teaming methodology: structured adversarial evaluation

### Evaluator Robustness
- Grader gaming: when the system learns to satisfy the evaluator rather than the task
- Reward hacking: optimizing proxy objectives instead of true objectives
- Goodhart's Law in AI: "when a measure becomes a target, it ceases to be a good measure"
- Weak evaluator detection: identifying when your evaluator can't distinguish good from bad

## Expected Artifacts

1. **Property-based test suite** — define and test invariants for an AI system
2. **Metamorphic test suite** — test transformation invariants
3. **Adversarial evaluation** — structured attempt to break a system
4. **Engineering report** — falsification findings and recommended mitigations

## Exit Criteria

The learner can:
- Design property-based and metamorphic tests for AI systems
- Conduct structured adversarial evaluation (red-teaming)
- Detect when an evaluator is being gamed
- Identify Goodhart's Law dynamics in evaluation-optimization loops
- Distinguish genuine improvement from metric gaming

## Competency Targets

```yaml
competency:
  sfia: 5-6
  bloom: Evaluate → Create
  solo: Relational → Extended Abstract
  dreyfus: Competent → Proficient
```

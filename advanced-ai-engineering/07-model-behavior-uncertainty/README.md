# Module 06 — Model Behavior and Uncertainty

## Why This Module Exists

Models fail in predictable ways. Understanding failure taxonomy, calibration, and uncertainty enables engineering controls: selective prediction, abstention, escalation, and regression testing. This module shifts from "the model works" to "I know when and how the model fails."

## Key Engineering Questions

- What type of failure is this? (hallucination, instruction failure, reasoning failure, retrieval failure, tool failure)
- How calibrated is the model? Is its confidence meaningful?
- When should the system abstain or escalate instead of answering?
- How do I detect behavior regression after model or prompt changes?
- How do I handle distribution shift in production?

## Prerequisites

- Module 00 (measurement, experimental design)
- Module 01 (model architecture)

## Topics

### Failure Taxonomy
- Hallucination types: factual, reasoning, attribution
- Instruction failure: ignoring or misinterpreting instructions
- Reasoning failure: logical errors, arithmetic errors
- Retrieval failure: wrong context selected, relevant context missed
- Tool failure: incorrect tool calls, missing parameters

### Calibration and Uncertainty
- Calibration: does 80% confidence mean 80% correct?
- Overconfidence: when models are wrong but certain
- Model disagreement: using multiple models or samples to estimate uncertainty
- Consistency: does the model give the same answer when asked differently?

### Decision Under Uncertainty
- Selective prediction: only answer when confidence is high enough
- Abstention: refusing to answer with explicit uncertainty signal
- Escalation: routing to human or more capable system
- Threshold selection: precision-recall trade-offs for abstention

### Production Behavior
- Distribution shift: when production inputs differ from training/evaluation data
- Behavior regression: detecting degradation after changes
- Monitoring model behavior over time

## Expected Artifacts

1. **Failure taxonomy for a specific system** — categorize failures from a real or realistic evaluation set
2. **Calibration analysis** — measure calibration of a model on a specific task
3. **Selective prediction investigation** — implement and evaluate an abstention strategy
4. **Engineering report** — recommendation for handling uncertainty in a specific application

## Exit Criteria

The learner can:
- Classify model failures by type and identify the root cause
- Measure calibration and assess whether model confidence is usable
- Design and evaluate a selective prediction / abstention strategy
- Set up behavior regression testing for model or prompt changes

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

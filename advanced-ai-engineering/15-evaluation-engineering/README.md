# Module 12 — Evaluation Engineering

## Why This Module Exists

Evaluation is the hardest unsolved problem in AI engineering. Bad evaluation leads to bad decisions. This module treats evaluation as an engineering discipline: measurement design, calibration, regression testing, and understanding the gap between offline metrics and production behavior.

## Key Engineering Questions

- How do I evaluate something that doesn't have a single right answer?
- How do I calibrate and validate an LLM judge?
- How do I detect evaluation dataset contamination?
- How do I test for regression after model/prompt/system changes?
- Why do my offline evaluations disagree with production behavior?

## Prerequisites

- Module 00 (measurement, experimental design)
- Module 06 (failure taxonomy)
- Module 07 (data quality, leakage, contamination)

## Topics

### Evaluation Methods
- Deterministic evaluation: exact match, string matching, code execution
- Semantic evaluation: embedding similarity, entailment, LLM-as-judge
- Pairwise evaluation: comparing outputs head-to-head
- Trajectory evaluation: evaluating multi-step agent behavior
- Tool evaluation: correctness of tool calls and parameters
- Retrieval evaluation: quality of retrieved context (Recall@K, MRR, NDCG)
- Human evaluation: annotation protocols, guidelines, calibration

### Judge Engineering
- Judge calibration: is the LLM judge's scoring consistent and meaningful?
- Inter-rater agreement: do multiple judges agree? (Cohen's kappa, Krippendorff's alpha)
- Judge bias: position bias, verbosity bias, style bias
- Evaluator overfitting: when the system optimizes for the evaluator rather than the task

### Evaluation Infrastructure
- Regression testing: automated evaluation on every change
- Dataset versioning: tracking evaluation set changes over time
- Slicing: evaluating performance on specific subgroups
- Leakage: evaluation data appearing in training or fine-tuning data
- Contamination: benchmark data in model pretraining

### Production Gap
- Offline-online mismatch: when lab results don't match production behavior
- Distribution mismatch between evaluation data and production traffic
- Temporal drift: evaluation sets becoming stale

## Expected Artifacts

1. **Evaluation framework** — implement a multi-method evaluation for a specific task
2. **Judge calibration study** — measure and improve LLM judge reliability
3. **Regression test suite** — automated evaluation triggered on changes
4. **Engineering report** — evaluation strategy for a production system

## Exit Criteria

The learner can:
- Design an evaluation strategy combining multiple methods
- Calibrate an LLM judge and measure inter-rater agreement
- Detect evaluation contamination and leakage
- Build regression testing into the development workflow
- Explain and mitigate offline-online evaluation mismatch

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

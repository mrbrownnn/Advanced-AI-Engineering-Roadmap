# Module 00 — Foundations: Scientific Engineering Methodology

## Why This Module Exists

Every subsequent module depends on the ability to form hypotheses, design experiments, collect measurements, analyze results, and make evidence-based engineering decisions. Without this foundation, labs become "make it work" exercises rather than engineering investigations.

This module establishes the scientific rigor that separates an advanced engineer from someone who can follow tutorials.

## Key Engineering Questions

- How do I know my measurement is reliable?
- How do I distinguish signal from noise?
- How do I design an experiment that actually answers my question?
- How do I avoid fooling myself with confounders?
- How do I make decisions under uncertainty?
- How do I communicate engineering evidence?

## Prerequisites

- Basic programming proficiency (Python)
- Basic statistics awareness (mean, median, standard deviation)
- No ML-specific prerequisites — this module is domain-general

## Topics

### Measurement and Reproducibility
- What makes a measurement reliable
- Sources of variance: hardware, software, environment, workload
- Reproducibility: pinning environments, seeds, configurations
- Reporting methodology

### Experimental Design
- Hypothesis formation: falsifiable, specific, quantitative
- Independent, dependent, and controlled variables
- Confounders and how to control for them
- Ablation studies: isolating the contribution of each component
- Sample size and statistical power

### Distributions and Statistics
- Distributions: normal, skewed, heavy-tailed
- Percentiles (p50, p95, p99) and why mean is often misleading
- Variance and standard deviation
- Confidence intervals and bootstrap intuition
- Effect size: practical significance vs statistical significance

### Evaluation Strategies
- Offline evaluation: static datasets, hold-out sets, cross-validation
- Shadow deployment: running new systems against production traffic without serving
- Canary deployment: gradual rollout with monitoring
- A/B testing: controlled experiments in production
- Rollback: criteria, mechanisms, and decision framework

## Expected Artifacts

1. **Experiment report** — a complete investigation following the [engineering report template](../templates/engineering-report.md) on a simple measurement task
2. **Measurement reliability analysis** — demonstrate that a measurement is (or isn't) trustworthy
3. **Ablation study** — isolate the contribution of components in a simple system

## Exit Criteria

The learner can:
- Design a controlled experiment for a given engineering question
- Identify confounders in an experimental setup
- Collect measurements with appropriate warmup, sample size, and variance reporting
- Distinguish meaningful differences from noise using confidence intervals
- Write a complete engineering report with predictions, measurements, and decision rationale

## Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze
  solo: Relational
  dreyfus: Competent
```

# Module 07 — AI Data Engineering

## Why This Module Exists

Data quality determines model quality. This module covers the engineering of data systems for AI — from provenance and validation through drift detection and active learning. It focuses on the infrastructure and processes that ensure data reliability, not on data science methodology.

## Key Engineering Questions

- Can I trace every data point back to its origin?
- How do I detect and prevent data leakage and contamination?
- How do I validate data quality at scale?
- How do I detect and respond to data drift?
- When is synthetic data appropriate and how do I validate it?

## Prerequisites

- Module 00 (measurement, experimental design)
- Module 06 (failure taxonomy — data-related failures)

## Topics

### Data Lifecycle
- Provenance: where did this data come from?
- Lineage: what transformations were applied?
- Versioning: reproducible snapshots of datasets
- Data contracts: schema and quality agreements between producers and consumers

### Quality
- Validation: schema validation, statistical validation, semantic validation
- Deduplication: near-duplicate detection at scale
- Leakage: train-test leakage, evaluation contamination
- Contamination: benchmark data in training sets

### Distribution
- Slicing: analyzing performance across data subgroups
- Imbalance: detection and mitigation strategies
- Drift: covariate shift, concept drift, label drift
- Monitoring drift in production

### Data Generation
- Human annotation: guidelines, inter-rater agreement, quality control
- Weak labeling: programmatic labeling with noise-aware training
- Synthetic data: generation, validation, and risks
- Active learning: selecting the most informative examples for labeling
- Hard-example mining: finding the data the model struggles with

## Expected Artifacts

1. **Data validation pipeline** — implement quality checks for a dataset
2. **Leakage audit** — check for data contamination in an evaluation setup
3. **Drift detection** — implement and test a drift detection mechanism
4. **Engineering report** — data quality strategy for a specific application

## Exit Criteria

The learner can:
- Design a data validation pipeline with provenance tracking
- Detect data leakage and contamination in an evaluation setup
- Implement drift detection and define response policies
- Evaluate the quality and risks of synthetic data

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

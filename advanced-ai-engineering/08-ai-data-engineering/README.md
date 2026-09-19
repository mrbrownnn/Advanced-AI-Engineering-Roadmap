# Module 07 — AI Data Engineering

## Why This Module Exists

Data quality determines model quality. This module covers the engineering of data systems for AI — from provenance and validation through drift detection and active learning. It focuses on the infrastructure and processes that ensure data reliability, not on data science methodology.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build a data validation pipeline that detects exact and near-duplicates, schema violations, and test-set leakage.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the false positive rate of the near-duplicate detection at scale using MinHash.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Inject subtle train-test leakage that standard exact-match filters miss, causing an artificially inflated evaluation score.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the leakage path. Explain why the evaluation metric is now fundamentally compromised.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement semantic deduplication and rigorous cryptographic hashing of train/test splits.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a data provenance and versioning strategy for a production AI pipeline.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the data preparation pipeline scripts for a major open-source dataset like Dolma.

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

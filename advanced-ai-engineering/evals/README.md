# Evals

_This directory will contain evaluation suite definitions and configurations._

## Conventions

- Each eval defines: task, dataset, metrics, judge configuration, and pass criteria
- Evals are versioned alongside the datasets they use
- Results are stored in `benchmarks/results/`, not here
- Check for contamination between eval data and any training/fine-tuning data

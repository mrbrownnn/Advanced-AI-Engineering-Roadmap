# Module 17 — Harness Evolution

## Why This Module Exists

AI systems degrade over time not just from data drift, but from upstream model upgrades, provider API changes, and shifting user distributions. Upgrading a foundational model or modifying prompt harnesses in production is fraught with regressions. Harness evolution establishes the discipline of continuous migration, automated prompt optimization, model routing, and backward compatibility.

## Key Engineering Questions

- How do I safely upgrade an underlying foundation model (e.g., from version N to N+1) without breaking downstream agent behaviors?
- How do automated prompt optimization techniques (e.g., DSPy, MIPRO, TextGrad) fit into an engineering workflow?
- When should prompt optimizations be baked into a fine-tuned adapter versus kept in dynamic harness layers?
- How do I design shadow-routing and canary pipelines specifically for generative AI harnesses?
- What are the rollback criteria when a prompt or harness change exhibits subtle semantic degradation?

## Prerequisites

- Module 13 (Harness Engineering)
- Module 15 (Evaluation Engineering)
- Module 16 (Falsification & Adversarial Engineering)

## Topics

### Model Upgrade and Migration Engineering
- Behavioral diffing between model checkpoints and provider revisions
- Detecting silent capability collapse and prompt sensitivity shifts
- Maintaining backward compatibility in multi-step agent workflows

### Automated Prompt and Harness Optimization
- Metric-driven prompt tuning frameworks (DSPy teleprompters, search algorithms)
- Few-shot bootstrapping and automated demonstration selection
- Overfitting to evaluation sets and out-of-distribution generalization

### Progressive Rollout and Lifecycle Management
- Canary routing strategies based on task complexity and semantic similarity
- Shadow evaluation against live production traffic
- Automated rollback triggers based on judge scores and user feedback loops

## Expected Artifacts

1. **Migration harness** — automated test runner comparing model N vs model N+1 outputs on a suite of golden tasks
2. **Automated harness optimization pipeline** — end-to-end DSPy or custom compilation script optimizing prompt demonstrations against a test metric
3. **Engineering report** — migration analysis document detailing regression rates, cost differences, and go/no-go rollback decisions

## Exit Criteria

The learner can:
- Execute a zero-downtime model migration with quantifiable behavioral regression bounds
- Use automated prompt optimization with safeguards against evaluation set leakage
- Establish canary routing and automated rollback mechanisms for production harnesses

## Competency Targets

```yaml
competency:
  sfia: 5-6
  bloom: Evaluate → Create
  solo: Extended Abstract
  dreyfus: Proficient
```

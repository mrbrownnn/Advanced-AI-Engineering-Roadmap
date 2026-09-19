# Engineering Report Template

This report is mandatory for major checkpoints. Every section should be completed with evidence, not speculation.

---

## Problem

_What engineering problem are you solving? Be specific._

## Context

_System architecture, workload characteristics, constraints, business requirements._

## Assumptions

_List every assumption explicitly. Each will be tested._

## Hypotheses

_Ordered by likelihood. Each must be falsifiable._

1.
2.
3.

## Prediction

_What do you expect to observe if each hypothesis is correct? Be quantitative._

## Experimental Design

_How will you test the hypotheses? What variables are controlled? What is the sample size?_

## Environment

```yaml
hardware: _
gpu: _
cpu: _
memory: _
os: _
software_versions:
  python: _
  torch: _
  # ... relevant packages
commit: _
date: _
```

## Measurements

_Raw data. Include methodology for collection._

## Results

_Aggregated metrics with confidence intervals or error bars. Visualizations if appropriate._

## Failure Analysis

_What broke? Why? What was surprising? What was the root cause?_

## Alternative Explanations

_What else could explain the results? Why do you favor your explanation?_

## Decision

_What engineering decision did you make based on the evidence?_

## Trade-offs

_What did you gain? What did you give up? Under what conditions would you choose differently?_

## Cost

_What does this decision cost in terms of compute, latency, complexity, maintenance?_

## Risks

_What could go wrong with this decision in production?_

## Rollback Criteria

_Under what specific, measurable conditions should this decision be reversed?_

## What Would Change My Decision?

_What new information, workload change, or constraint change would invalidate this decision?_

# Module 17 — AI Economics

## Why This Module Exists

Every engineering decision is an economic decision. This module teaches cost modeling, optimization, and the quality-latency-cost trade-off space. The engineer who can quantify the economic impact of technical decisions is far more valuable than one who only optimizes for technical metrics.

## Key Engineering Questions

- What does a request actually cost? What does a successful task cost?
- How do I model the cost-quality-latency Pareto frontier for my workload?
- When should I route to a cheaper model? When does that hurt more than it saves?
- When should I build vs use an API?
- How do I plan capacity across reserved and elastic resources?

## Prerequisites

- Module 04 (capacity planning)
- Module 05 (optimization trade-offs)
- Module 02 (throughput, goodput)

## Topics

### Cost Modeling
- Cost per request: compute + memory + network + storage
- Cost per token: input tokens vs output tokens
- Cost per successful task: including retries, fallbacks, and failures
- GPU utilization economics: idle GPUs, batching efficiency, right-sizing

### Optimization Strategies
- Model routing: directing requests to the cheapest model that meets quality requirements
- Cascades: try cheap model first, escalate to expensive model on failure/uncertainty
- Fallback: graceful degradation when the primary model is unavailable or over budget
- Semantic caching: reusing responses for similar queries
- Workload segmentation: different strategies for different traffic classes

### Trade-off Analysis
- Quality-latency-cost Pareto frontier: mapping the feasible trade-off space
- Multi-objective optimization: balancing competing concerns
- Marginal cost analysis: the cost of the next unit of quality or speed
- Opportunity cost: what you give up by choosing one strategy

### Strategic Decisions
- Build vs API: total cost of ownership analysis
- Reserved vs elastic capacity: when to commit vs when to scale on demand
- Model size selection: larger model with lower utilization vs smaller model with higher utilization

## Expected Artifacts

1. **Cost model** — compute cost per request and per successful task for a real workload
2. **Routing experiment** — implement and evaluate a model routing strategy
3. **Pareto analysis** — map the quality-latency-cost frontier for a specific task
4. **Engineering report** — cost optimization strategy with ROI analysis

## Exit Criteria

The learner can:
- Build a complete cost model for an AI workload
- Design and evaluate model routing and cascade strategies
- Map and navigate the quality-latency-cost Pareto frontier
- Produce a build-vs-API analysis with total cost of ownership
- Make capacity planning decisions with economic justification

## Competency Targets

```yaml
competency:
  sfia: 5-6
  bloom: Evaluate → Create
  solo: Relational → Extended Abstract
  dreyfus: Competent → Proficient
```

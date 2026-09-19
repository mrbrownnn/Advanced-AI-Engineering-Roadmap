# Module 06 — Reasoning and Test-Time Compute

## Why This Module Exists

Modern LLMs increasingly use test-time compute strategies — chain-of-thought, search, verification, and self-correction — to solve harder problems. Understanding how models allocate compute at inference time is essential for engineering systems that handle reasoning tasks, and for understanding the cost and latency implications of reasoning-heavy workloads.

## Key Engineering Questions

- What is test-time compute scaling and when does it outperform parameter scaling?
- How do chain-of-thought, tree-of-thought, and search-based methods differ in cost and reliability?
- How do I engineer systems for reasoning workloads where output length is unpredictable?
- What are the serving and scheduling implications of reasoning-heavy requests?
- How do I verify reasoning outputs and detect reasoning failures?

## Prerequisites

- Module 01 (model internals, autoregressive generation)
- Module 02 (inference costs, TTFT/TPOT)
- Module 04 (scheduling — reasoning requests create variable-length decode)

## Topics

### Reasoning Mechanisms
- Chain-of-thought prompting and its variants
- Self-consistency: sampling multiple reasoning paths
- Tree-of-thought: structured search over reasoning steps
- Verification and self-correction loops
- Process reward models: scoring intermediate steps

### Test-Time Compute Scaling
- Compute-optimal inference: when to think longer vs use a bigger model
- Scaling laws for test-time compute
- Variable output length: implications for scheduling and batching
- Cost modeling for reasoning workloads

### Reasoning Failures
- Faithful vs unfaithful chain-of-thought
- Reasoning shortcuts and pattern matching
- Compounding errors in multi-step reasoning
- Detecting unreliable reasoning

### System Implications
- Scheduling: reasoning requests consume unpredictable decode tokens
- KV cache pressure from long reasoning chains
- Batching: mixing reasoning and non-reasoning requests
- Cost: reasoning tokens are "hidden" cost multipliers
- Timeout and budget management for open-ended reasoning

## Expected Artifacts

1. **Reasoning cost analysis** — measure the compute and latency cost of chain-of-thought vs direct answering across task types
2. **Self-consistency experiment** — measure how sampling multiple reasoning paths affects accuracy and cost
3. **Scheduling impact study** — analyze how reasoning workloads affect serving throughput and tail latency
4. **Engineering report** — when to enable reasoning and how to manage its cost

## Exit Criteria

The learner can:
- Explain the test-time compute scaling hypothesis and its engineering implications
- Measure the cost-accuracy trade-off of reasoning strategies
- Identify reasoning failures and distinguish faithful from unfaithful reasoning
- Design scheduling and budgeting policies for reasoning-heavy workloads
- Make evidence-based decisions about when reasoning is worth its cost

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

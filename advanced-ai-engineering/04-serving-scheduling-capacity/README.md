# Module 04 — Serving, Scheduling, and Capacity

## Why This Module Exists

The scheduler is the brain of an inference serving system. It decides what runs, when, and in what combination. Poor scheduling turns expensive GPUs into idle hardware or causes SLO violations under moderate load. This module connects queueing theory to practical scheduler design.

## Key Engineering Questions

- What is the throughput difference between static, dynamic, and continuous batching?
- How does chunked prefill prevent decode latency spikes?
- At what arrival rate does the system saturate? How do I predict the saturation knee?
- How do I apply Little's Law to capacity planning for LLM serving?
- How do I design admission control and load shedding policies?
- How do I plan capacity for a given SLO, traffic pattern, and growth rate?

## Prerequisites

- Module 02 (TTFT, TPOT, throughput, goodput)
- Module 03 (KV cache lifecycle, memory pressure)

## Topics

### Batching Strategies
- Static batching as baseline: wait for batch, process, return
- Dynamic batching: group arriving requests
- Continuous batching (iteration-level scheduling): add/remove requests per iteration
- Chunked prefill: breaking long prefills into chunks to reduce decode interference

### Scheduling
- Priority scheduling: latency-sensitive vs throughput-optimized requests
- Fairness: preventing starvation under mixed workloads
- Admission control: rejecting requests to protect SLOs
- SLO-aware scheduling: optimizing for goodput, not just throughput

### Queueing Fundamentals
- Arrival rate, service rate, utilization
- Little's Law: L = λW
- Burstiness and its impact on tail latency
- Saturation knee: the utilization point where latency explodes
- Queueing latency vs processing latency

### Capacity and Resilience
- Backpressure: propagating overload signals upstream
- Load shedding: graceful degradation under extreme load
- Capacity planning: sizing for steady state, bursts, and growth
- Multi-model and multi-tenant scheduling

## Expected Artifacts

1. **Batching comparison** — benchmark static vs dynamic vs continuous batching for the same workload
2. **Saturation analysis** — identify the saturation knee for a specific system and workload
3. **Capacity plan** — produce a capacity plan for a given SLO, traffic pattern, and growth projection
4. **Engineering report** — scheduling strategy recommendation with evidence

## Exit Criteria

The learner can:
- Explain why continuous batching outperforms static batching for autoregressive generation
- Predict and measure the saturation knee for a serving system
- Apply Little's Law to estimate queue depth and latency
- Design an admission control policy for a given SLO and traffic pattern
- Produce a defensible capacity plan with quantitative justification

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

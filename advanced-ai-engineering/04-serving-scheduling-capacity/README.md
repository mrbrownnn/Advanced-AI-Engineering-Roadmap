# Module 04 — Serving, Scheduling, and Capacity

## Why This Module Exists

The scheduler is the brain of an inference serving system. It decides what runs, when, and in what combination. Poor scheduling turns expensive GPUs into idle hardware or causes SLO violations under moderate load. This module connects queueing theory to practical scheduler design.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal continuous batching scheduler (iteration-level scheduling) that handles request arrivals and departures dynamically.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the throughput (tokens/sec) and goodput (requests meeting SLO). Model the queue using Little's Law.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Inject a burst of traffic that exceeds the service rate. Observe the saturation knee where tail latency explodes.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the queue buildup. Explain the mathematical relationship between utilization and queueing delay as utilization approaches 100%.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement an admission control and load shedding policy to protect the SLO during traffic bursts.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a capacity plan for a given expected traffic pattern and strict p99 latency SLO.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace the scheduling loop in vLLM's `Scheduler` class.

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

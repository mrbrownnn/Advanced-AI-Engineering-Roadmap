# Module 18 — Observability and Reliability

## Why This Module Exists

You cannot improve what you cannot observe. This module covers the instrumentation, telemetry, alerting, and incident response practices for AI systems. Observability is not just logging — it is the ability to ask new questions about system behavior without deploying new code.

> **Note:** Observability concepts are progressively introduced starting in Module 02. This module integrates and deepens them into a complete operational practice.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a distributed tracing system (e.g., OpenTelemetry) across an LLM, a retrieval database, and an agent tool.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Define and measure specific SLOs (latency, quality, cost) for the system.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Simulate a silent failure (e.g., an upstream API degrades subtly but doesn't throw errors). Watch standard metrics fail to catch it.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the observability gap. Explain why traditional APM metrics (CPU, HTTP 500s) are insufficient for AI.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement semantic monitoring (e.g., tracking embedding distances or LLM-as-a-judge scores in the telemetry pipeline).
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend an incident response runbook and the chosen alerting thresholds.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the OpenTelemetry specification for generative AI spans.

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

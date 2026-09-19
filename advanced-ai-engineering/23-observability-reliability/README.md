# Module 18 — Observability and Reliability

## Why This Module Exists

You cannot improve what you cannot observe. This module covers the instrumentation, telemetry, alerting, and incident response practices for AI systems. Observability is not just logging — it is the ability to ask new questions about system behavior without deploying new code.

> **Note:** Observability concepts are progressively introduced starting in Module 02. This module integrates and deepens them into a complete operational practice.

## Key Engineering Questions

- What telemetry do I need to diagnose a problem I haven't seen before?
- How do I trace a request through inference, retrieval, agent execution, and evaluation?
- How do I detect quality degradation before users report it?
- How do I define and enforce SLOs for AI systems?
- How do I attribute costs to specific features, models, or users?

## Prerequisites

- Module 04 (latency metrics, SLOs)
- Module 12 (evaluation — online quality measurement)
- Module 17 (cost attribution)

## Topics

### Distributed Tracing
- Trace structure: spans, attributes, parent-child relationships
- Tracing across system boundaries: inference, retrieval, agents, tools
- Correlation IDs: connecting user actions to system behavior

### AI-Specific Telemetry
- Scheduler telemetry: queue depth, batch composition, preemption events
- Retrieval telemetry: query latency, recall estimation, cache hit rates
- Agent traces: step-by-step agent execution with tool calls and decisions
- Model telemetry: token throughput, latency distributions, error rates
- Online quality signals: automated quality checks on production traffic

### Operational Practice
- Drift detection: monitoring for distribution shift in production
- SLO definition and enforcement for AI systems (latency, quality, cost)
- Alerting: what to alert on, thresholds, noise reduction
- Cost attribution: associating costs with features, teams, or customers

### Incident Response
- Incident response process for AI systems
- Runbooks: structured diagnostic procedures
- Post-incident review: learning from failures
- Connection to [incidents/](../incidents/) methodology

## Expected Artifacts

1. **Instrumentation design** — define the telemetry schema for a specific AI system
2. **Trace analysis** — trace a request through a multi-component system and identify bottlenecks
3. **SLO definition** — define SLOs and alerting for an AI application
4. **Runbook** — write a diagnostic runbook for a specific failure mode
5. **Engineering report** — observability architecture recommendation

## Exit Criteria

The learner can:
- Design instrumentation for an AI system that supports diagnosis of novel problems
- Implement distributed tracing across inference, retrieval, and agent components
- Define meaningful SLOs and alerting for AI systems
- Attribute costs to specific features and traffic classes
- Write and follow incident response runbooks

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Create
  solo: Relational
  dreyfus: Competent
```

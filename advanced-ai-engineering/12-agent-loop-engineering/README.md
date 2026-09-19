# Module 12 — Agent Loop Engineering

## Why This Module Exists

Before building durable agent runtimes (Module 14), the engineer must understand the fundamental agent loop: observe → reason → plan → act → observe. This module covers the core engineering patterns of agent execution — tool integration, action selection, error recovery, and the critical boundary between model decisions and system actions.

## Key Engineering Questions

- What are the fundamental agent loop patterns and when does each apply?
- How do I design tool interfaces that are robust to model misuse?
- How do I handle tool execution failures within the agent loop?
- How do I control agent behavior without over-constraining it?
- What is the right level of autonomy for a given task and risk level?

## Prerequisites

- Module 07 (model behavior and failure modes)
- Module 11 (context management — agents need context engineering)

## Topics

### Agent Loop Patterns
- ReAct: interleaved reasoning and acting
- Plan-then-execute: generate plan, then execute steps
- Reflexion: observe outcome, reflect, retry
- Multi-agent patterns: delegation, debate, specialization

### Tool Engineering
- Tool interface design: schemas, input validation, output formatting
- Tool discovery and selection
- Tool execution: synchronous, asynchronous, streaming
- Error handling: tool failures, timeouts, partial results
- Tool composition: chaining tools, parallel tool calls

### Control and Safety
- Action selection: when to act vs when to ask
- Autonomy levels: fully autonomous → human-in-the-loop → human-on-the-loop
- Guardrails: action filtering, output validation
- Escalation policies: when to stop and ask for help
- Budget and resource limits: token budgets, time budgets, action counts

### Observation and Feedback
- Observation parsing: structured vs unstructured tool outputs
- Memory within a loop: what to remember across iterations
- Stopping conditions: how the agent knows when to stop
- Loop pathology: infinite loops, oscillation, goal drift

## Expected Artifacts

1. **Agent loop implementation** — build a basic observe-reason-act loop with tool integration
2. **Tool failure analysis** — systematically test agent behavior under tool failures
3. **Autonomy level design** — design control policies for a specific use case
4. **Engineering report** — agent loop architecture for a given task and risk profile

## Exit Criteria

The learner can:
- Implement the core agent loop patterns and choose between them
- Design robust tool interfaces with proper error handling
- Set appropriate autonomy levels and escalation policies
- Diagnose loop pathologies (infinite loops, oscillation, drift)
- Reason about the cost of agent execution in tokens, time, and actions

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Apply → Evaluate
  solo: Relational
  dreyfus: Competent
```

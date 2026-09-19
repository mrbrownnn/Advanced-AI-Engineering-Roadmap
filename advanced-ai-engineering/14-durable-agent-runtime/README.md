# Module 11 — Durable Agent Runtime

## Why This Module Exists

LLM agents are stateful, long-running, non-deterministic processes with side effects. They fail in the middle, need to be resumed, fan out into parallel sub-tasks, and interact with humans. This module treats agent execution as a systems engineering problem: state management, durability, failure handling, and correctness.

## Key Engineering Questions

- How do I checkpoint and resume agent execution across failures?
- How do I handle partial failures in fan-out/join patterns?
- How do I make agent steps idempotent when the LLM is non-deterministic?
- How do I manage side effects (API calls, writes) in a resumable workflow?
- How do I version models and prompts without breaking running agents?

## Prerequisites

- Module 06 (model failure modes)
- Module 10 (context management)

## Topics

### Execution Models
- Execution graphs: DAG-based workflow composition
- State machines: explicit state transitions with guard conditions
- Hybrid models: graphs with state machine nodes

### Durability
- Checkpoint: serializing agent state for later resumption
- Resume: rehydrating agent state and continuing execution
- Idempotency: ensuring retried steps don't cause duplicate side effects
- Retries: exponential backoff, jitter, retry budgets
- Timeout and cancellation: bounding execution time

### Parallelism and Failure
- Fan-out / join: parallel sub-task execution with aggregation
- Partial failure: what happens when some branches fail?
- Transactional outbox: reliable side-effect delivery
- Compensation: undoing side effects when a workflow fails

### Human-in-the-Loop (HITL)
- Approval gates: pausing for human review
- Escalation: routing to humans on uncertainty
- Feedback integration: incorporating human corrections

### Agent-Specific Challenges
- Non-deterministic LLM replay: same prompt, different output on retry
- Model / prompt versioning: upgrading without breaking running workflows
- Side-effect boundaries: separating computation from effects

## Expected Artifacts

1. **Checkpoint/resume prototype** — implement basic agent state persistence and recovery
2. **Idempotency analysis** — identify and solve idempotency challenges in an agent workflow
3. **Failure injection test** — verify agent behavior under partial failures
4. **Engineering report** — agent runtime architecture for a specific use case

## Exit Criteria

The learner can:
- Design an agent runtime with checkpoint, resume, and failure recovery
- Implement idempotent agent steps despite non-deterministic LLM output
- Handle fan-out/join with partial failure recovery
- Manage side-effect boundaries in resumable workflows
- Reason about model/prompt versioning for running agents

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Create
  solo: Relational → Extended Abstract
  dreyfus: Competent
```

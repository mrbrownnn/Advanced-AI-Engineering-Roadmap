# Module 12 — Agent Loop Engineering

## Why This Module Exists

Before building durable agent runtimes (Module 14), the engineer must understand the fundamental agent loop: observe → reason → plan → act → observe. This module covers the core engineering patterns of agent execution — tool integration, action selection, error recovery, and the critical boundary between model decisions and system actions.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Build a minimal ReAct (Reason + Act) loop from scratch. Implement a strict tool schema parser and action dispatcher. Do not use LangChain.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the overhead of the agent loop (tokens consumed by reasoning vs tokens consumed by tool outputs). Track the latency of the loop iterations.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Provide the agent with a tool that intermittently fails, times out, or returns a schema violation. Break the loop into an infinite retry or hallucination state.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the loop pathology. Explain why the LLM failed to recover from the broken tool state (e.g., context window flooding with error messages).
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement a robust escalation policy, strict retry budgets, and context summarization during long loops.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the level of autonomy granted to the agent. When should the loop pause for human-in-the-loop (HITL) approval?
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the core execution loop in AutoGen or LangGraph to understand state transitions.

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

# Module 10 — Context and Memory Engineering

## Why This Module Exists

Context is the most constrained resource in an LLM system. Every token of context costs latency, compute, and money. This module covers the engineering of context selection, compression, and memory systems — deciding what goes into the context window and what stays outside it.

## Key Engineering Questions

- Given a task and limited context budget, what information is most valuable to include?
- How does context length affect quality, latency, and cost?
- What memory architecture is appropriate for my application?
- How do I exploit prefix stability and cache reuse?
- Where does long-context performance degrade and why?

## Prerequisites

- Module 03 (KV cache, memory costs)
- Module 08-09 (retrieval as an alternative to long context)

## Topics

### Context Engineering
- Context selection: choosing what to include given a budget
- Context budgeting: allocating tokens across system prompt, history, retrieval, user input
- Context compression: summarization, pruning, distillation
- Task utility vs context cost: is this context worth its tokens?

### Memory Architectures
- Working memory: current conversation state
- Episodic memory: past interaction summaries
- Semantic memory: structured knowledge stores
- Persistent application state: state that outlives a conversation

### Context Behavior
- Long-context degradation: quality loss with increasing context length
- Position effects: "lost in the middle" phenomenon
- Context isolation: preventing cross-contamination between users/sessions
- Prefix stability: keeping system prompt and common prefix constant for cache reuse

### Cache Integration
- KV cache reuse (connection to Module 03)
- Prefix caching for stable context portions
- Dynamic context vs cacheable context partitioning

## Expected Artifacts

1. **Context budgeting experiment** — measure quality as a function of context allocation strategy
2. **Position effect analysis** — quantify the "lost in the middle" effect for a specific model and task
3. **Memory architecture design** — propose a memory system for a specific application
4. **Engineering report** — context strategy recommendation with cost-quality analysis

## Exit Criteria

The learner can:
- Design a context budgeting strategy for a specific application
- Measure and mitigate long-context degradation
- Choose between retrieval, long context, and memory systems based on evidence
- Optimize context layout for KV cache reuse

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

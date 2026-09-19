# Module 10 — Context and Memory Engineering

## Why This Module Exists

Context is the most constrained resource in an LLM system. Every token of context costs latency, compute, and money. This module covers the engineering of context selection, compression, and memory systems — deciding what goes into the context window and what stays outside it.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a tiered memory system: short-term working memory (context) and long-term semantic memory (retrieval).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the token budget allocation. Quantify the 'lost in the middle' effect for the specific model being used.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Overflow the context window with perfectly relevant semantic memory, crowding out the system instructions and causing instruction-following failure.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the attention degradation. Explain why relevance does not equal utility in context window allocation.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement context compression (summarization or embedding distillation) and strict token budgeting.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a context layout architecture that optimizes for KV cache reuse (prefix stability) across turns.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the context management logic in a framework like MemGPT.

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

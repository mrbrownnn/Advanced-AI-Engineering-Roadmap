# Vision-Language Models (VLMs)

## Why This Track Exists

VLMs combine LLM reasoning with visual understanding. Engineering these systems requires managing heterogeneous modalities, massive token sequences, and cross-attention architectures.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a visual embedding projection layer that maps CLIP patch embeddings into an LLM's vocabulary space.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Calculate the token budget consumed by a 4K image. Measure the TTFT (Time To First Token) impact of visual context.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Create a multi-turn conversation with high-resolution images in every turn, breaking the KV cache limit.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the KV cache exhaustion. Explain how visual tokens fragment memory differently than text.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement visual token compression (e.g., Perceiver Resampler) or dynamic image resolution scaling.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a production choice between early fusion (native VLM) and late fusion (pipeline models).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the visual encoding and projection logic in the LLaVA repository.

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

# Module 01 — LLM Internals

## Why This Module Exists

You cannot engineer systems around models you do not understand. This module builds a precise mental model of the modern decoder-only transformer — every layer, every normalization, every attention variant — so that subsequent modules on inference, caching, and optimization connect to concrete architectural reality.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal decoder-only Transformer layer in PyTorch from scratch, focusing on matrix dimensions and memory layout.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Calculate the exact parameter count, FLOPs per token, and theoretical memory bandwidth requirement for the forward pass.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Increase the batch size until the forward pass OOMs. Predict the exact batch size where this will happen before running it.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the OOM by mapping the activation sizes at each layer. Explain which intermediate tensor consumed the most memory.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement gradient checkpointing or activation offloading to trade compute for memory.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend a hardware selection (e.g., A100 vs H100) based on the calculated arithmetic intensity of the model.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Trace the forward pass of `LlamaForCausalLM` in the Hugging Face Transformers library.

## Expected Artifacts
- **Engineering Report**: Document the entire BUILD → MEASURE → BREAK → DEFEND loop with empirical evidence.
- **Implementation Code**: The scratch code demonstrating the mechanism.

## Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

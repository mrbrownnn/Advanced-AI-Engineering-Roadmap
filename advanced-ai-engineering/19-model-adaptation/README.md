# Module 15 — Model Adaptation

## Why This Module Exists

Fine-tuning is one option in a space of adaptation strategies. Before training anything, the engineer must ask: would prompting, retrieval, context engineering, tool use, or model routing solve this problem more cheaply? This module teaches both the techniques of adaptation and the decision framework for when to use them.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Execute a full LoRA fine-tuning run on a custom dataset, modifying the attention weights.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the GPU memory required for training (activations + optimizer states). Compare the inference latency of the base model vs the LoRA adapter.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Train the model on a highly imbalanced dataset or use a learning rate that causes catastrophic forgetting of its base capabilities.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the catastrophic forgetting by evaluating on general benchmarks. Explain the weight degradation.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement data curriculum strategies, strict filtering, or DPO (Direct Preference Optimization) to align the model without destroying capabilities.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice to fine-tune versus using few-shot prompting or RAG, based on a total cost of ownership analysis.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the LoRA implementation in the PEFT library.

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

# Module 21 — Multimodal AI Systems

## Why This Module Exists

Modern AI engineering extends beyond text to vision, audio, and multimodal coordination. This module bridges the core engineering curriculum with the satellite tracks, focusing on the systems engineering challenges of multimodal ingestion, cross-modal embedding, token budget management, and multi-stream synchronization in production.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a multimodal pipeline that calculates the token budget for mixed image and text inputs.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the TTFT impact of a 4K image vs a 1080p image. Calculate the memory consumption of visual tokens in the KV cache.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Overwhelm the batch scheduler with heterogeneous inputs (short text vs high-res images) causing massive padding waste or OOM.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the batching inefficiency. Explain the KV cache fragmentation caused by the visual tokens.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement image tiling, token compression (e.g. cross-attention resamplers), or heterogeneous batching strategies.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend an architecture choice between a native multimodal model vs a decoupled pipeline (STT -> LLM -> TTS).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the visual token extraction logic in the LLaVA codebase.

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

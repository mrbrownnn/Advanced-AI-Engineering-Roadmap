# Speech AI & Audio Systems

## Why This Track Exists

Audio models (ASR, TTS) operate on continuous signals with strict real-time constraints. This requires mastering streaming inference, DSP fundamentals, and latency-sensitive scheduling.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a streaming chunked inference loop for an ASR model (e.g., Whisper).
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the Real-Time Factor (RTF). Calculate the exact latency overhead of chunking vs full-utterance processing.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Provide highly noisy audio or overlapping speech that breaks the model's alignment, causing hallucination loops.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the alignment failure. Explain how the lack of future context in streaming causes cascading errors.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement Voice Activity Detection (VAD) gating and beam search with a language model prior.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the choice of chunk size (latency vs accuracy) for a real-time conversational agent.
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the streaming inference logic in the faster-whisper repository.

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

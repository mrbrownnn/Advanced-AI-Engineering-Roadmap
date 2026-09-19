# World Models & Embodied AI

## Why This Track Exists

Embodied AI requires models to predict and act within physical environments. This track explores predictive coding, state space models, and reinforcement learning loops.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal recurrent predictive model (e.g., a simple RNN or SSM) that predicts the next frame in a sequence.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the training instability. Calculate the horizon length before the model's predictions diverge from reality.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Introduce stochasticity or out-of-distribution actions that cause the world model to hallucinate physically impossible states.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the state collapse. Explain why deterministic models fail in stochastic environments.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement a latent variable model (e.g., VAE) or a discrete bottleneck (VQ) to handle uncertainty.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend an architecture for a robotics planning system (e.g., model-based RL vs behavior cloning).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the architecture of DreamerV3 or similar open-source world models.

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

# Generative Media & Diffusion Systems

## Why This Track Exists

Diffusion models form the foundation of modern image and video generation. Understanding their inference bottlenecks, U-Net scaling, and condition mechanisms is critical for media AI engineers.

## The Engineering Mastery Loop

### 1. BUILD (Implementation)
- **Task**: Implement a minimal Denoising Diffusion Implicit Model (DDIM) sampling loop from scratch.
- **Goal**: Do not rely on high-level abstractions. Build the mechanism so you understand the fundamental constraints.

### 2. MEASURE (Quantitative Reasoning)
- **Task**: Measure the latency of the sampling loop. Calculate the exact FLOPs required for a 50-step generation vs a 20-step generation.
- **Goal**: Instrument the system. Establish a quantitative baseline and derive expected behavior before running the code.

### 3. BREAK (Falsification & Failure)
- **Task**: Push the resolution bounds until the U-Net activations OOM the GPU. Force a catastrophic failure of the spatial dimensions.
- **Goal**: Break the assumption that the system scales linearly or handles all inputs gracefully. Force a catastrophic failure.

### 4. EXPLAIN (Diagnosis)
- **Task**: Diagnose the memory explosion. Explain the quadratic memory scaling of self-attention at high resolutions.
- **Goal**: Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point using profiling or traces.

### 5. IMPROVE (Optimization)
- **Task**: Implement latent space diffusion (e.g., using a VAE) or chunked cross-attention to reduce memory footprint.
- **Goal**: Apply an optimization, adaptation, or architectural change based on evidence from the failure.

### 6. DEFEND (Production Trade-offs)
- **Task**: Defend the trade-off between inference speed (fewer steps) and output quality (FID score).
- **Goal**: Present the final engineering decision. Defend the trade-offs with empirical evidence and acknowledge remaining uncertainties.

## Source-Code Reading
- **Task**: Read the inference loop in the Diffusers library for Stable Diffusion.

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

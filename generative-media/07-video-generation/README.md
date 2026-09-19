# 07 — Video Generation

## Purpose

Understand temporal modeling in generative systems, extending 2D spatial diffusion to 3D spatio-temporal video architectures, and compute scaling.

## Topics

- Adding the temporal dimension: 3D convolutional blocks vs factored spatial-temporal attention
- Spatio-temporal autoencoders (3D VAEs) for video compression
- Temporal consistency, motion modeling, and physics priors
- Autoregressive frame extension vs joint spatio-temporal diffusion (e.g. Sora architecture concepts)
- Computational requirements: orders of magnitude increase in memory, FLOPs, and dataset scale compared to image generation

## Key Questions

- How do factored spatio-temporal attention layers balance cross-frame consistency with memory constraints?
- What causes temporal flicker or identity distortion across extended video generations?
- What infrastructure and parallelization strategies (e.g. sequence parallelism) are necessary to train and serve video generation models?

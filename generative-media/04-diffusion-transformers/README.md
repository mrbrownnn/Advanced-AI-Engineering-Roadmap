# 04 — Diffusion Transformers

## Purpose

Understand the shift from convolutional U-Net backbones to Diffusion Transformers (DiT) and the scaling laws of generative vision backbones.

## Topics

- DiT architecture: replacing convolutional downsampling/upsampling blocks with pure transformer blocks
- Patchifying latent representations: converting spatial latent feature maps into token sequences
- Conditioning mechanisms: adaptive layer normalization (adaLN) vs cross-attention
- Compute and parameter scaling laws in generative diffusion
- Memory bandwidth vs arithmetic intensity: DiT compute characteristics on modern GPUs

## Key Questions

- Why do transformers scale more predictably with compute in generative modeling compared to U-Nets?
- How does adaLN-Zero inject timestep and class conditioning directly into transformer layer norms?
- What are the serving latency and KV cache implications if DiT models are scaled to tens of billions of parameters?

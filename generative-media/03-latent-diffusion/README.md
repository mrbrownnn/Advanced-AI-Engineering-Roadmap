# 03 — Latent Diffusion

## Purpose

Understand how running diffusion processes in a compressed latent space of a pre-trained autoencoder makes high-resolution synthesis computationally tractable.

## Topics

- Two-stage architecture: perceptual compression (Autoencoder KL / VQ-GAN) + semantic diffusion
- Latent downsampling factor (e.g. $f=8$) and reduction in spatial computational complexity ($64\times$ fewer elements)
- Cross-attention mechanisms for multi-modal conditioning (text prompts, CLIP embeddings)
- Classifier-Free Guidance (CFG): training unconditional/conditional branches, trade-off between fidelity and diversity
- Stable Diffusion architectural anatomy

## Key Questions

- How does separating perceptual compression from generative diffusion reduce memory and FLOP overhead?
- How does Classifier-Free Guidance steer samples toward conditioning prompts without explicit classifier gradients?
- What visual artifacts emerge when CFG scale is set too high (e.g. oversaturation, burn-in)?

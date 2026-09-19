# 02 — Diffusion

## Purpose

Understand the core diffusion probabilistic framework: forward Gaussian corruption, reverse iterative denoising, and objective parameterizations.

## Topics

- Forward noising process: Markov chain, variance schedules (linear, cosine), closed-form sampling at step $t$
- Reverse denoising process: parameterizing neural networks to predict noise ($\epsilon$-prediction) or original image ($x_0$-prediction)
- Training objectives: simplified L2 loss between true and predicted noise
- Sampling mechanics: Ancestral sampling (DDPM) vs deterministic accelerated sampling (DDIM)
- Connection to score-based generative modeling and Langevin dynamics

## Key Questions

- Why does predicting the added noise $\epsilon$ yield more stable training than predicting the clean image directly?
- How does DDIM enable deterministic sampling with significantly fewer inference steps?
- What causes the quadratic or high linear inference compute cost in raw pixel-space diffusion?

# 02 — Latent World Models

## Purpose

Understand how modern world models operate inside learned, compact latent spaces rather than raw high-dimensional observation space.

## Topics

- Latent dynamics: compressing observations $o_t \to z_t$ via VAE/encoders, predicting forward dynamics in $z$-space
- Dreamer architecture family (DreamerV1, V2, V3): Recurrent State-Space Models (RSSM), deterministic and stochastic states
- Action-conditioned latent transitions
- Actor-critic training purely inside "imagined" latent trajectories
- Robustness across diverse domains (discrete/continuous control, fixed hyperparameter scaling)

## Key Questions

- Why is predicting dynamics in a compact latent space vastly superior to predicting future video frames directly?
- How does the combination of deterministic (GRU/RNN) and stochastic latent variables prevent information loss?
- What enables DreamerV3 to achieve superhuman performance across Atari, Minecraft, and robotics without per-domain tuning?

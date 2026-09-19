# 05 — Flow Matching

## Purpose

Understand Continuous Normalizing Flows, flow matching objectives, and straight-line probability paths as an efficient alternative to stochastic diffusion.

## Topics

- Continuous Normalizing Flows (CNF) and Neural Ordinary Differential Equations (ODEs)
- Flow Matching: regressing vector fields that transport simple prior noise to empirical data distributions
- Optimal Transport displacement interpolations and straight trajectories
- Rectified Flow: reflowing trajectories to minimize curvature and achieve 1-to-few-step generation
- Comparing Flow Matching with traditional diffusion: training stability, ODE solver step efficiency

## Key Questions

- How does learning straight probability paths reduce the number of numerical ODE solver steps required during sampling?
- Why does flow matching avoid the complex variance scheduling and SDE formulations of traditional diffusion?
- What performance advantages do flow-matching architectures (e.g. Flux, Stable Diffusion 3) exhibit in production?

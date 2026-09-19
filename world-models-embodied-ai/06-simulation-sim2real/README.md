# 06 — Simulation & Sim-to-Real

## Purpose

Understand physics simulation as a synthetic data engine and the critical methodologies for overcoming the reality gap.

## Topics

- Physics simulation engines: MuJoCo, Isaac Sim/Gym, Genesis, PyBullet
- Massively parallel GPU-accelerated simulation (tens of thousands of parallel environments)
- The Reality Gap (Sim-to-Real): differences in friction, sensor noise, latency, contact mechanics
- Domain Randomization (DR): randomizing mass, friction, restitution, and visual textures during training
- System Identification and residual policy learning
- Compute costs: trade-offs between physical data collection and synthetic GPU simulation

## Key Questions

- Why is GPU-accelerated physics simulation essential for training embodied policies at scale?
- What physical phenomena remain notoriously difficult to simulate accurately (e.g. deformable objects, fluids)?
- How does domain randomization guarantee zero-shot transfer without overfitting to the simulator's physics approximations?

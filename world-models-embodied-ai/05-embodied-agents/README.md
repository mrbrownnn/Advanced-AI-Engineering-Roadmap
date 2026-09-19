# 05 — Embodied Agents

## Purpose

Understand the systems and architectural engineering challenges of deploying AI agents in the real, physical, non-deterministic world.

## Topics

- Physical agent loops: sensor polling, state estimation, high-level planning, low-level trajectory execution
- Real-time frequency hierarchy: 5Hz semantic planning vs 50Hz action generation vs 500Hz joint impedance control
- Manipulation vs Navigation: spatial SLAM, point-goal navigation, mobile manipulation
- Safety boundaries: collision avoidance, emergency stops, fail-safe actuation contracts
- Hardware heterogeneity and actuation latency

## Key Questions

- How do embodied systems bridge the frequency mismatch between slow LLM/VLM reasoning and fast motor control?
- What architectural isolation prevents non-deterministic foundation model outputs from damaging physical hardware?
- How do physical world failures differ fundamentally from software-sandbox agent failures?

# 04 — Vision-Language-Action Models

## Purpose

Understand how vision, natural language instructions, and robot action spaces are unified into foundation model architectures (VLA).

## Topics

- Vision-Language-Action (VLA) paradigm: conditioning on camera feeds + text goals to output end-effector actions
- Tokenizing continuous action spaces: discretizing robot joint velocities and Cartesian coordinates into vocab tokens
- Architecture anatomy: RT-1, RT-2, OpenVLA
- Transfer learning: transferring web-scale visual and semantic knowledge into robotic manipulation policies
- Generalization to unseen objects, instructions, and physical environments

## Key Questions

- How are continuous multi-axis robotic actions represented as discrete tokens in an autoregressive vocabulary?
- What capabilities transfer from pre-trained VLMs to physical robot control?
- What are the inference latency constraints when running a multi-billion parameter VLA in a real-time control loop?

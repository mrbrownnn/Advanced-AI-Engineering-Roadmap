# 03 — Planning

## Purpose

Understand search and decision-making using learned forward models: trajectory optimization, Model Predictive Control (MPC), and policy distillation.

## Topics

- Model Predictive Control (MPC): plan $H$ steps into the future, execute first action, observe state, replan
- Shooting methods and Cross-Entropy Method (CEM) for trajectory optimization
- Monte Carlo Tree Search (MCTS) combined with neural evaluation (MuZero / AlphaZero principles)
- Policy distillation: amortizing online planning compute into an instant reactive neural policy
- Parallels to LLM test-time compute: reasoning trees and verification loops as planning

## Key Questions

- How does receding-horizon replanning (MPC) provide robustness against imperfect world model predictions?
- When is online planning (e.g. MCTS / CEM) worth the runtime latency cost vs executing an amortized policy network?
- How does search in world models conceptually parallel test-time compute scaling in LLM reasoning?

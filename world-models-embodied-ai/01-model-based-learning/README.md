# 01 — Model-Based Learning

## Purpose

Understand model-based reinforcement learning intuition: learning an internal model of environment transition dynamics to simulate and plan actions before execution.

## Topics

- Model-free vs model-based reinforcement learning: policy search vs environment modeling
- Transition models: predicting next state $s_{t+1}$ given current state $s_t$ and action $a_t$
- Reward prediction models: predicting immediate and discounted return
- Sample efficiency: why learning an environment model reduces required physical environment interactions
- Compounding model errors: why rollout inaccuracies multiply over long time horizons

## Key Questions

- Under what circumstances is model-based RL substantially more sample efficient than model-free RL?
- How does model error compounding limit the viable horizon for imagination-based rollouts?
- What distinguishes an environment transition model from an autoregressive next-token predictor?

# World Models & Embodied AI — 80/20 Architectural Literacy

An architectural literacy track covering world models, planning, and embodied AI. The goal is to understand how AI systems that interact with the physical world differ from text/retrieval systems, without turning this into a robotics engineering curriculum.

## Five Fundamental Questions

1. **Representation**: Observations → latent world state; actions; rewards
2. **Architecture**: Latent dynamics models (Dreamer), vision-language-action models, embodied foundation models
3. **Learning**: Model-based RL, action-conditioned prediction, imitation learning
4. **Evaluation**: Prediction accuracy, task completion, sim-to-real transfer gap
5. **Production**: Simulation cost, sim-to-real gap, safety, real-time control, embodied deployment

## Mental Model

```
Observation → Representation → World State → Predict Future → Plan → Action → New Observation
```

## Modules

| Module | Focus |
|--------|-------|
| [01 — Model-Based Learning](01-model-based-learning/) | Model-based RL intuition, prediction for planning |
| [02 — Latent World Models](02-latent-world-models/) | Latent dynamics, Dreamer-style systems |
| [03 — Planning](03-planning/) | Planning with learned models, search, policies |
| [04 — Vision-Language-Action](04-vision-language-action/) | VLA models, embodied foundation models |
| [05 — Embodied Agents](05-embodied-agents/) | Robot manipulation, navigation, real-world deployment |
| [06 — Simulation & Sim-to-Real](06-simulation-sim2real/) | Simulation environments, domain randomization, transfer |

## Connection to Core Track

```
World Models → reasoning and planning (Module 06)
             → agent loops (Module 12)
             → multimodal perception (Module 21)
             → evaluation (Module 15)
             → safety (Module 18)
```

## The Learner Should Finish Understanding

- What a world model is and how it differs from an LLM/VLM
- How prediction interacts with action in a control loop
- Why simulation is essential and what the sim-to-real gap means
- Why world models matter for the future of autonomous AI

## Recommended Timing

Month 5, alongside core Modules 14–16.

## Competency Target

```yaml
competency:
  bloom: Understand → Apply → Analyze
  solo: Multistructural → Relational
  dreyfus: Advanced Beginner
```

## Status

🏗️ Skeleton established. Content to be developed.

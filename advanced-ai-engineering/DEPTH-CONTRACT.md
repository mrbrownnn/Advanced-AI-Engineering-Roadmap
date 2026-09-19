# Depth Contract

## Purpose
This document defines what it means for a module in this curriculum to be "advanced." 

The Advanced AI Engineering Roadmap is **not** a structured list of topics. It is a rigorous mastery-oriented engineering curriculum. 

## The Core Rule: Topic Coverage vs Mastery
- **Topic Coverage** (Unacceptable): "Learn about KV Caches", "Understand LoRA", "Read about FlashAttention". This treats knowledge as information consumption.
- **Mastery Specification** (Required): Define observable, verifiable engineering behaviors. 

## Definition of a Genuinely Advanced Module
A module is genuinely advanced when the learner is required to demonstrate an appropriate subset of the following capabilities, backed by empirical engineering evidence.

A module does NOT need every item, but it must include the subset appropriate to its engineering domain:

1. **Mechanisms and Internals**
   - I can explain why the mechanism exists.
   - I can explain how it works internally.
   - I can locate the mechanism in a real implementation (e.g., source code reading).

2. **Quantitative Reasoning**
   - I can derive or estimate its important behavior (e.g., FLOPs, memory bounds).
   - I can predict what happens when the workload changes.

3. **Experimentation and Evidence**
   - I can instrument the system.
   - I can design a valid experiment.
   - I can benchmark it rigorously.
   - I can distinguish observation from interpretation.

4. **Failure and Falsification**
   - I can create a workload that intentionally breaks an assumption.
   - I can diagnose the resulting behavior.
   - I can falsify my own explanation.

5. **Production Engineering**
   - I can improve the system based on evidence.
   - I can identify trade-offs.
   - I can compare alternatives.
   - I can defend an architecture decision.
   - I can connect the mechanism to neighboring subsystems.
   - I can identify what remains uncertain.

## Scope Boundaries
To maintain the integrity of the depth contract:
- **No shallow API tutorials**: Wrapping an API does not demonstrate mastery of the underlying mechanism.
- **No framework idolatry**: We learn the mechanism, not just the syntax of a specific library.
- **Do not confuse completion with seniority**: Completing a module does not grant a professional title. It provides the opportunity to practice the responsibilities of that level.

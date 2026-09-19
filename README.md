# Intensive AI Engineer

> A competency-driven journey through modern AI engineering, from models and data to agents, inference systems, and multimodal AI.

## Program Architecture

This program has three layers:

### Layer 1 — Core / Deep Track

**[advanced-ai-engineering/](advanced-ai-engineering/)** — The 26-week professional curriculum developing deep competency across the full AI engineering chain: from scientific experimentation and model internals through inference systems, agents, evaluation, security, and model-system co-design.

### Layer 2 — Satellite / 80-20 Tracks

Architectural literacy across the broader AI landscape:

| Track | Purpose |
|-------|---------|
| [vision-language-models/](vision-language-models/) | Image understanding, VLM architectures, multimodal serving |
| [speech-ai/](speech-ai/) | Audio representation, ASR, TTS, voice agent systems |
| [generative-media/](generative-media/) | Diffusion, flow matching, image and video generation |
| [world-models-embodied-ai/](world-models-embodied-ai/) | World models, planning, embodied agents, sim-to-real |

**80/20 principle**: Learn the smallest set of concepts that enables reading new papers, analyzing production systems, and identifying what additional knowledge would be needed to specialize.

Satellite tracks are **NOT** superficial keyword surveys. They reach architectural literacy.

### Layer 3 — Paper Radar

**[paper-library/](paper-library/)** — Continuous literature tracking across all AI domains. Curated, classified, and connected to curriculum modules.

---

## The Engineering Evidence Philosophy

This program rejects "topic coverage." Understanding an API or reading a paper is insufficient. The curriculum is governed by a strict [Depth Contract](advanced-ai-engineering/DEPTH-CONTRACT.md), demanding empirical engineering evidence for every completed module.

### Central Engineering Loop

Defined fully in the [Engineering Learning Protocol](advanced-ai-engineering/ENGINEERING-LEARNING-PROTOCOL.md):

```
BUILD → MEASURE → BREAK → EXPLAIN → IMPROVE → DEFEND
```

A system merely "working" is not considered completion. You must break it, diagnose the failure, and defend the trade-offs of your fix.

### Central Literature Loop

```
READ → REPRODUCE → CHALLENGE → CONNECT → APPLY
```

A paper merely "read" is not considered understood. You must reproduce its core mechanism and identify where it fails.

---

## What This Is NOT

- ❌ An attempt to become an expert in every AI subfield
- ❌ A beginner AI course
- ❌ A framework tutorial collection
- ❌ A paper-reading list
- ❌ An LLM application tutorial
- ❌ An interview-cramming repository

## What This IS

**Deep expertise** in the AI Engineering core — inference, serving, agents, evaluation, security, economics, reliability

**+ Architectural literacy** across the broader AI landscape — vision, speech, generative media, world models

---

## The Learner Should Repeatedly Answer

- What did I assume?
- What did I predict?
- What did I measure?
- How reliable is the measurement?
- What broke?
- Why did it break?
- What evidence supports the explanation?
- What alternatives exist?
- What trade-off did I choose?
- Under what workload would the decision change?
- What is the rollback condition?

## Repository Structure

```
Intensive AI Engineer/
│
├── README.md                       ← this file
├── ROADMAP.md                      ← 26-week schedule with satellite integration
├── COMPETENCY.md                   ← SFIA, Bloom, SOLO, Dreyfus framework
│
├── advanced-ai-engineering/        ← CORE: 25-module deep track
│   ├── DEPTH-CONTRACT.md           ← Defines the standard for mastery vs topic coverage
│   └── ENGINEERING-LEARNING-PROTOCOL.md ← The rigorous loops used to execute modules
│
├── vision-language-models/         ← SATELLITE: VLM 80/20
├── speech-ai/                      ← SATELLITE: Speech 80/20
├── generative-media/               ← SATELLITE: Generative media 80/20
├── world-models-embodied-ai/       ← SATELLITE: World models 80/20
│
├── paper-library/                  ← Shared paper index across all tracks
├── templates/                      ← Shared templates
└── references/                     ← Quick reference material
```

## Getting Started

1. Read [ROADMAP.md](ROADMAP.md) for the 26-week schedule
2. Read [COMPETENCY.md](COMPETENCY.md) for the competency framework
3. Read [DEPTH-CONTRACT.md](advanced-ai-engineering/DEPTH-CONTRACT.md) and [ENGINEERING-LEARNING-PROTOCOL.md](advanced-ai-engineering/ENGINEERING-LEARNING-PROTOCOL.md) to understand the required rigor.
4. Start with [advanced-ai-engineering/](advanced-ai-engineering/) Module 00
5. Satellite tracks integrate starting Month 2

## Status

🏗️ **Curriculum Depth Upgrade Completed.** The repository architecture and mastery contracts are in place. Content is being progressively developed over 6 months.

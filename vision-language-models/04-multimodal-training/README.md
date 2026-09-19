# 04 — Multimodal Training

## Purpose

Understand the training stages and data pipelines required to build functional multimodal foundation models.

## Topics

- Stage 1: Feature alignment pre-training (frozen vision encoder + frozen LLM, trainable projector)
- Stage 2: Visual instruction tuning (fine-tuning LLM + projector on conversational and QA data)
- Stage 3: Full end-to-end training and unfreezing the vision encoder
- Data pipelines: web-scraped pairs, synthetic captioning, instruction datasets (ShareGPT4V, LLaVA-Instruct)
- Mitigating catastrophic forgetting of pure language capabilities

## Key Questions

- Why is training staged rather than end-to-end from scratch?
- How does synthetic data generation contribute to multimodal alignment?
- What proportion of text-only data is necessary during multimodal training to prevent language degradation?

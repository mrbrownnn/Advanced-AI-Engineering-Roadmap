# 03 — VLM Architectures

## Purpose

Understand the structural blueprints connecting vision encoders to autoregressive language model decoders.

## Topics

- Encoder-Projector-LM paradigm: Vision Encoder (ViT) → Projection Layer → Language Model
- Projection variants: Linear projection, MLP, Perceiver Resampler, Q-Former
- Cross-attention architectures (Flamingo) vs prefix-token architectures (LLaVA)
- Unified native multimodal transformers (e.g. Chameleon, Gemini architectural concepts)
- Parameter distribution: vision encoder vs projector vs language backbone

## Key Questions

- What is the difference between injecting vision as prefix tokens vs cross-attention keys/values?
- Why did simple MLP projectors largely replace complex cross-attention resamplers in modern open VLMs?
- How do visual tokens participate in standard causal attention masks?

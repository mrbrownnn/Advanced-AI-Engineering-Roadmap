# 03 — Text-to-Speech

## Purpose

Understand modern Text-to-Speech (TTS) pipelines from text normalization to neural vocoding and autoregressive/diffusion audio generation.

## Topics

- The classical vs modern TTS pipeline: text normalization → acoustic model / latent generation → neural vocoder
- Autoregressive speech synthesis (e.g. VALL-E, Bark) using audio codecs
- Diffusion and Flow-Matching TTS models (e.g. Voicebox, Matcha-TTS)
- Neural vocoders: HiFi-GAN, BigVGAN
- Zero-shot voice cloning and speaker conditioning

## Key Questions

- How do audio codec language models treat voice generation as language modeling?
- Why are flow-matching and diffusion models emerging as strong competitors to autoregressive TTS?
- What causes unnatural prosody, hallucinations, or speaker drift in zero-shot cloning?

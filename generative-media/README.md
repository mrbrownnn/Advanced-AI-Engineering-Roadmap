# Generative Media — 80/20 Architectural Literacy

An architectural literacy track covering generative image and video models. The goal is to understand the evolution from VAEs through diffusion to flow matching, and the engineering implications of serving generative models.

## Five Fundamental Questions

1. **Representation**: Pixel space vs latent space; noise → signal; text/image conditioning
2. **Architecture**: VAE, GAN (historical context), U-Net diffusion, Diffusion Transformer (DiT), flow matching
3. **Learning**: Reconstruction loss, adversarial loss, denoising score matching, flow matching objectives
4. **Evaluation**: FID, CLIP score, human preference, prompt adherence
5. **Production**: Sampling steps → latency, memory, GPU compute, quality/speed trade-offs, serving at scale

## Modules

| Module | Focus |
|--------|-------|
| [01 — Generative Model Foundations](01-generative-model-foundations/) | VAE, GAN (historical), latent variables |
| [02 — Diffusion](02-diffusion/) | DDPM, denoising, noise schedules |
| [03 — Latent Diffusion](03-latent-diffusion/) | Stable Diffusion, latent space generation |
| [04 — Diffusion Transformers](04-diffusion-transformers/) | DiT, scaling transformers for generation |
| [05 — Flow Matching](05-flow-matching/) | Continuous normalizing flows, flow matching |
| [06 — Image Generation](06-image-generation/) | Conditioning, guidance, controlability |
| [07 — Video Generation](07-video-generation/) | Temporal modeling, video diffusion |

## Evolutionary Arc

```
VAE → GAN → Diffusion (DDPM) → Latent Diffusion → DiT → Flow Matching
  (historical context)     (current production)      (frontier)
```

Historical techniques are clearly marked as context, not current production defaults.

## Connection to Core Track

```
Generative Media → GPU compute (Module 02)
                 → serving (Module 04)
                 → economics (Module 22)
                 → multimodal systems (Module 21)
```

## Recommended Timing

Month 4, alongside core Modules 09–11.

## Competency Target

```yaml
competency:
  bloom: Understand → Apply → Analyze
  solo: Multistructural → Relational
  dreyfus: Advanced Beginner
```

## Status

🏗️ Skeleton established. Content to be developed.

# 06 — Image Generation

## Purpose

Understand practical high-fidelity image synthesis, spatial conditioning (ControlNet, IP-Adapter), inpainting, and serving economics.

## Topics

- Conditioned generation pipelines: text-to-image, image-to-image, inpainting, outpainting
- Structural conditioning: ControlNet (edges, depth, pose, segmentation maps)
- Style and identity conditioning: IP-Adapter, LoRA fine-tuning for generative models
- Evaluation metrics: Fréchet Inception Distance (FID), Inception Score (IS), CLIP Score, human preference benchmarks (ImageReward)
- Production serving: step distillation (LCM, SDXL-Turbo), FP8/INT8 quantization, TensorRT-LLM / TensorRT acceleration

## Key Questions

- How does ControlNet lock structural spatial features without corrupting pre-trained generative priors?
- Why is FID increasingly supplemented or replaced by multimodal vision-language judges (e.g. CLIP Score, ImageReward)?
- What is the serving throughput and latency difference between full 50-step diffusion and 4-step distilled models?

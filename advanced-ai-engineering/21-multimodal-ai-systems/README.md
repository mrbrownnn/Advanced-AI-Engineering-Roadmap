# Module 21 — Multimodal AI Systems

## Why This Module Exists

Modern AI engineering extends beyond text to vision, audio, and multimodal coordination. This module bridges the core engineering curriculum with the satellite tracks, focusing on the systems engineering challenges of multimodal ingestion, cross-modal embedding, token budget management, and multi-stream synchronization in production.

## Key Engineering Questions

- How do visual and audio tokens impact KV cache sizing, TTFT, and serving memory bandwidth?
- How do I design pipelines that ingest, compress, and route heterogeneous media streams (video, audio, text) without introducing latency bottlenecks?
- How do cross-attention and projection layers affect distributed model partitioning (tensor parallelism vs pipeline parallelism)?
- How do I evaluate multimodal outputs where errors can originate in perception, reasoning, or cross-modal alignment?

## Prerequisites

- Module 02 (Inference & GPU Systems Fundamentals)
- Module 03 (KV Cache Engineering)
- Module 04 (Serving, Scheduling & Capacity Engineering)
- Satellite tracks (Vision-Language Models, Speech AI)

## Topics

### Multimodal Token Mechanics & System Costs
- Image patchification and visual token count calculation (ViT, CLIP, SigLIP)
- Audio frame representations and token dilation
- Memory and computational overhead of high-resolution image and video inputs
- Token compression techniques (spatial pooling, cross-attention resamplers)

### Serving Multimodal Models
- Heterogeneous batching: handling variable image resolutions and audio lengths
- Memory footprint: KV cache allocation for large image/video context prefixes
- Prefill vs decode balance when prefill consists of thousands of visual tokens

### Cross-Track Synthesis & End-to-End Pipelines
- Real-time voice-to-voice architectures: STT → LLM → TTS vs native speech-to-speech
- Multimodal document understanding (OCR-free vs OCR-augmented)
- Integrating satellite track principles into the core AI Runtime Platform

## Expected Artifacts

1. **Multimodal token and latency profiler** — measurement tool computing TTFT and memory cost as a function of image resolution and audio duration
2. **Dynamic resolution handler** — preprocessing and serving module implementing token-efficient image tiling
3. **Engineering report** — architectural analysis comparing decoupled multimodal pipelines vs unified native multimodal models

## Exit Criteria

The learner can:
- Accurately calculate and optimize the memory and compute footprint of multimodal inputs
- Architect serving systems capable of handling mixed-modality workloads without degrading text throughput
- Design end-to-end multimodal systems that synthesize text, vision, and speech components

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Analyze → Evaluate
  solo: Relational
  dreyfus: Competent
```

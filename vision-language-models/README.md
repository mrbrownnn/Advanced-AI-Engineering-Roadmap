# Vision-Language Models — 80/20 Architectural Literacy

An architectural literacy track covering vision-language models. The goal is to understand how vision is integrated into LLMs well enough to read papers, evaluate production systems, and identify where the domain connects to the core AI engineering stack.

## Five Fundamental Questions

1. **Representation**: How are images converted into tokens that an LLM can process? (Patches, ViT, learned features)
2. **Architecture**: How are vision and language combined? (Encoder-projector-LM, cross-attention, native multimodal)
3. **Learning**: How are multimodal models trained? (Pre-training on image-text pairs, instruction tuning, alignment)
4. **Evaluation**: How do we evaluate VLM outputs? (Hallucination benchmarks, grounding, OCR/document accuracy)
5. **Production**: What are the serving and systems costs of visual tokens? (Memory, compute, TTFT, throughput)

## Modules

| Module | Focus |
|--------|-------|
| [01 — Vision Representation](01-vision-representation/) | Patches, ViT, visual features, resolution strategies |
| [02 — Contrastive Learning](02-contrastive-learning/) | CLIP, SigLIP, shared embedding spaces |
| [03 — VLM Architectures](03-vlm-architectures/) | Encoder-projector-LM (LLaVA), cross-attention, unified models |
| [04 — Multimodal Training](04-multimodal-training/) | Image-text pre-training, instruction tuning, data mixtures |
| [05 — Grounding, Documents, OCR](05-grounding-documents-ocr/) | Document understanding, OCR, bounding box prediction |
| [06 — VLM Evaluation](06-vlm-evaluation/) | Hallucination (POPE), grounding, multimodal benchmarks |
| [07 — VLM Serving](07-vlm-serving/) | Visual tokens in KV cache, serving memory, TTFT, throughput |

## Connection to Core Track

```
VLM → visual tokens in KV cache (Module 03)
    → serving memory and TTFT (Module 02, 04)
    → multimodal retrieval / RAG (Module 09, 10)
    → evaluation of multimodal outputs (Module 15)
    → cost modeling for visual tokens (Module 22)
    → unified multimodal systems (Module 21)
```

## Recommended Timing

Month 2 of the program, alongside core Modules 03–05.

## Competency Target

```yaml
competency:
  bloom: Understand → Apply → Analyze
  solo: Multistructural → Relational
  dreyfus: Advanced Beginner
```

## Status

🏗️ Skeleton established. Content to be developed.

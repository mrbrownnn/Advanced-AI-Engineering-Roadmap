# 07 — VLM Serving

## Purpose

Understand the systems engineering, memory consumption, and serving constraints introduced by visual tokens in production.

## Topics

- Visual tokens in the KV cache: 576 to 2,000+ tokens per image
- Time to First Token (TTFT) impact of large visual prefill
- Memory footprint: HBM requirements for multi-image and high-res requests
- Chunked prefill and scheduling policies for mixed text-vision traffic
- Token compression at inference: dynamic token dropping, pooling, pruning
- Cost modeling: image input pricing vs text pricing

## Key Questions

- Why do visual tokens disproportionately degrade serving capacity in shared LLM engines?
- How can visual token pruning reduce KV cache pressure without sacrificing fine-grained reasoning?
- What scheduling policy prevents a batch of high-resolution images from stalling conversational decodes?

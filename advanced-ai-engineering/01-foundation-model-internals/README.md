# Module 01 — LLM Internals

## Why This Module Exists

You cannot engineer systems around models you do not understand. This module builds a precise mental model of the modern decoder-only transformer — every layer, every normalization, every attention variant — so that subsequent modules on inference, caching, and optimization connect to concrete architectural reality.

## Key Engineering Questions

- Given a model card, can I compute the parameter count from first principles?
- Given a model and sequence length, can I estimate the FLOP count for a forward pass?
- Given a model and batch of sequences, can I compute the KV cache memory requirement?
- How does the choice of attention mechanism (MHA/MQA/GQA/MLA) affect memory and compute?
- How does MoE change the parameter-vs-compute relationship?

## Prerequisites

- Module 00 (experimental methodology)
- Linear algebra basics (matrix multiplication, dimensions)
- PyTorch fundamentals (tensors, autograd not required)

## Topics

### Decoder Architecture
- Full decoder-only transformer walkthrough
- Token embedding and un-embedding
- Residual stream as the central data flow
- Layer structure: attention → FFN with residual connections

### Normalization and Activation
- RMSNorm: why it replaced LayerNorm in modern architectures
- SwiGLU: gated linear unit activation in FFN

### Positional Encoding
- RoPE: rotary position embedding mechanism and properties

### Attention Variants
- Multi-Head Attention (MHA): full KV heads per attention head
- Multi-Query Attention (MQA): single KV head shared across all query heads
- Grouped-Query Attention (GQA): KV heads shared across groups of query heads
- Multi-head Latent Attention (MLA): compressed KV via low-rank projection

### Mixture of Experts
- MoE layer structure: router + expert FFNs
- Sparse activation: active parameters vs total parameters
- Load balancing and routing

### Accounting
- Parameter counting: embedding, attention, FFN, output
- FLOP estimation: per-layer, per-token, per-sequence
- KV cache accounting: bytes per token per layer as a function of attention variant

## Expected Artifacts

1. **Parameter accounting spreadsheet** — compute parameter counts for 2-3 real models from architecture specs
2. **FLOP estimation** — estimate FLOPs for prefill and decode for a specific model and workload
3. **KV cache calculator** — compute KV memory requirements as a function of batch size, sequence length, and attention variant
4. **Architecture annotated diagram** — trace data flow through a specific model

## Exit Criteria

The learner can:
- Given a model's architecture specification, compute parameter count, FLOP estimate, and KV cache size without reference materials
- Explain how GQA reduces KV memory relative to MHA and the quality trade-off
- Explain how MoE decouples parameter count from per-token compute cost
- Trace a token through the full forward pass from embedding to logits

## Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze
  solo: Relational
  dreyfus: Advanced Beginner → Competent
```

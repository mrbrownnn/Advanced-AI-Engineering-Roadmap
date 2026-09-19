# Module 03 — KV Cache Engineering

## Why This Module Exists

The KV cache is the single largest source of memory consumption in LLM serving and the primary constraint on concurrency. How you manage KV memory determines how many requests you can serve simultaneously, how you handle memory pressure, and how you exploit sharing across requests. This is where OS-level systems thinking meets ML inference.

## Key Engineering Questions

- How much KV memory does a specific model/workload require?
- Why does naive contiguous allocation waste memory, and what fragmentation patterns emerge?
- How does PagedAttention solve fragmentation, and what are the overhead trade-offs?
- When does prefix caching help, and when does it hurt?
- What are the memory-quality-latency trade-offs of KV quantization?
- How do eviction policies interact with scheduling and SLOs?

## Prerequisites

- Module 01 (KV accounting, attention variants)
- Module 02 (memory hierarchy, HBM)

## Topics

### KV Lifecycle
- Allocation at prefill, growth during decode, deallocation at completion
- Contiguous allocation: simplicity, waste, fragmentation

### PagedAttention
- Block-based non-contiguous storage
- Block tables: mapping logical to physical blocks
- Reference counting for shared blocks
- Copy-on-write for parallel sampling (beam search, best-of-N)

### Memory Management
- Preemption: swapping vs recomputation when memory is exhausted
- Eviction policies: LRU, frequency-based, priority-based
- Memory fragmentation patterns and mitigation

### Caching Strategies
- Prefix caching: sharing KV for common system prompts
- Radix caching: tree-based sharing for branching conversations
- Cache-aware routing: directing requests to nodes with relevant cached prefixes

### KV Optimization
- KV quantization: reducing bytes per KV element (INT8, INT4)
- KV offloading: moving cold KV blocks to CPU/NVMe
- Interaction with attention variants (GQA reduces KV size, MLA compresses KV)

## Expected Artifacts

1. **KV memory calculator** — compute KV requirements for different models, batch sizes, and sequence lengths
2. **Fragmentation analysis** — demonstrate and measure fragmentation under different allocation strategies
3. **Prefix caching investigation** — measure when prefix caching helps and when it doesn't
4. **Engineering report** — recommend a KV management strategy for a given workload

## Exit Criteria

The learner can:
- Given an unfamiliar serving workload, estimate KV memory pressure, identify likely fragmentation behaviour, select an allocation strategy, and validate the prediction experimentally
- Explain the PagedAttention block table mechanism and its overhead
- Analyze prefix caching effectiveness for a specific workload pattern
- Reason about KV quantization trade-offs for a given quality and latency budget

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

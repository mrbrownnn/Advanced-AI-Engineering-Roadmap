# Module 16 — Distributed Inference

## Why This Module Exists

Large models don't fit on a single GPU. This module covers the parallelism strategies and communication primitives needed to serve models across multiple GPUs and nodes. The goal is architectural understanding and capacity reasoning, NOT an NCCL implementation course.

## Key Engineering Questions

- Which parallelism strategy (TP, PP, DP, EP) is appropriate for my model and hardware?
- What are the communication costs and how do they scale?
- How does the interconnect (NVLink, PCIe, InfiniBand) constrain my options?
- How do I place MoE experts across devices?
- When does P/D disaggregation make sense?

## Prerequisites

- Module 02 (GPU execution, memory hierarchy)
- Module 03 (KV cache)
- Module 04 (serving, scheduling)

## Topics

### Parallelism Strategies
- Tensor Parallelism (TP): splitting individual operations across GPUs
- Pipeline Parallelism (PP): splitting model layers across GPUs
- Data Parallelism (DP): replicating the model, splitting data
- Context Parallelism / Sequence Parallelism (CP/SP): splitting along sequence dimension
- Expert Parallelism (EP): distributing MoE experts across GPUs

### Communication Primitives
- All-reduce: aggregating gradients/activations across replicas
- All-gather: collecting distributed tensors
- Reduce-scatter: combined reduction and distribution
- All-to-all: redistribution for MoE expert routing

### Interconnects
- PCIe: bandwidth, latency, topology constraints
- NVLink: intra-node GPU-to-GPU high bandwidth
- NVSwitch: full bisection bandwidth within a node
- RDMA / InfiniBand: inter-node high-performance networking
- How interconnect topology constrains parallelism choices

### Advanced Patterns
- MoE placement: expert distribution strategies for load balance and communication
- P/D disaggregation: separate GPU pools for prefill and decode (connection to DistServe)
- Hybrid parallelism: combining TP + PP + DP

## Expected Artifacts

1. **Parallelism analysis** — determine the optimal strategy for a specific model on specific hardware
2. **Communication cost model** — estimate communication overhead for different parallelism configurations
3. **Topology-aware placement** — design expert/layer placement for a given interconnect topology
4. **Engineering report** — distributed serving architecture with cost-performance analysis

## Exit Criteria

The learner can:
- Select a parallelism strategy based on model size, hardware, and interconnect topology
- Estimate communication costs for different configurations
- Reason about MoE expert placement across devices
- Decide when P/D disaggregation is beneficial
- Produce a deployment plan for a multi-GPU/multi-node configuration

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

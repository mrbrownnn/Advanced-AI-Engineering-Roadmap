# Module 19 — Model-System Co-design

## Why This Module Exists

Model architecture decisions propagate through the entire system stack. This module connects the dots: how attention mechanism choices cascade through KV cache sizing, HBM consumption, concurrency limits, scheduler design, and ultimately economics. This is the integration module — it ties the full curriculum together.

## Key Engineering Questions

- How does the choice between MHA, GQA, and MLA propagate to system capacity and cost?
- How does the choice between dense and MoE propagate to serving architecture?
- Given a target workload, SLO, and budget, what model-system configuration is optimal?
- How do I evaluate architecture trade-offs across the full stack?

## Prerequisites

- Modules 01–05 (architecture, inference, KV cache, serving, optimization)
- Module 16 (distributed inference)
- Module 17 (economics)

## Topics

### Attention → System Chain

```
MHA → GQA → MLA
  → KV cache footprint per token
    → HBM consumption per request
      → Maximum concurrent requests
        → Scheduler behavior under load
          → Throughput and goodput
            → Cost per request
              → System economics
```

For each link: quantitative analysis of how the upstream choice constrains the downstream design.

### Dense → MoE → System Chain

```
Dense → MoE
  → Sparse compute (active vs total parameters)
    → Expert placement across devices
      → Communication patterns (all-to-all)
        → Interconnect topology requirements
          → Latency profile
            → Serving economics
```

For each link: quantitative analysis of how model sparsity changes system requirements.

### Integration Analysis
- End-to-end cost modeling: from model architecture to dollar per request
- Workload-architecture matching: which model architecture suits which workload pattern
- Scaling analysis: how costs change with growth in traffic, context length, or model size
- Migration analysis: the cost and risk of changing model architectures

## Expected Artifacts

1. **Attention chain analysis** — quantify the full MHA → GQA → MLA → system impact for a specific workload
2. **Dense vs MoE comparison** — end-to-end system analysis for both architectures on the same task
3. **Architecture selection decision** — recommend a model-system configuration for a given workload, SLO, and budget
4. **Engineering report** — full-stack architecture recommendation with quantitative justification

## Exit Criteria

The learner can:
- Trace the impact of a model architecture decision through the full system stack
- Quantify how attention variants affect capacity, throughput, and cost
- Compare dense and MoE architectures from a systems perspective
- Make architecture recommendations that account for model, system, and economic factors simultaneously
- Defend recommendations with end-to-end quantitative analysis

## Competency Targets

```yaml
competency:
  sfia: 5-6
  bloom: Create
  solo: Extended Abstract
  dreyfus: Competent → Proficient
```

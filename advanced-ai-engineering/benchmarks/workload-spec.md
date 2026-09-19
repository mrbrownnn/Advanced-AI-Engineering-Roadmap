# Workload Specification

A benchmark workload must be explicitly defined before any measurement.

## Required Fields

```yaml
workload:
  name: _
  description: _

  requests:
    arrival_pattern: _          # constant, Poisson, bursty, trace-replay
    rate: _                     # requests/second
    duration: _                 # total benchmark duration

  input:
    prompt_length_distribution: _   # fixed, uniform, normal, or specific distribution
    prompt_length_p50: _
    prompt_length_p95: _

  output:
    max_tokens: _
    output_length_distribution: _

  model:
    name: _
    size: _
    quantization: _

  concurrency:
    concurrent_requests: _
    ramp_up: _
```

## Guidelines

- **Realistic distributions** — Real workloads have skewed input/output length distributions. Use trace data when available.
- **Arrival patterns** — Constant rate is a baseline. Poisson is more realistic. Bursty tests resilience.
- **Duration** — Long enough for the system to reach steady state and for tail behavior to emerge.
- **Scaling dimension** — State which variable you are sweeping (concurrency, input length, batch size, etc.)

# Benchmark Result Schema

Every benchmark result must include these fields.

```yaml
result:
  id: _                         # unique identifier
  date: YYYY-MM-DD
  question: _                   # what question does this benchmark answer?

  environment:
    hardware:
      gpu: _
      gpu_count: _
      cpu: _
      memory_gb: _
      interconnect: _           # if multi-GPU
    software:
      os: _
      python: _
      torch: _
      cuda: _
      # ... other relevant packages
    commit: _                   # git commit of the code being benchmarked

  workload:
    # reference to workload-spec or inline definition
    model: _
    dataset: _
    configuration: _

  execution:
    warmup_iterations: _
    measurement_iterations: _
    total_duration_seconds: _

  raw_metrics:
    # all individual measurements
    latencies_ms: []
    throughputs: []
    # ... other raw data

  aggregated_metrics:
    latency_p50_ms: _
    latency_p95_ms: _
    latency_p99_ms: _
    latency_mean_ms: _
    latency_std_ms: _
    throughput_mean: _
    throughput_std: _
    ttft_p50_ms: _
    tpot_p50_ms: _
    # ... other aggregated metrics

  confidence:
    method: _                   # bootstrap, analytical, etc.
    confidence_level: _         # e.g., 0.95
    intervals: {}               # CI for key metrics

  known_limitations:
    - _

  notes: _
```

## Rules

1. **Append-only.** Never delete or overwrite results.
2. **Record raw data.** Aggregated metrics can be recomputed; raw data cannot.
3. **Document limitations.** No benchmark is perfect. State what this result does NOT tell you.
4. **Link to code.** Reference the exact commit and configuration used.

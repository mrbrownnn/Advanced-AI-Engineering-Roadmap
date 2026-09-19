# Benchmark Methodology

## Design Principles

1. **Define the question first.** Every benchmark answers a specific question. If you can't state the question, don't benchmark yet.

2. **Control variables.** Change one thing at a time. Document everything you hold constant.

3. **Warmup.** Discard initial iterations. GPU kernels, JIT compilation, and caching all affect early measurements.

4. **Sample size.** A single run is an anecdote, not a measurement. Run enough iterations to compute meaningful statistics.

5. **Report variance.** Mean without variance is meaningless. Report standard deviation, percentiles (p50, p95, p99), or confidence intervals.

6. **Pin everything.** Software versions, model weights, datasets, hardware, configuration, git commit.

7. **Append, don't overwrite.** Every result is a data point. Record all results, including unexpected ones.

## Before Running

- [ ] State the question this benchmark answers
- [ ] Define the workload (see [workload-spec.md](workload-spec.md))
- [ ] Pin the environment (hardware, software versions, commit)
- [ ] Define warmup strategy
- [ ] Define sample size and repetitions
- [ ] Identify potential confounders
- [ ] Predict the expected outcome (write it down before running)

## After Running

- [ ] Record raw results using [result-schema.md](result-schema.md)
- [ ] Compute aggregated statistics with confidence information
- [ ] Compare against prediction
- [ ] Document surprises
- [ ] Note known limitations
- [ ] Append to `results/`

## Common Pitfalls

| Pitfall | Mitigation |
|---------|-----------|
| Measuring cold start as steady state | Warmup iterations |
| Single-run conclusions | Multiple runs + statistics |
| Comparing across different hardware | Pin hardware, report specs |
| Ignoring tail latency | Report p95/p99, not just mean |
| Benchmarking the wrong thing | State the question first |
| Optimizing for the benchmark | Use realistic workloads |
| Discarding inconvenient results | Append-only policy |

# Module 13 — Harness Engineering

## Why This Module Exists

In production, models are never invoked in isolation. They operate inside an execution harness that formats prompts, enforces schemas, parses structured outputs, manages retries, handles fallbacks, and isolates failures. Harness engineering treats prompts and invocation protocols as software engineering artifacts with rigorous versioning, typing, and validation.

## Key Engineering Questions

- How do I structure output constraints (JSON schema, Pydantic, regex, grammar-based decoding) without degrading model reasoning quality?
- When does constrained decoding (e.g. grammar-guided sampling) cause latency spikes or degenerative repetition?
- How do I build deterministic fallback and recovery pipelines when a model violates the harness contract?
- How should temperature, top-p, seed, and stop tokens be systematically configured per task archetype?
- How do I version, test, and lint prompt templates as production code?

## Prerequisites

- Module 01 (Foundation Model Internals & token mechanics)
- Module 07 (Model Behavior, Uncertainty & Calibration)
- Module 12 (Agent Loop Engineering)

## Topics

### Structured Outputs and Constrained Generation
- JSON mode vs schema enforcement vs grammar-guided decoding (GBNF, Outlines, guidance)
- Performance impact of constrained decoding on TTFT and TPOT
- Handling schema truncation and invalid token states

### Harness Reliability Patterns
- Fallback cascades: retry with increased temperature, retry with correction prompt, fallback to stronger model
- Stop token discipline and runaway output prevention
- Deterministic extraction: few-shot exemplars, scratchpads, and delimiter engineering

### Prompt Engineering as Software Engineering
- Prompt template modularity, parameter typing, and linting
- CI/CD pipelines for prompt regression testing
- Environment parity: testing prompts against varying model quantizations and runtime backends

## Expected Artifacts

1. **Structured output harness** — an implementation comparing grammar-guided sampling vs schema validation + retry across 1,000 synthetic outputs
2. **Harness latency benchmark** — measurement of token generation overhead introduced by grammar constraints
3. **Engineering report** — contract design, error budgets, and fallback strategies for structured extraction

## Exit Criteria

The learner can:
- Design a type-safe, production-grade model harness with explicit error boundaries
- Quantify the latency and throughput trade-off between client-side retries and engine-level grammar constraints
- Implement automated regression testing for prompt modifications

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Apply → Analyze
  solo: Relational
  dreyfus: Competent
```

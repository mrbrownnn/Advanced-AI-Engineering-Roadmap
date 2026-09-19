# Advanced AI Engineering

A competency-driven, 26-week advanced AI engineering program.

## What This Is

A structured curriculum for engineers at Junior+ / SFIA Level 4 targeting strong Middle / SFIA Level 5 technical capability, with selected SFIA Level 6 stretch exercises.

The program covers the complete chain from scientific experimentation through production AI systems:

```
Scientific Experimentation → LLM Internals → GPU / Inference Fundamentals
→ KV Cache Engineering → Serving / Scheduling / Capacity → Inference Optimization
→ Model Behaviour / Uncertainty → AI Data Engineering → Retrieval → RAG
→ Context / Memory → Durable Agent Runtime → Evaluation → Falsification
→ AI Security → Model Adaptation → Distributed Inference → AI Economics
→ Observability / Reliability → Model-System Co-design → Capstone
```

### Central Principle

```
BUILD → MEASURE → BREAK → EXPLAIN → IMPROVE → DEFEND
```

A system merely "working" is **not** considered completion.

## What This Is NOT

- ❌ A beginner AI course
- ❌ A framework tutorial collection
- ❌ A paper-reading list
- ❌ An LLM application tutorial
- ❌ An interview-cramming repository

## Learning Loop

Every module follows this default loop:

```
Concept → Paper / Official Documentation → Source Reading → Small Implementation
→ Instrument → Benchmark → Break → Diagnose → Optimize → Benchmark Again
→ Engineering Decision → Engineering Report
```

The learner should repeatedly answer:

- What did I assume?
- What did I predict?
- What did I measure?
- How reliable is the measurement?
- What broke?
- Why did it break?
- What evidence supports the explanation?
- What alternatives exist?
- What trade-off did I choose?
- Under what workload would the decision change?
- What is the rollback condition?

## Repository Structure

```
advanced-ai-engineering/
├── 00-foundations/          Scientific engineering baseline
├── 01-llm-internals/       Decoder architecture deep dive
├── 02-inference-fundamentals/  GPU execution and profiling
├── 03-kv-cache-engineering/    Memory management for KV
├── 04-serving-scheduling-capacity/  Batching and queueing
├── 05-inference-optimization/  Bottleneck-driven optimization
├── 06-model-behavior-uncertainty/  Failure modes and calibration
├── 07-ai-data-engineering/     Data quality and lifecycle
├── 08-retrieval-engineering/   Search and ranking systems
├── 09-advanced-rag/            Retrieval lifecycle engineering
├── 10-context-memory-engineering/  Context selection and memory
├── 11-durable-agent-runtime/   Stateful agent execution
├── 12-evaluation-engineering/  Measurement and regression
├── 13-falsification-engineering/  Adversarial testing
├── 14-ai-security/             Threat modeling and defense
├── 15-model-adaptation/        Fine-tuning and data flywheel
├── 16-distributed-inference/   Parallelism and communication
├── 17-ai-economics/            Cost optimization and routing
├── 18-observability-reliability/  Tracing and incident response
├── 19-model-system-codesign/   Architecture-model coupling
├── capstone/                   AI Runtime Platform project
├── graduation/                 Final architecture review
├── papers/                     Curated paper index
├── source-reading/             Source code study methodology
├── benchmarks/                 Benchmark methodology and results
├── incidents/                  Diagnostic scenario templates
├── templates/                  Checkpoint, report, experiment templates
└── references/                 Supplementary reference material
```

## Getting Started

1. Read [ROADMAP.md](ROADMAP.md) for the 26-week schedule
2. Read [CURRICULUM.md](CURRICULUM.md) for the module index
3. Read [COMPETENCY.md](COMPETENCY.md) for the competency framework
4. Start with [00-foundations/](00-foundations/) before any other module

## Conventions

- All competency metadata uses the format defined in [COMPETENCY.md](COMPETENCY.md)
- Engineering reports follow [templates/engineering-report.md](templates/engineering-report.md)
- Checkpoints follow [templates/checkpoint.md](templates/checkpoint.md)
- Benchmark results are append-only; never overwrite inconvenient results
- Source reading exercises pin a specific commit/tag

## Status

🏗️ **Repository skeleton established.** Content is being progressively developed over 26 weeks.

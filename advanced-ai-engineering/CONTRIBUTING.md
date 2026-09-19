# Contributing

## Content Standards

### Checkpoints
- Use ambiguous engineering situations, not definition quizzes
- Include competency metadata (SFIA, Bloom, SOLO, Dreyfus)
- Follow [templates/checkpoint.md](templates/checkpoint.md)

### Engineering Reports
- Follow [templates/engineering-report.md](templates/engineering-report.md)
- Must include measurements, failure analysis, and rollback criteria

### Paper References
- Use only real, verifiable references
- Add `TODO: VERIFY REFERENCE` if uncertain about any bibliographic data
- Never invent titles, authors, URLs, or publication years

### Source Reading
- Always pin a specific commit or tag
- Record the execution path through the codebase
- Follow methodology in [source-reading/README.md](source-reading/README.md)

### Benchmarks
- Results are **append-only** — never overwrite inconvenient results
- Record full environment, hardware, and software versions
- Follow [benchmarks/methodology.md](benchmarks/methodology.md)

### Objectives
- Use observable outcomes only
- ❌ "Understand PagedAttention"
- ✅ "Given an unfamiliar serving workload, estimate KV memory pressure, identify likely fragmentation behaviour, select an allocation strategy, and validate the prediction experimentally"

## File Conventions

- Do not create empty files for symmetry
- Every file should contain meaningful initial content
- Module structure follows the pattern in module READMEs
- Markdown files use ATX-style headers (`#`)

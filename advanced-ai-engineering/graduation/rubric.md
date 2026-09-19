# Graduation Rubric

## Assessment Dimensions

### 1. Problem Analysis (SFIA 5 / Bloom: Analyze)

| Level | Description |
|-------|-------------|
| Does not meet | Accepts requirements at face value, misses ambiguity |
| Meets | Identifies ambiguities, documents assumptions, asks clarifying questions |
| Exceeds | Identifies second-order implications and unstated constraints |

### 2. Architecture Design (SFIA 5 / Bloom: Create)

| Level | Description |
|-------|-------------|
| Does not meet | Copies an existing design without adaptation |
| Meets | Designs a system tailored to the specific constraints and workload |
| Exceeds | Considers multiple architectures, quantitatively compares, and selects with evidence |

### 3. Quantitative Reasoning (SFIA 5 / Bloom: Evaluate)

| Level | Description |
|-------|-------------|
| Does not meet | No quantitative analysis; hand-waving arguments |
| Meets | Produces capacity model, cost model, and performance predictions with evidence |
| Exceeds | Includes sensitivity analysis — how conclusions change under different assumptions |

### 4. Failure Analysis (SFIA 5 / Bloom: Evaluate)

| Level | Description |
|-------|-------------|
| Does not meet | Assumes the system works; no failure analysis |
| Meets | Identifies failure modes, designs mitigations, defines rollback criteria |
| Exceeds | Designs falsification experiments targeting the weakest assumptions |

### 5. Security (SFIA 5 / Bloom: Evaluate)

| Level | Description |
|-------|-------------|
| Does not meet | No security analysis |
| Meets | Produces threat model, identifies trust boundaries, designs defenses |
| Exceeds | Tests defenses, identifies residual risks, documents accepted risk |

### 6. Economics (SFIA 5 / Bloom: Evaluate)

| Level | Description |
|-------|-------------|
| Does not meet | No cost analysis |
| Meets | Produces cost model, identifies optimization opportunities |
| Exceeds | Maps the quality-latency-cost Pareto frontier, recommends operating point with justification |

### 7. Communication (SFIA 5 / Bloom: Create)

| Level | Description |
|-------|-------------|
| Does not meet | Disorganized, cannot explain decisions |
| Meets | Clear engineering report with evidence, trade-offs, and recommendations |
| Exceeds | Anticipates counterarguments, addresses them proactively |

## Competency Targets

```yaml
competency:
  sfia: 5 (stretch: 6)
  bloom: Evaluate → Create
  solo: Relational → Extended Abstract
  dreyfus: Competent → Proficient
```

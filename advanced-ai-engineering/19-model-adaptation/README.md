# Module 15 — Model Adaptation

## Why This Module Exists

Fine-tuning is one option in a space of adaptation strategies. Before training anything, the engineer must ask: would prompting, retrieval, context engineering, tool use, or model routing solve this problem more cheaply? This module teaches both the techniques of adaptation and the decision framework for when to use them.

## Key Engineering Questions

- Should I fine-tune, or can I solve this with prompting / retrieval / tools / routing?
- What data do I need, how much, and how do I curate it?
- What is the total cost of adaptation (data, compute, evaluation, maintenance)?
- How do I avoid contamination and evaluate fairly?
- How do I build a data flywheel from production failures?

## Prerequisites

- Module 06 (failure taxonomy)
- Module 07 (data engineering)
- Module 12 (evaluation)

## Topics

### Training Techniques
- SFT (Supervised Fine-Tuning): task-specific training on instruction-output pairs
- LoRA: low-rank adaptation without full model training
- QLoRA: quantized LoRA for reduced memory
- PEFT landscape: adapters, prefix tuning, prompt tuning
- DPO: preference optimization without explicit reward models
- RLHF concepts: reward modeling and policy optimization

### Data for Adaptation
- Failure mining: extracting training data from production failures
- Synthetic generation: creating training data with LLMs
- Rejection sampling: generating and filtering for quality
- Filtering and deduplication
- Difficulty curriculum: ordering training data by complexity
- Data mixture: balancing task types and domains
- Hard negatives: mining challenging examples for contrastive tasks
- Contamination: preventing evaluation data from leaking into training

### Data Flywheel
- Production failure → data collection → curation → training → evaluation → deployment
- Active learning loop: prioritizing which failures to fix
- Continuous improvement cycle

### Decision Framework
Every adaptation decision should be compared against:
- Prompting: can better instructions solve this?
- Retrieval: can providing the right context solve this?
- Context engineering: can restructuring the context solve this?
- Tool use: can giving the model the right tools solve this?
- Model routing: can using a different model for this task type solve this?

## Expected Artifacts

1. **Adaptation decision analysis** — for a specific failure pattern, compare fine-tuning against alternatives
2. **Data curation pipeline** — build a training dataset from failure examples with quality controls
3. **LoRA training experiment** — fine-tune with LoRA, evaluate, and compare against prompt-based solutions
4. **Engineering report** — adaptation strategy with cost-benefit analysis

## Exit Criteria

The learner can:
- Make an evidence-based decision about whether to fine-tune or use an alternative strategy
- Curate training data with proper quality controls and contamination prevention
- Execute a LoRA/QLoRA training run and evaluate the result rigorously
- Design a data flywheel from production failures to model improvement
- Quantify the total cost of adaptation including data, compute, and maintenance

## Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate → Create
  solo: Relational → Extended Abstract
  dreyfus: Competent
```

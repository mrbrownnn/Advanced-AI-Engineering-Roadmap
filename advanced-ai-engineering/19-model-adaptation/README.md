# Module 19 — Model Adaptation

## 00 Why This Module Exists

Fine-tuning changes the model's conditional distribution for every future input. Prompting, retrieval, and routing change only what a single request sees or which model answers it. That asymmetry decides most adaptation questions: a persistent format, procedure, or skill gap is a candidate for weight updates; a missing or changing fact is usually a retrieval problem; and any weight update can move behavior the target metric never looks at—general capability and safety included.

```text
failure analysis --> failure class --> alternative? (prompt / RAG / route) --no--> adapt
                                                                                 |
     data snapshot (lineage, dedup, contamination, template, loss mask) <--------+
                   |
                   v
     method (full / LoRA / QLoRA; SFT / DPO / GRPO) + memory plan + run manifest
                   |
                   v
     gate: target gain  AND  retained capability  AND  safety   (paired vs base, repeated seeds)
                   |
                   v
     Module 17 rollout: canary --> promote / rollback to previous adapter
```

Module 01 introduced parameter accounting; Module 05 quantization contracts; Module 08 data lineage and contamination; Module 15 paired evaluation and release gates; Module 17 harness rollout; Module 18 the security threat model. This module owns the adaptation decision, training memory, parameter-efficient mechanics, adaptation data, preference optimization, forgetting, and adaptation-specific safety gates—including harmful or poisoned training data in adaptation pipelines. Test-time reasoning stays in Module 06, distributed execution in Module 20, and full cost modeling in Module 22.

**Research cutoff:** 2026-09-30.

**Module Orientation**
- **Engineering Problem**: Decide whether a failure should be fixed by changing weights, and if so, change them with a known memory budget, leak-free data, and evidence that nothing important regressed.
- **What You Will Do**: Write an adaptation decision record, compute training memory and LoRA parameter counts, implement and trace LoRA at a pinned PEFT commit, build SFT and preference data with correct templates and loss masks, run DPO/GRPO-style updates on a small model, and gate an adapter on target, retained capability, and safety.
- **Environment**: Python 3.10+, PyTorch, Transformers, PEFT (and optionally TRL), one GPU able to fine-tune a ≤3B open-weight model with LoRA, the Module 15 evaluation harness, and a fixed safety prompt suite. No production data without the Module 08 lineage controls.
- **Evidence Rule**: Keep source observations (**O**), explicit derivations (**D**), and measurement-dependent hypotheses (**H**) separate. All numbers in worked examples are declared hypothetical or derived; author-reported results are labeled as such.

## 01 Baseline Assumptions

- Module 01: parameter accounting and decoder tensor shapes.
- Module 02: mixed precision and GPU memory.
- Module 05: quantization formats and their accuracy contracts.
- Module 08: data lineage, deduplication, and contamination.
- Module 15: paired evaluation, slices, and release gates.
- Module 17: canary rollout and rollback of harness and model versions.
- Module 18: threat modeling and the harmful fine-tuning risk class.

## 02 Target Mastery

```yaml
depth_contract:
  conceptual: REQUIRED
  mechanistic: REQUIRED
  mathematical: REQUIRED
  quantitative: REQUIRED
  implementation: REQUIRED
  source_code: REQUIRED
  instrumentation: REQUIRED
  experimental: REQUIRED
  statistical: REQUIRED
  production_reasoning: REQUIRED
  failure_analysis: REQUIRED
  falsification: REQUIRED
  security: SELECTIVE
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: REQUIRED

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 13h
  assessment: 3h
  source_trace: 2h
  total: 26h
```

The learner must justify adaptation against cheaper alternatives, predict memory before training, implement and trace LoRA, construct data whose measured gains are not leakage, explain the objectives of RLHF, DPO, and GRPO and their failure modes, and ship an adapter only with paired evidence on target gain, retained capability, and safety.

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

Failure Analysis $\to$ Failure Class $\to$ Adaptation Decision (prompt / retrieve / route / SFT / preference). If adapting: Training Memory (model states + activations) $\to$ Method (full / LoRA / QLoRA) $\to$ Data (lineage, dedup, contamination, template, loss mask) $\to$ Objective (SFT; RLHF; DPO; GRPO) $\to$ Gate (target, retained capability, safety; paired, repeated) $\to$ Rollout (Module 17). Cross-cutting risks: forgetting, reward/length pathologies, train/serve mismatch, safety regression, poisoned training data.

## 04 Lessons

### Lesson 19.1 — The Adaptation Decision: Adapt, Retrieve, Prompt, or Route

**Engineering Question:**
For this failure class, is changing weights the cheapest intervention that fixes it durably?

**Concepts & Definitions:**
- **Failure class**: a cluster of errors with a shared cause found by error analysis (Module 15)—missing knowledge, stale knowledge, format/procedure, capability, or policy.
- **Weight adaptation** changes the conditional distribution for all future inputs; **prompting, retrieval, and routing** change the input or the model per request (**D**, CLM-001).
- **Knowledge injection evidence**: Ovadia et al. report that RAG consistently outperformed unsupervised fine-tuning for both previously seen and new knowledge on their tasks, and that models struggled to learn new facts via fine-tuning unless shown many variations of each fact (**O**, CLM-014; unsupervised fine-tuning, evaluated tasks only).

**Mechanism Explanation:**
Fine-tuned knowledge is refreshed only by retraining and cannot cite its source per answer; a retrieval index is refreshed by an index write and returns citable evidence. Format and procedure, by contrast, are persistent behaviors: a prompt re-teaches them on every request at a token cost, while an adapter learns them once. Hybrid designs—an adapter for behavior plus retrieval for facts—are common; the decision is made per failure class, not per product.

**Quantitative Model / Trade-off Comparison:**
Per failure class, compare over a planning horizon with $N$ requests and $U$ refreshes:
$C_{prompt} = N\,\Delta t_{prompt}\,c_{tok}$; $C_{RAG} = N\,(\Delta t_{ctx}\,c_{tok} + c_{retr}) + U\,c_{index}$; $C_{adapt} = U\,(c_{train} + c_{eval}) + N\,\Delta c_{serve}$.
Cost is necessary but not sufficient: each option must also meet the quality target. Hypothesis to test (**H**, CLM-013): RAG wins post-training-fact accuracy at lower update cost; fine-tuning matches or beats prompting on format compliance at lower per-request token cost.

**Worked Example (declared hypothetical inputs):**
A support bot fails in two ways: 35% of errors cite last quarter's return policy (stale facts); 40% emit invalid ticket JSON (format). A 1,200-token instruction block with examples fixes most JSON errors but costs 1,200 tokens × 2M requests/month = 2.4B input tokens/month. Decision record: route stale-policy failures to retrieval with dated policy documents; test an SFT adapter for JSON format against the prompt baseline at equal validity, with the prompt kept as the fallback. Fine-tuning policy text into weights is rejected: the next policy change would require retraining.

**Knowledge Check:**
1. Why can a fine-tuned model not provide provenance for a fact it learned?
2. Which evidence would falsify "SFT is cheaper than prompting for this format"?

**Guided Practice:**
Take 50 labeled failures from a Module 15 error analysis. Cluster them into failure classes, and write one decision record per class with the alternative tried, the predicted winner, and the falsifying observation.

**Feedback Contract:**
- *Expected Evidence*: Failure classes with counts, the alternative per class, cost terms, quality target, and falsifier.
- *Common Failure*: Choosing fine-tuning because the product "needs domain knowledge."
- *Diagnostic Hint*: Will this fact change before the next training run?
- *Concept to Revisit*: Error Analysis and Slices (Module 15); RAG Freshness (Module 10).

**Learning Outcome:**
Justify or reject weight adaptation per failure class with a testable hypothesis.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 19.2 — Training Memory: Model States, Activations, and Quantized Bases

**Engineering Question:**
Will this run fit, and which term dominates?

**Concepts & Definitions:**
- **Model states**: parameters, gradients, and optimizer states. ZeRO states that mixed-precision Adam needs $2\Psi$ bytes for fp16 parameters, $2\Psi$ for fp16 gradients, and $K\Psi=12\Psi$ for the fp32 parameter copy, momentum, and variance—$16\Psi$ bytes before activations (**O**, CLM-002).
- **Activations**: saved forward tensors, scaling with batch, sequence length, depth, and width; reduced by activation checkpointing at recompute cost.
- **Parameter-efficient fine-tuning**: LoRA trains only low-rank adapters on a frozen base; its authors report up to 10,000× fewer trainable parameters and 3× lower GPU memory than Adam fine-tuning of GPT-3 175B (**O**, CLM-003).
- **QLoRA**: backpropagates through a frozen 4-bit NF4 base into LoRA adapters with double quantization and paged optimizers; its authors report fine-tuning a 65B model on one 48 GB GPU while preserving 16-bit fine-tuning task performance on evaluated tasks (**O**, CLM-004).

**Mechanism Explanation:**
Full fine-tuning pays $16\Psi$ for every parameter because every parameter has gradients and Adam moments. LoRA keeps the base frozen—it needs only resident weights—and pays the full per-parameter state cost only for adapter parameters. QLoRA further shrinks the frozen base term by storing it in 4 bits and dequantizing on the fly. None of these methods removes activation memory, which often becomes the dominant term once model states shrink.

**Quantitative Model / Derivation (D):**
- Full mixed-precision Adam: $M_{states} = 16\Psi$ bytes.
- LoRA with a 16-bit frozen base: $M \approx 2\Psi + 16\,\Psi_{LoRA} + M_{act}$, where 16 bytes/adapter parameter assumes fp32 adapter weights, gradients, and two Adam moments.
- QLoRA: $M \approx 0.5\Psi + M_{quant\ consts} + 16\,\Psi_{LoRA} + M_{act}$, plus any layers kept in higher precision.
Memory is a prediction to be checked against `torch.cuda.max_memory_allocated()` and allocator reserved memory (Module 02), not a guarantee.

**Worked Example (derived; hypothetical 8B decoder):**
$\Psi = 8\times10^9$. Full fine-tuning model states: $16 \times 8\times10^9 = 128$ GB before activations—more than one 80 GB GPU. LoRA ($\Psi_{LoRA}\approx 41.9$M, computed in Lesson 19.3): $2\times8\times10^9 = 16$ GB frozen base + $16\times41.9\text{M}\approx0.67$ GB adapter states ≈ 16.7 GB + activations. QLoRA: ≈ 4 GB base + constants + 0.67 GB + activations. At sequence length 4,096 and micro-batch 8, activations can exceed all of these; measure with and without checkpointing.

**Knowledge Check:**
1. Why does LoRA not reduce activation memory proportionally to trainable parameters?
2. What changes in $K$ if the optimizer is SGD with momentum instead of Adam?

**Guided Practice:**
For a 1B model, predict peak memory for full, LoRA, and QLoRA at two sequence lengths; then measure peak allocated memory for one training step of each and explain every gap above 10%.

**Feedback Contract:**
- *Expected Evidence*: Per-term predictions, measured peaks, checkpointing setting, and explanation of residuals.
- *Common Failure*: Counting only the weights and discovering optimizer states at OOM.
- *Diagnostic Hint*: Which tensors have gradients, and which have Adam moments?
- *Concept to Revisit*: Memory Hierarchy and Mixed Precision (Module 02); Quantization Contracts (Module 05).

**Learning Outcome:**
Predict and verify training memory for full, LoRA, and QLoRA runs.

*(Effort: 45m instruction, 15m practice)*

---

### Lesson 19.3 — LoRA Mechanics: Rank, Scaling, Targets, and Merging

**Engineering Question:**
What exactly does LoRA add to a layer, and what do rank, alpha, and target modules control?

**Concepts & Definitions:**
- **Low-rank update**: for frozen $W\in\mathbb{R}^{d_{out}\times d_{in}}$, learn $B\in\mathbb{R}^{d_{out}\times r}$ and $A\in\mathbb{R}^{r\times d_{in}}$: $y = Wx + s\,B(Ax)$ with $s=\alpha/r$ ($\alpha/\sqrt r$ with rsLoRA).
- **Zero initialization**: $B=0$ at start, so the adapted model initially equals the base.
- **Merge**: $W' = W + sBA$ removes adapter compute at serving time; unmerged adapters allow many adapters over one base.
- **Capacity**: Biderman et al. report standard low-rank LoRA substantially underperformed full fine-tuning on code and math target domains yet better preserved out-of-domain performance, and that full fine-tuning learned perturbations of 10–100× higher rank (**O**, CLM-006). LoRA Without Regret reports LoRA matched full fine-tuning on small-to-medium SFT datasets and in RL, that attention-only LoRA significantly underperformed MLP-only LoRA, and that the best LoRA learning rate was about 10× the full fine-tuning rate (**O**, CLM-015; lab post, not peer reviewed).

**Mechanism Explanation — Pinned PEFT Trace (O, CLM-005):**
At `huggingface/peft` commit `b8674c86183a5dee38d0c3ede392e189593025e5` (static inspection, 2026-09-30), `src/peft/tuners/lora/layer.py`:
1. `LoraLayer.update_layer` creates `lora_A = nn.Linear(in_features, r)` and `lora_B = nn.Linear(r, out_features)` and sets `scaling = lora_alpha / r`, or `lora_alpha / math.sqrt(r)` when `use_rslora`.
2. `LoraLayer.reset_lora_parameters` initializes `lora_A` with Kaiming-uniform (default) or Gaussian and zeroes `lora_B`. Other `init_lora_weights` values (PiSSA, OLoRA, LoftQ, and others) take different paths.
3. `Linear.forward` computes `base_layer(x)` and, for each active unmerged vanilla adapter, adds `lora_B(lora_A(dropout(x))) * scaling`; variants such as DoRA dispatch to their own forward.
4. `Linear.get_delta_weight` returns `transpose(B @ A, fan_in_fan_out) * scaling`; `Linear.merge` adds it into the base weight (with an optional NaN-checked `safe_merge`), and `unmerge` subtracts it.
Lesson: rank and alpha are coupled through `scaling`; changing `r` without revisiting `alpha` or the learning rate changes the effective update size.

**Quantitative Model / Derivation (D):**
Trainable parameters per adapted matrix: $r(d_{in}+d_{out})$. Extra forward FLOPs per token per unmerged matrix: $\approx 2r(d_{in}+d_{out})$ versus $2d_{in}d_{out}$ for the base.

**Worked Example (derived; hypothetical decoder with 32 layers, $d=4096$, GQA $k/v$ width 1024, MLP width 14336, $r=16$ on all linear projections):**
Per layer: $q,o$: $16(4096+4096)=131{,}072$ each; $k,v$: $16(4096+1024)=81{,}920$ each; gate, up, down: $16(4096+14336)=294{,}912$ each. Sum $=1{,}310{,}720$; × 32 layers ≈ **41.9M** trainable parameters. Base linear parameters per layer ≈ 218.1M, so unmerged adapter FLOPs add ≈ 0.6%; any larger measured slowdown comes from extra kernels and memory traffic (**H**, measure it). Scaling: $r=16,\alpha=32\Rightarrow s=2$; moving to $r=64$ with $\alpha=32$ gives $s=0.5$ (rsLoRA: $32/\sqrt{64}=4$).

**Knowledge Check:**
1. Why does zero-initializing $B$ make the first training step start exactly from the base model?
2. Why can the Biderman and LoRA Without Regret results both be true?

**Guided Practice:**
Implement a from-scratch LoRA wrapper for `nn.Linear`, check numerical equivalence with PEFT on the same seeds, then verify that merged and unmerged outputs agree within a stated tolerance in fp32 and bf16.

**Feedback Contract:**
- *Expected Evidence*: Parameter counts, equivalence test with tolerance, merge parity per dtype, and trace citations by symbol.
- *Common Failure*: Doubling rank and reporting "no improvement" while the effective update scale halved.
- *Diagnostic Hint*: What is `scaling` before and after your change?
- *Concept to Revisit*: Parameter Accounting (Module 01).

**Learning Outcome:**
Implement, count, trace, and merge LoRA, and explain its capacity trade-off.

*(Effort: 45m instruction, 15m practice; source trace 2h)*

---

### Lesson 19.4 — Adaptation Data: Failure Mining, Templates, Loss Masks, and Poisoning

**Engineering Question:**
Will the measured gain come from generalization, or from leakage, formatting accidents, or poisoned examples?

**Concepts & Definitions:**
- **Failure mining**: turning production failures into demonstrations or preference pairs, with labels from experts or verified corrections.
- **Train/evaluation separation**: adaptation data mined from logs must be separated from evaluation data by source and time and near-duplicate filtered (**D**, CLM-007; Module 08).
- **Chat template fidelity**: the role markers, special tokens, and system prompt used in training must match serving.
- **Loss masking**: computing loss only on target (assistant) tokens.
- **Harmful and poisoned training data**: Qi et al. report that fine-tuning GPT-3.5 Turbo on 10 adversarial examples for under $0.20 made it responsive to nearly any harmful instruction (**O**, CLM-011). Every example in an adaptation set is therefore an input with authority over future behavior.

**Mechanism Explanation:**
If near-duplicates of evaluation items enter training, the gain measures memorization. If the template differs, the model learns a conversation format it will never see at serving time. If prompt tokens carry loss, the model spends capacity on reproducing prompts. If data sources are writable by untrusted parties, the training set becomes an attack surface—Module 18's source–sink analysis applies, with the adapter as the sink.

**Quantitative Model / Derivation (D):**
Fraction of loss on target tokens without masking: $T_{resp}/(T_{prompt}+T_{resp})$. Contamination check: fraction of evaluation items with an $n$-gram or embedding near-duplicate in training above a fixed threshold, reported with the threshold.

**Worked Example (declared hypothetical):**
Examples average 900 prompt tokens and 100 response tokens. Without masking, only 10% of the loss is on responses. After masking, validation loss on responses drops faster, and JSON validity improves on a held-out set that shares no ticket IDs or dates with training. A near-duplicate scan finds 6% of the evaluation set in the mined data; those items are removed from training, and the pre-fix gain is reported as leaked.

**Knowledge Check:**
1. Why can a template mismatch hurt only at serving time?
2. Which Module 18 controls apply to a data flywheel that ingests user feedback?

**Guided Practice:**
Build an SFT set from 500 mined failures: lineage IDs, dedup and contamination report, template rendering through the serving tokenizer, loss-mask verification by printing masked tokens, and a provenance filter for untrusted sources.

**Feedback Contract:**
- *Expected Evidence*: Data manifest, contamination rate with threshold, rendered-template diff against serving, mask visualization, and source trust labels.
- *Common Failure*: Evaluating on a random split of the same mined logs.
- *Diagnostic Hint*: Could any evaluation item's answer be copied from a training item?
- *Concept to Revisit*: Contamination and Lineage (Module 08); Source–Sink Models (Module 18).

**Learning Outcome:**
Construct adaptation data whose measured gains survive leakage, template, and provenance checks.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 19.5 — Preference Optimization: RLHF, DPO, and GRPO

**Engineering Question:**
How do preference and reward objectives move the policy, and what keeps them from moving it too far or in the wrong direction?

**Concepts & Definitions:**
- **RLHF**: InstructGPT used supervised demonstrations, a reward model trained on human rankings, and RL; labelers preferred its 1.3B model's outputs to 175B GPT-3's (**O**, CLM-008).
- **DPO**: a classification loss on preference pairs that implicitly defines a reward relative to a reference policy, with $\beta$ controlling deviation (**O**, CLM-009).
- **GRPO**: drops PPO's value model; estimates a baseline from $G$ sampled outputs per prompt, normalizes rewards by group mean and standard deviation, and adds a KL estimator to the loss (**O**, CLM-010).
- **GRPO biases**: Liu et al. show response-length normalization favors brevity among correct and length among incorrect answers, and standard-deviation normalization up-weights low-variance questions; Dr. GRPO removes both (**O**, CLM-017). DAPO's Dynamic Sampling filters prompts whose samples are all correct or all incorrect (**O**, CLM-018).
- **Forgetting and RL**: RL's Razor reports on-policy RL forgot less than SFT at similar new-task performance, with forgetting tracking KL to the base on the new task (**O**, CLM-016; FRONTIER preprint).

**Mechanism Explanation:**
All three regularize toward a reference: RLHF and GRPO with a KL term, DPO through the reference log-probabilities inside its loss. The reward signal is the weakest link: a learned reward model can be exploited, offline preference pairs cannot explore, and programmatic rewards reward only what they check. Normalization details are not cosmetic—they decide which samples and questions dominate the update.

**Quantitative Model / Derivation:**
- DPO: $\mathcal{L} = -\log\sigma\big(\beta[(\log\pi_\theta(y_w|x)-\log\pi_{ref}(y_w|x)) - (\log\pi_\theta(y_l|x)-\log\pi_{ref}(y_l|x))]\big)$.
- GRPO outcome advantage: $A_i = (r_i-\bar r)/\mathrm{std}(r)$ over the group; if all $r_i$ are equal, $A_i=0$ and the prompt contributes no policy gradient (**D**).

**Worked Example (derived):**
DPO with $\beta=0.1$: at initialization $\pi_\theta=\pi_{ref}$, the margin is 0 and $\mathcal{L}=\log 2\approx0.693$. After training, chosen log-ratio $+2.0$ and rejected $-1.0$ give margin $0.1\times3=0.3$ and $\mathcal{L}=\log(1+e^{-0.3})\approx0.554$. GRPO with $G=4$, rewards $[1,0,0,1]$: mean 0.5, population std 0.5, advantages $[+1,-1,-1,+1]$. Rewards $[1,1,1,1]$: zero advantage—wasted samples unless filtered. Standard-deviation conventions (population versus sample, epsilon) vary by implementation; read the one you run.

**Knowledge Check:**
1. What does DPO's reference model do that plain likelihood on chosen responses does not?
2. Why can a GRPO run's average response length grow while accuracy stalls?

**Guided Practice:**
On a small model, run DPO on 2,000 preference pairs at three $\beta$ values, and a GRPO-style run on a verifiable math or format task. Log chosen/rejected log-ratios, KL to reference, response length by correctness, and the fraction of zero-advantage groups.

**Feedback Contract:**
- *Expected Evidence*: Loss and margin curves, KL-to-reference, length split by correctness, zero-advantage fraction, and held-out reward validity checks.
- *Common Failure*: Reporting rising reward as improvement without checking the reward against independent labels.
- *Diagnostic Hint*: What does the reward not check?
- *Concept to Revisit*: Verifier Gaming (Module 06); Judge Validity (Module 15).

**Learning Outcome:**
Apply and diagnose DPO- and GRPO-style preference optimization.

*(Effort: 45m instruction, 15m practice)*

---

### Lesson 19.6 — Forgetting, Safety Regression, and the Release Gate

**Engineering Question:**
What evidence justifies shipping an adapted model instead of the base?

**Concepts & Definitions:**
- **Catastrophic forgetting**: loss of capability outside the target domain after adaptation (CLM-006, CLM-016).
- **Safety regression**: benign fine-tuning can degrade safety alignment, to a lesser extent than adversarial fine-tuning (**O**, CLM-011). Fraser et al. report surprising variance in safety-benchmark results under trivial fine-tuning setup changes (**O**, CLM-019). Huang et al. (2026) model alignment reversal under later fine-tuning (**O**, CLM-020; FRONTIER).
- **Three-part gate**: target gain, retained general capability, and safety/refusal behavior, each paired against the base on suites frozen before training (**D**, CLM-012).

**Mechanism Explanation:**
A single target metric is blind to the two regressions most likely to follow adaptation. The gate therefore has three pre-registered components with non-inferiority margins for retained capability and safety, repeated seeds or runs to expose variance, and an over-refusal check so that "safer" does not mean "useless." Passing the gate authorizes a canary (Module 17), not full rollout.

**Quantitative Model / Derivation:**
For paired items, estimate $\Delta = \mathrm{score}_{adapted} - \mathrm{score}_{base}$ with a paired confidence interval (Module 15). Ship only if target $\Delta$'s lower bound exceeds the minimum useful gain and the general and safety $\Delta$ lower bounds exceed $-m_{gen}$ and $-m_{safe}$, with margins fixed before training. Report run-to-run spread across at least three training seeds.

**Worked Example (declared hypothetical):**
Target JSON validity: +8.0 points [6.1, 9.9]. General suite: −1.2 [−2.0, −0.4] against $m_{gen}=1.5$: the lower bound −2.0 violates the margin, so non-inferiority is not shown. Harmful-request refusal: 97% base; adapted 91–96% across three seeds. Decision: do not ship; retry with lower rank or learning rate, mix in general and safety replay data, and re-gate. A single-seed 96% would have passed and hidden the spread.

**Knowledge Check:**
1. Why must safety suites be run on every adapter refresh, even with benign data?
2. Why is one seed insufficient for a safety gate?

**Guided Practice:**
Define the gate for the support-bot adapter: suites, sizes, margins, seeds, over-refusal checks, and the rollback target.

**Feedback Contract:**
- *Expected Evidence*: Frozen suite versions, margins, paired intervals, seed spread, over-refusal rate, and rollout/rollback plan.
- *Common Failure*: Reporting only target gain.
- *Diagnostic Hint*: What did the base model do that the adapted one no longer does?
- *Concept to Revisit*: Release Gates (Module 15); Canary Rollout (Module 17).

**Learning Outcome:**
Gate adapted models on target gain, retained capability, and safety with variance-aware paired evidence.

*(Effort: 40m instruction, 20m practice)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**
- [ZeRO](https://arxiv.org/abs/1910.02054) — Rajbhandari et al., 2019; model-state accounting.
- [InstructGPT](https://arxiv.org/abs/2203.02155) — Ouyang et al., 2022; SFT + reward model + RL.
- [Fine-Tuning or Retrieval?](https://arxiv.org/abs/2312.05934) — Ovadia et al., 2023; knowledge injection.
- [Fine-tuning Aligned Language Models Compromises Safety](https://arxiv.org/abs/2310.03693) — Qi et al., 2023.

**CURRENT DEFAULT:** per-failure-class adaptation decisions; [LoRA](https://arxiv.org/abs/2106.09685) (Hu et al., 2021) as the default parameter-efficient family; [DPO](https://arxiv.org/abs/2305.18290) (Rafailov et al., 2023) for offline preference tuning; run manifests; lineage-controlled data; three-part paired release gates.

**WORKLOAD-DEPENDENT:** [QLoRA](https://arxiv.org/abs/2305.14314) (Dettmers et al., 2023) for memory-constrained runs; LoRA rank and target choice ([LoRA Learns Less and Forgets Less](https://arxiv.org/abs/2405.09673), Biderman et al., 2024); full versus parameter-efficient tuning by data scale.

**FRONTIER:** [GRPO / DeepSeekMath](https://arxiv.org/abs/2402.03300) (Shao et al., 2024) and its corrections [Dr. GRPO](https://arxiv.org/abs/2503.20783) (Liu et al., 2025) and [DAPO](https://arxiv.org/abs/2503.14476) (Yu et al., 2025); [LoRA Without Regret](https://thinkingmachines.ai/blog/lora/) (Thinking Machines, 2025); [RL's Razor](https://arxiv.org/abs/2509.04259) (Shenfeld et al., 2025); [Fine-Tuning Lowers Safety and Disrupts Evaluation Consistency](https://arxiv.org/abs/2506.17209) (Fraser et al., 2025); [Alignment Dynamics in LLM Fine-Tuning](https://arxiv.org/abs/2605.18309) (Huang et al., 2026).

**LEGACY / INSUFFICIENT:** fine-tuning to inject changing facts; evaluating only on the target task; random splits of mined logs; single-seed safety evaluation; attention-only low-rank LoRA as an unexamined default.

**PRODUCTION SOURCE TRACE**
- Repository: `huggingface/peft`
- Revision: `b8674c86183a5dee38d0c3ede392e189593025e5` (committed 2026-09-25)
- Verified: 2026-09-30; static inspection only.
- File/symbols: `src/peft/tuners/lora/layer.py::{LoraLayer.update_layer, LoraLayer.reset_lora_parameters, Linear.forward, Linear.get_delta_weight, Linear.merge, Linear.unmerge}`.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY` and record a run manifest: base model hash, data snapshot IDs, template, loss mask, method/rank/alpha/targets, precision, seeds, hyperparameters, and evaluation suite versions.

### LAB A — Adaptation Decision Record
- **Objective**: For two failure classes (fact freshness and output format), compare prompting, retrieval, and a LoRA SFT adapter on quality, tokens per request, and update cost.
- **Pre-Registered Hypothesis**: CLM-013—RAG wins post-training-fact accuracy; SFT matches or beats prompting on format validity at lower per-request tokens.
- **Independent Variables**: Intervention, failure class, fact update after training.
- **Dependent Variables**: Accuracy on updated facts, format validity, tokens/request, training and index update cost.
- **Break & Falsify**: Update 20 facts after training; if SFT answers them without retraining, the freshness half is falsified for this setup.
- **Alignment**: Lesson 19.1.
- **Effort Estimate**: 3h total.

### LAB B — Memory and LoRA Mechanics
- **Objective**: Predict and measure peak memory for full, LoRA, and QLoRA steps on a ≤3B model; implement LoRA from scratch; verify parity with PEFT and merge parity; complete the pinned source trace.
- **Pre-Registered Hypothesis**: Predicted model-state memory is within 10% of measured after subtracting activations; merged and unmerged outputs agree within tolerance in fp32.
- **Independent Variables**: Method, sequence length, checkpointing, rank, alpha, dtype.
- **Dependent Variables**: Peak allocated/reserved memory, trainable parameters, step time, merge error.
- **Break & Falsify**: Change `r` without `alpha`; merge an adapter trained on a 4-bit base into a 16-bit base and measure output drift.
- **Alignment**: Lessons 19.2–19.3.
- **Effort Estimate**: 3.5h total (plus 2h source trace).

### LAB C — Data and Preference Optimization
- **Objective**: Build a leak-checked SFT set and preference pairs from mined failures; run SFT, DPO at three $\beta$ values, and a GRPO-style run on a verifiable task.
- **Pre-Registered Hypothesis**: Removing near-duplicates lowers the apparent target gain; lower $\beta$ increases KL to reference; with length-normalized GRPO, incorrect responses lengthen more than correct ones.
- **Independent Variables**: Dedup on/off, loss mask on/off, template variant, $\beta$, loss normalization, zero-advantage filtering.
- **Dependent Variables**: Target metric, contamination rate, KL to reference, length by correctness, zero-advantage fraction.
- **Break & Falsify**: Train with the wrong chat template and with no loss mask; report both effects on held-out validity.
- **Alignment**: Lessons 19.4–19.5.
- **Effort Estimate**: 3.5h total.

### LAB D — Forgetting and the Release Gate
- **Objective**: Compare LoRA at two ranks and target sets against full fine-tuning (or the highest-capacity feasible setting) on target, general, and safety suites over three seeds.
- **Pre-Registered Hypothesis**: Higher-capacity updates gain more on target and regress more outside it (CLM-006); safety refusal varies across seeds enough to change a single-seed decision (CLM-019).
- **Independent Variables**: Rank, target modules, learning rate, replay mix, seed.
- **Dependent Variables**: Paired target/general/safety deltas with intervals, over-refusal rate, KL to base on the target task.
- **Break & Falsify**: If no configuration shows forgetting, increase learning rate or epochs until one does, then show the gate catches it.
- **Alignment**: Lesson 19.6 and Incident 19.1.
- **Effort Estimate**: 3h total.

---

## 07 Break / Incident Scenarios

### Incident 19.1 — The Monthly Refresh

- **Incident Symptoms**: After the monthly adapter refresh for a support assistant, target-task CSAT rose in the canary. A week later, trust-and-safety reports that the assistant now complies with some requests it previously refused, and the tool team reports more malformed tool calls in long conversations. The refresh used 30% more mined data, raised rank from 16 to 64 with `alpha` unchanged, switched to QLoRA training, and merged into the 16-bit base for serving. The offline gate passed on one seed.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: benign-data safety regression; harmful or poisoned examples in newly mined data; forgetting from a larger effective update; train/serve template mismatch; 4-bit-trained adapter merged into a 16-bit base; wrong adapter version deployed; safety suite changed between runs; seed variance.
  2. *Rank Initial Plausibility*: Use the change list and the timing, but do not assign a cause without a discriminating test.
  3. *Identify Missing Evidence*: Both run manifests, data diff with source trust labels, rendered templates, suite versions, per-seed results, deployed adapter hash, and effective `scaling` in both runs.
  4. *Design Discriminating Tests*: Re-evaluate old and new adapters on identical frozen suites over three seeds; evaluate the new adapter unmerged on the 4-bit base versus merged; retrain on old data with new settings and on new data with old settings; audit new examples for harmful content.
  5. *Execute Causal Diagnosis*: Rank causes with the evidence each test produced; state what remains unexplained.
  6. *Prescribe Mitigation and Prevention*: Roll back to the previous adapter; fix manifest-level checks (scaling, template hash, merge precision); add multi-seed safety and long-conversation tool-call suites to the gate; add provenance filtering to the flywheel.
  7. *Remeasure*: Paired target/general/safety deltas with seed spread, and malformed tool-call rate in canary.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Adapting a Regulated Claims Assistant

An insurer's claims assistant must follow a strict adjudication procedure, output a structured decision record, cite current policy clauses that change quarterly, and refuse to give legal advice. Training data comes from adjuster-corrected transcripts, some submitted through a feedback tool open to external agents. One 48 GB GPU is available for training; serving hosts several tenant adapters on one base.

**Required Deliverables**:
1. Adaptation decision record per failure class, with alternatives and falsifiers.
2. Memory plan for the chosen method with predicted and measured peaks.
3. LoRA configuration (rank, alpha, targets, learning rate) with justification and a merged-versus-multi-adapter serving decision.
4. Data manifest with lineage, contamination report, template and loss-mask verification, and provenance filtering for externally submitted data.
5. Preference-optimization plan (or justified omission) with reward validity checks.
6. Three-part release gate with margins, seeds, and over-refusal checks.
7. Pinned PEFT source trace.
8. Diagnosis and remediation for Incident 19.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace
Trace LoRA creation, scaling, initialization, forward, delta computation, and merge/unmerge at the pinned PEFT commit. State what was statically inspected and what was executed in your parity tests.

### Rubric Dimensions
- **Adaptation Decision**: *Insufficient* fine-tunes by default. *Competent* compares alternatives per failure class. *Strong* pre-registers falsifiers and cost terms and tests them.
- **Memory and Mechanics**: *Insufficient* counts weights only. *Competent* predicts model states and LoRA parameters. *Strong* reconciles predictions with measurements and traces scaling and merge in source.
- **Data Integrity**: *Insufficient* uses random splits. *Competent* dedups and masks correctly. *Strong* separates by source/time, verifies templates against serving, and filters untrusted provenance.
- **Preference Optimization**: *Insufficient* reports reward curves. *Competent* tracks KL and margins. *Strong* diagnoses length, normalization, and reward-validity pathologies.
- **Release Gate**: *Insufficient* reports target gain. *Competent* adds general and safety suites. *Strong* uses paired intervals, margins, multiple seeds, and over-refusal checks.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Adaptation decision | 19.1 | LAB A | Mastery 1 | Decision records with falsifiers |
| Training memory estimation | 19.2 | LAB B | Mastery 2 | Predicted vs measured peaks |
| LoRA mechanics and source trace | 19.3 | LAB B | Mastery 3, 7 | Parity tests, PEFT trace |
| Leak-free, provenance-controlled data | 19.4 | LAB C | Mastery 4 / Incident | Data manifest and reports |
| Preference optimization diagnosis | 19.5 | LAB C | Mastery 5 | KL, margins, length, reward checks |
| Forgetting and safety gating | 19.6 | LAB D | Mastery 6 / Incident | Paired multi-seed gate report |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria
A learner successfully completing Module 19 must be able to:
1. Decide per failure class whether to prompt, retrieve, route, or adapt, with a falsifiable prediction.
2. Predict and verify training memory for full, LoRA, and QLoRA runs.
3. Implement and trace LoRA, including scaling, initialization, and merging.
4. Build adaptation data that survives leakage, template, loss-mask, and provenance checks.
5. Explain and diagnose RLHF, DPO, and GRPO-style objectives and their failure modes.
6. Gate an adapted model on target gain, retained capability, and safety with paired, multi-seed evidence.

### Module Wrap-Up (Final Mental Model Reconstruction)
- **The Core Invariant**: A weight update changes behavior on every future input, so it must be justified against per-request alternatives and gated on what the target metric does not measure.
- **The Adaptation Path**: failure class → decision → memory plan → data manifest → objective → paired three-part gate → canary → promote or roll back.
- LoRA trades capacity for retention; preference objectives are only as good as their rewards and normalization; safety is re-measured on every refresh.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

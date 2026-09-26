# Module 05 — Inference Optimization

## 00 Why This Module Exists

An optimization is useful only when it improves the declared production objective without violating numerical, quality, memory, reliability, or portability constraints. A smaller checkpoint may execute through a slow conversion path. A faster attention microbenchmark may be irrelevant to end-to-end latency. Speculation may reduce target-model calls yet lose after draft and verification costs. Fusing more work may reduce launches while creating a resource-heavy kernel.

This module teaches a disciplined optimization loop:

$$
\text{workload contract} \to \text{baseline profile} \to \text{bottleneck hypothesis}
\to \text{mechanism} \to \text{correctness/quality gate}
\to \text{end-to-end measurement} \to \text{break and falsify}.
$$

The scope is single-node inference mechanisms: IO-aware attention, kernel fusion and Triton fundamentals, FP8/INT8/INT4 quantization, SmoothQuant/GPTQ/AWQ, and speculative decoding including Medusa and EAGLE. Scheduling policy remains in Module 04, KV allocation internals in Module 03, and distributed inference specialization in Module 20. This module considers those systems only where an optimization changes their inputs or costs.

**Research cutoff:** 2026-09-26.

**Module Orientation**

- **Engineering problem:** choose, implement, compose, and defend inference optimizations for a pinned model, runtime, hardware target, workload distribution, and SLO.
- **What you will do:** compare reference and IO-aware attention; write and profile a fused Triton kernel; build and validate quantized artifacts; instrument speculative cycles; trace a current FlashAttention source path; and diagnose an optimization stack that regresses production goodput.
- **Evidence rule:** distinguish source observation (**O**), assumption-backed derivation (**D**), and telemetry-dependent engineering hypothesis (**H**). Paper speedups do not transfer without their model, hardware, shapes, configuration, baseline, and measurement boundary.

## 01 Baseline Assumptions

- **Model semantics (Module 01):** causal attention, MHA/GQA tensor shapes, autoregressive decoding, logits, sampling, and the difference between semantic equivalence and bitwise identity.
- **GPU performance (Module 02):** memory hierarchy, arithmetic intensity, Roofline bounds, launch overhead, warm-up and synchronization, kernel timelines, and profiler-counter limitations.
- **KV cache (Module 03):** logical KV payload, cache growth, layouts, and paged addressing. This module uses these facts but does not reteach allocation lifecycle.
- **Serving metrics (Module 04):** TTFT, ITL/TPOT, throughput, goodput, batching, arrival load, and scheduler interference. Optimization experiments must declare whether they are isolated kernels, offline model runs, or loaded serving runs.

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
  instruction: 6h
  guided_practice: 3h
  labs: 14h
  assessment: 3h
  source_trace: 2h
  total: 28h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
Model + workload + hardware + runtime revision
                    |
             establish baseline
                    |
     profile time, bytes, FLOPs, shapes, memory
                    |
       form competing bottleneck hypotheses
                    |
  +-----------------+-------------------+
  |                 |                   |
attention IO     kernel/launch       representation / decode
tiling           and fusion          algorithm
  |                 |                   |
FlashAttention   Triton/compiler     quantization/speculation
  +-----------------+-------------------+
                    |
 correctness + numerical + task-quality gates
                    |
 cold/warm + single/batched + load measurements
                    |
      end-to-end latency, throughput, goodput
                    |
        ablation, adversarial workload, rollback
```

Every branch consumes resources and can shift the next bottleneck. The optimization target is not “fewer bits,” “fewer kernels,” or “more accepted tokens” in isolation. It is a declared end-to-end objective measured at a stable boundary.

## 04 Lessons

### Lesson 5.1 — IO-Aware Exact Attention

**Engineering question:** How can dense attention retain its mathematical semantics while changing the memory traffic that dominates execution?

For one attention head, the reference expression is:

$$
S = QK^T/\sqrt{d}, \qquad P=\operatorname{softmax}(S), \qquad O=PV.
$$

A straightforward implementation can materialize the $N_q\times N_k$ score or probability matrix in HBM. FlashAttention instead tiles Q, K, and V, maintains online softmax statistics, and avoids full intermediate materialization. The original paper classifies this as an exact dense-attention algorithm: it does not prune attention entries or approximate the softmax. “Exact” does not mean bitwise-identical results because tiling and reduction order change floating-point behavior.

**Mechanistic invariants**

- Causal/local masks, scale, head layout, and GQA/MQA mapping must match the reference contract.
- The online softmax must maintain numerically stable row maxima and normalization across tiles.
- Backward support, dropout, head dimension, dtype, device generation, sequence shape, and variable-length packing are implementation capabilities, not properties implied by the name.
- A kernel can win its attention microbenchmark while leaving end-to-end inference unchanged if MLP, collectives, sampling, host work, or queueing dominates.

**Currentness:** the original IO-aware exact-attention mechanism is **REFERENCE**, while selection of a particular backend is **WORKLOAD-DEPENDENT**. FlashAttention-3 is Hopper-specific; FlashAttention-4 is a 2026 Blackwell-oriented **FRONTIER** design. Neither paper establishes portable speedups across other accelerators.

**Guided practice:** Build a shape matrix over batch, $N_q$, $N_k$, head dimension, causal mode, dtype, and GQA ratio. For each cell, predict the likely traffic advantage, validate outputs against a reference with declared tolerances, and measure warm kernel time, peak allocated memory, and end-to-end share. Identify at least one shape where dispatch or unsupported-path overhead weakens the expected benefit.

**Learning outcome:** explain FlashAttention as a tiled IO transformation, validate semantic coverage, and refuse to infer service speedup from an isolated favorable kernel result.

---

### Lesson 5.2 — Kernel Fusion and Triton Fundamentals

**Engineering question:** Which boundaries should be fused, and how do we know the fused kernel did not exchange one cost for another?

Vertical fusion can keep producer output on chip for a consumer and avoid intermediate HBM writes/reads. Horizontal fusion can combine independent small operations to amortize launch overhead. Compiler capture, legality, aliases, dynamic shapes, and graph breaks determine what can fuse. Generated code determines what actually fused.

A Triton kernel expresses a grid of program instances. Each program computes offsets, loads tiles under masks, transforms values, and stores results. Compile-time meta-parameters such as block sizes determine mappings and can be autotuned. This programming model lowers the barrier to custom kernels; it does not make an arbitrary kernel correct or fast.

**Fusion cost ledger**

$$
T_{candidate}=T_{launch}+T_{loads}+T_{compute}+T_{stores}+T_{sync}+T_{conversion}+T_{compile\ amortized}.
$$

This is an accounting decomposition, not a guarantee that terms add independently. Fusion targets launch and intermediate-memory terms, while potentially increasing register use, shared-memory use, synchronization, code size, compilation, or shape specialization.

**Required measurements**

- cold compile latency and warm steady-state latency;
- launch count and generated code;
- achieved bandwidth/FLOPs, registers, shared memory, occupancy, and spills where available;
- output error versus reference across adversarial sizes and boundary conditions;
- performance distribution over representative shapes, not one favorable tensor.

**Guided practice:** Fuse a bias-add plus activation or RMSNorm-like chain in Triton. Compare with a strong framework/compiler baseline. Sweep widths that are aligned and misaligned to the chosen block, and include a tiny tensor where launch overhead dominates and a large tensor where resource pressure matters. A slower fused result is valid evidence, not a failed lab.

**Learning outcome:** implement and inspect a fused kernel, then attribute its outcome to measured traffic, launch, compilation, or resource effects.

---

### Lesson 5.3 — Quantization Is a Deployment Contract

**Engineering question:** What exactly has changed when a model is called “FP8,” “INT8,” or “INT4”?

For uniform affine quantization:

$$
q=\operatorname{clamp}(\operatorname{round}(x/s)+z,q_{min},q_{max}),
\qquad \hat{x}=s(q-z).
$$

The scale $s$, zero point $z$, clipping range, rounding rule, and granularity may be per tensor, channel, group, token, or block. Symmetric quantization often fixes $z=0$. Floating formats instead define exponent/mantissa encodings and are still commonly paired with scaling policies. FP8 E4M3 and E5M2 trade precision and dynamic range differently; “FP8” alone is incomplete.

For $N$ logical weights at $w$ packed payload bits:

$$
M_{payload}=\frac{Nw}{8}\ \text{bytes}.
$$

This is a nominal lower bound. Add scales, zero points, padding, alignment, codebooks, duplicated weights, runtime workspaces, allocator fragmentation, and KV/activation memory before predicting device capacity. Do not convert the storage ratio into a latency ratio.

**Quantization manifest**

- model and tokenizer revisions;
- tensor coverage: weights, activations, KV, logits, or selected layers;
- encoding and group/granularity;
- calibration dataset and sampling procedure;
- clipping, smoothing, rounding, and exceptional layers;
- accumulator/output dtypes;
- packing layout, runtime, kernel/backend, and supported shapes;
- quality suite and numerical tolerances;
- memory, cold/warm latency, throughput, and goodput boundaries.

**Worked exercise (explicit assumption):** Suppose a hypothetical model has $N=8\times10^9$ weights, densely packed at 4 bits. The nominal payload is $4\times10^9$ bytes. This does not establish GB versus GiB reporting, checkpoint size, GPU allocation, or speed. List every additional byte category that must be measured before a deployment claim.

**Learning outcome:** specify low precision as a reproducible artifact/runtime contract and separate representation, arithmetic, memory, quality, and performance.

---

### Lesson 5.4 — SmoothQuant, GPTQ, and AWQ

**Engineering question:** Which error source does each post-training method address, and which serving path realizes its promised benefit?

- **SmoothQuant:** uses an equivalent offline rescaling to migrate activation-outlier difficulty into weights, enabling a W8A8 path. The smoothing parameter and calibration distribution influence the resulting ranges.
- **GPTQ:** performs one-shot weight quantization using approximate second-order information to control reconstruction error. The resulting artifact still needs a packing scheme and executable matrix kernel.
- **AWQ:** uses observed activations to identify/protect salient weights through searched per-channel scaling for low-bit weight-only quantization. Its system results include a tailored runtime.

These methods solve different problems. SmoothQuant is not “GPTQ for activations”; AWQ is not defined only by a bit width; and a GPTQ-format checkpoint does not prove that a selected runtime executes an optimized kernel rather than unpacking or dequantizing inefficiently.

**Evaluation matrix**

| Axis | Required evidence |
|---|---|
| Representation | actual tensor dtypes, scale tensors, group sizes, packed bytes |
| Numerical | layer/output error, NaN/Inf, outlier behavior, reference tolerance |
| Task quality | representative tasks, prompts, decoding settings, uncertainty |
| Kernel | selected operator/backend, conversion/dequant kernels, shape support |
| Memory | weights, metadata, workspace, activations/KV, peak and steady state |
| Serving | TTFT, ITL/TPOT, throughput, SLO-goodput at declared load |

**Break cases:** calibration drift; rare activation outliers; sensitive output heads; unsupported group size; small batches where unpack overhead dominates; long-prefill compute shapes where weight bandwidth is not dominant; and mixed batches that select a fallback path.

**Learning outcome:** select and falsify a quantization method based on error source, runtime support, workload, and quality—not label popularity.

---

### Lesson 5.5 — Exact Speculative Decoding and Its Cost Model

**Engineering question:** When does doing extra draft work reduce time per committed target token?

The reference algorithm uses a cheaper approximation model to draft multiple tokens. The target scores the continuation in parallel and applies a modified rejection/resampling procedure that preserves the target distribution under its assumptions. That guarantee belongs to the specified algorithm; relaxed thresholds, greedy variants, tree methods, or auxiliary heads need their own correctness/quality statement.

Let a cycle propose $k$ tokens, take

$$
T_{cycle}=T_{draft}(k)+T_{verify}(k)+T_{bookkeeping},
$$

and commit random reward $A$ output tokens. Under stationary ergodic renewal-style cycles with finite expectations:

$$
\text{token rate}=\frac{\mathbb{E}[A]}{\mathbb{E}[T_{cycle}]},
\qquad
\text{time/token}=\frac{\mathbb{E}[T_{cycle}]}{\mathbb{E}[A]}.
$$

This is not generally $\mathbb{E}[T_{cycle}/A]$. “Acceptance rate” is insufficient unless its denominator and proposal structure are defined. Record proposed tokens, committed tokens, first-rejection position, target calls, draft/verify times, synchronization, memory, and scheduler/batch context.

**Adversarial regimes**

- hard or distribution-shifted prompts reduce accepted tokens;
- a draft model is too large or resides on a contested device;
- verifier cost grows sharply with tree/block shape;
- high target batching already amortizes weight movement;
- extra model/KV/workspace memory reduces serving concurrency;
- long accepted bursts change streaming event semantics and ITL measurement.

**Guided practice:** With measured cycle records—not a supplied speedup—compute both ratio-of-means and mean-of-ratios, explain the difference, and compare against target-only decoding at identical output distribution, batch/concurrency, and timing boundaries.

**Learning outcome:** prove or disprove speculative benefit using total cycle reward/cost and explicit equivalence guarantees.

---

### Lesson 5.6 — Medusa, EAGLE, and Learned Drafting

**Engineering question:** How do proposal architecture and training change the speculative trade space?

Medusa adds decoding heads that propose multiple future tokens and uses tree-based verification. Its variants differ in whether the backbone remains frozen and in their quality/training trade-offs. EAGLE predicts at the target model's feature level with a shifted token sequence to address feature uncertainty. EAGLE-3 changes the learned-draft design to direct token prediction with multi-layer feature fusion and reports both latency and batched-serving experiments.

The comparison contract is broader than speed:

- target/draft compatibility and retraining requirements;
- artifact and serving-memory overhead;
- candidate-tree width/depth and verification shape;
- exact-distribution versus declared approximate acceptance;
- acceptance by prompt domain and decoding settings;
- single-request latency versus batched throughput;
- failure/fallback behavior and rollout complexity.

Treat Medusa, EAGLE, and later variants as **WORKLOAD-DEPENDENT** or **FRONTIER**, not universal defaults. Their paper results motivate experiments but do not select a production method without replication on the target stack.

**Learning outcome:** distinguish speculative families mechanistically and design a fair comparison that includes training, quality, memory, batching, and operations.

---

### Lesson 5.7 — Bottleneck-Driven Composition and Diagnosis

**Engineering question:** How do we compose optimizations without losing causal attribution?

Start with a pinned baseline. Measure the fraction of time and resources attributable to the target. Apply one mechanism. Re-run correctness/quality gates. Then measure at kernel, model, and loaded-serving boundaries. Only after main effects are understood should combinations be tested.

For identical boundaries:

$$
S_{e2e}=\frac{T_{baseline}}{T_{optimized}}.
$$

An Amdahl-style upper bound is useful only if non-target work remains unchanged. Quantization can change kernel selection; fusion can change layouts; speculation changes decode shapes and scheduler work; attention kernels can change memory headroom. These interactions violate a naïve fixed-fraction assumption.

Use the incident protocol:

$$
\text{symptom}\to\text{competing hypotheses}\to\text{missing evidence}
\to\text{discriminating measurement}\to\text{ranked explanation}
\to\text{intervention}\to\text{remeasurement}.
$$

**Necessary but insufficient signals**

- lower model-weight bytes do not prove lower latency;
- fewer launches do not prove faster kernels;
- higher occupancy does not prove more useful work;
- higher acceptance percentage does not prove speculative speedup;
- lower kernel time does not prove better TTFT, ITL, throughput, or goodput;
- close perplexity does not prove application-quality equivalence.

**Learning outcome:** construct an optimization evidence chain that survives bottleneck shifts, interactions, and adversarial workloads.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- *FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness* — Dao, Fu, Ermon, Rudra, and Ré (2022). Read the IO model and online-softmax tiling, not only headline speedups.
- *FP8 Formats for Deep Learning* — Micikevicius et al. (2022). Use for E4M3/E5M2 format semantics; scaling and kernel policy remain separate.
- *SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models* — Xiao et al. (ICML 2023).
- *GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers* — Frantar et al. (ICLR 2023).
- *Fast Inference from Transformers via Speculative Decoding* — Leviathan, Kalman, and Matias (ICML 2023), alongside Chen et al., *Accelerating Large Language Model Decoding with Speculative Sampling* (2023).

**WORKLOAD-DEPENDENT**

- *AWQ: Activation-aware Weight Quantization for On-Device LLM Compression and Acceleration* — Lin et al. (MLSys 2024).
- *Medusa: Simple LLM Inference Acceleration Framework with Multiple Decoding Heads* — Cai et al. (ICML 2024).
- *EAGLE: Speculative Sampling Requires Rethinking Feature Uncertainty* — Li et al. (2024; arXiv revision 2025).
- Triton and compiler fusion are current tool/mechanism families; an implementation is selected only after correctness and workload-specific measurement.

**FRONTIER**

- *FlashAttention-3* — Shah et al. (2024), specialized for Hopper.
- *EAGLE-3* — Li et al. (2025), learned direct-token drafting with multi-layer feature fusion.
- *FlashAttention-4* — Zadouri et al. (2026), Blackwell-oriented algorithm/kernel pipeline co-design. Do not treat its paper benchmarks as a default for other devices.

**PRODUCTION SOURCE TRACE**

- Repository: `Dao-AILab/flash-attention`
- Revision: `e9cf2c1651d2303191eb40a739a3c135fda00999`
- Verified: 2026-09-26 by static inspection; not executed here.
- File: `flash_attn/flash_attn_interface.py`
- Symbols: `flash_attn_func`, `flash_attn_varlen_func`, `flash_attn_with_kvcache`.
- Trace: public Python function to `FlashAttnFunc`/`FlashAttnVarlenFunc.apply`, or the KV-cache wrapper to `flash_attn_gpu.fwd_kvcache`, then the selected compiled CUDA/ROCm backend.
- Scope: this proves behavior of the pinned upstream interface, not every wheel, fork, backend, or serving runtime.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow:

$$
\text{PREDICT}\to\text{BUILD}\to\text{MEASURE}\to\text{EXPLAIN}
\to\text{BREAK}\to\text{IMPROVE}\to\text{FALSIFY}.
$$

### LAB A — Attention Parity and Shape-Regime Map

- **Objective:** compare a trusted reference attention path with an available IO-aware path over a pre-registered shape matrix.
- **Independent variables:** device, dtype, batch, query/key lengths, head dimension, causal/local mask, MHA/GQA ratio, and packed/variable-length representation.
- **Measurements:** correctness tolerance, NaN/Inf, warm kernel time, cold dispatch/compile time, peak memory, achieved bandwidth/FLOPs where reliable, and end-to-end attention share.
- **Break/falsify:** include tiny queries, unsupported or fallback-prone head sizes, ragged sequences, extreme logits, and shapes where attention is a small end-to-end fraction. Falsify “IO-aware always wins” with a reproducible counterexample if one exists.
- **Required artifact:** pinned environment, raw shape-level results, profiler traces for at least three regimes, and a source trace of the pinned interface.
- **Effort:** 4h.

### LAB B — Fusion and Triton Resource Trade-Off

- **Objective:** implement one fused inference kernel and compare it with eager and compiler-generated strong baselines.
- **Independent variables:** block size, warps/stages where applicable, width, batch, aligned/misaligned dimensions, contiguous/strided layout, and cold/warm execution.
- **Measurements:** numerical error, launch count, compile/recompile events, kernel time, end-to-end time, traffic counters, registers/shared memory, occupancy, and spills where supported.
- **Break/falsify:** find a shape where fusion regresses; determine whether launch, compilation, fallback, memory access, or resource pressure explains it. If counters do not support the initial mechanism, reject it.
- **Required artifact:** kernel, tests, generated-code excerpt or trace, benchmark protocol, uncertainty, and rollback criterion.
- **Effort:** 4h.

### LAB C — Quantized Artifact: Memory, Quality, and Kernel Reality

- **Objective:** produce at least two quantized configurations that differ in method or granularity and evaluate them against the same baseline.
- **Pre-registration:** declare acceptable task-quality loss, memory target, and practically important performance effect before running tests.
- **Measurements:** actual file and tensor bytes, scale/metadata bytes, peak/steady device memory, selected kernels, conversion/dequant time, numerical error, application-quality suite, TTFT, TPOT, throughput, and SLO-goodput.
- **Break/falsify:** evaluate calibration-domain shift, outlier prompts, small and large batches, short decode and long prefill, and a shape that triggers fallback. Falsify any claim that nominal bit width predicts proportional latency.
- **Required artifact:** complete quantization manifest, calibration provenance, quality confidence intervals or justified uncertainty method, profiler trace, and deployment compatibility table.
- **Effort:** 4h.

### LAB D — Speculative Decoding Reward/Cost Surface

- **Objective:** implement or instrument target-only and one speculative path, then measure full cycle reward/cost across proposal lengths and workloads.
- **Independent variables:** proposal length, draft family/size, prompt domain, sampling settings, output length, batch/concurrency, and scheduler load.
- **Measurements:** proposed and committed tokens, first rejection, draft time, verification time, bookkeeping/synchronization, target calls, memory, client-visible token-event timing, throughput, and goodput.
- **Break/falsify:** use a low-acceptance domain, an expensive draft, and high target batching. Locate regions where speculation loses and explain them using the complete cost ledger, not acceptance alone.
- **Required artifact:** cycle-level records, ratio-of-means calculation with assumptions, output-distribution or declared quality check, and an adaptive enable/disable policy supported by measurements.
- **Effort:** 4h.

## 07 Break / Incident Scenarios

### Incident 05.1 — The “Optimized” Release Loses Goodput

A release simultaneously enables weight-only INT4, compiler fusion, and speculative decoding. Offline model memory falls and a decode microbenchmark improves, yet loaded production shows higher P99 TTFT, intermittent ITL stalls, more out-of-memory restarts, and lower SLO-goodput. Some prompt domains also show a small but disputed quality change.

The incident deliberately does not identify one root cause. The learner must:

1. **Form competing hypotheses:** unsupported quantized shapes causing dequantization; fusion recompilation or register pressure; speculative draft/verification overhead; extra draft/workspace/KV memory reducing concurrency; calibration drift; scheduler interaction; or an unrelated traffic/runtime change.
2. **Identify missing evidence:** artifact manifests, selected-kernel logs, cold/warm traces, graph breaks/recompiles, generated kernels, GPU counters, memory breakdown, cycle-level acceptance/cost, prompt-domain labels, request queues, and matched quality results.
3. **Design discriminating experiments:** reproduce a pinned workload and use a factorial toggle matrix for INT4, fusion, and speculation. Preserve model/runtime/configuration except the declared factor and include interaction terms.
4. **Rank explanations:** use lead-lag timing and ablation effect sizes. Do not infer causality from occupancy, acceptance, or allocation alone.
5. **Intervene:** roll back or gate only the unsupported mechanism, constrain shapes/domains, or reserve memory based on evidence.
6. **Remeasure:** repeat the same correctness, quality, latency, throughput, memory, and goodput protocol; define pass/fail and automatic rollback thresholds before redeployment.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Transfer Problem — Defend an Optimization Portfolio

You own one model service with two declared workload classes: short interactive conversations and long-prompt offline generation. A hardware refresh provides a newer GPU generation, and stakeholders propose enabling the newest attention kernel, INT4 weights, graph fusion, and learned speculative drafting simultaneously.

Produce an evidence-backed optimization plan with:

1. a pinned baseline and workload distribution, including prompt/output correlations, arrival or concurrency model, batch shapes, and SLO-goodput definition;
2. a profile-based bottleneck model for each workload class;
3. attention, fusion, quantization, and speculation candidates with explicit applicability and failure conditions;
4. derivations for nominal weight payload and speculative cycle throughput with units and assumptions;
5. a correctness/numerical/task-quality gate appropriate to each mechanism;
6. a main-effect and interaction experiment design with warm/cold and isolated/loaded boundaries;
7. source inspection of one production kernel/interface at an exact revision;
8. a decision table that accepts, rejects, or gates each optimization by shape/domain/load;
9. deployment canary, observability, rollback thresholds, and a falsifying workload shift;
10. a defense explaining why stacking every locally favorable optimization is not an engineering conclusion.

No supplied speedup is accepted as a substitute for measurement. If the available evidence cannot support a deployment claim, record a bounded open verification item with the exact missing experiment.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

The reference trace uses FlashAttention commit `e9cf2c1651d2303191eb40a739a3c135fda00999`. The learner must record repository, revision, verification date, file, symbol, entry point, backend dispatch, configuration conditions, observed behavior, static-versus-executed status, and whether each conclusion generalizes.

### Rubric Dimensions

- **Mechanistic reasoning:** distinguishes mathematical function, representation, kernel, runtime dispatch, and serving behavior.
- **Quantitative reasoning:** supplies equations, units, assumptions, boundaries, and exclusions; does not convert bit ratio or acceptance rate into a performance claim.
- **Correctness and quality:** tests numerical parity and application quality with declared tolerances, populations, decoding settings, and uncertainty.
- **Experimental rigor:** pins revisions/configuration; warms and synchronizes correctly; uses representative shape/workload distributions; reports raw results and variability.
- **Failure diagnosis:** ranks competing causes through discriminating measurements and controlled ablations; allows interacting bottlenecks and transitions.
- **Architecture defense:** selects or gates mechanisms by evidence and defines operational rollback rather than naming a fashionable stack.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| IO-aware attention and parity | Lesson 5.1 | LAB A | Mastery 3, 5, 7 | Shape map, parity tests, source trace |
| Fusion and Triton kernel reasoning | Lesson 5.2 | LAB B | Incident / Mastery 3, 6 | Kernel, generated code, profile |
| Quantization contract and methods | Lessons 5.3–5.4 | LAB C | Incident / Mastery 3–6 | Manifest, quality and kernel report |
| Speculative reward/cost modeling | Lessons 5.5–5.6 | LAB D | Mastery 3–6 | Cycle telemetry and falsification map |
| Optimization composition diagnosis | Lesson 5.7 | All labs | Incident / Mastery 6, 8–10 | Factorial ablation and rollback plan |

## 11 Exit Criteria & Module Wrap-Up

A learner passes when they can:

1. explain how IO-aware attention changes traffic while preserving dense-attention semantics within numerical tolerance;
2. write, test, profile, and break a fused GPU kernel;
3. specify a quantized artifact beyond its bit width and measure its real memory, quality, and executable kernel path;
4. distinguish SmoothQuant, GPTQ, and AWQ by mechanism and applicability;
5. derive and measure speculative cycle reward/cost without treating acceptance rate as speedup;
6. compare draft families by training, quality, memory, batch, and operational trade-offs;
7. trace a current production source path at a pinned revision;
8. diagnose an interacting optimization regression and defend a rollback with remeasurement.

**Final mental model:** inference optimization is constrained resource transformation. It changes bytes, operations, launch structure, precision, or serial target calls, then hands a new workload to the rest of the system. Validate semantics and quality first; measure the transformed bottleneck at the end-to-end boundary; retain the optimization only where its full cost ledger improves the declared objective.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

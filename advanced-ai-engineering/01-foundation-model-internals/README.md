# Module 01 — Foundation Model Internals

## 00 Why This Module Exists

Serving, caching, profiling, quantization, and distributed placement all depend on model geometry. An engineer who cannot reconstruct the forward pass will miscount parameters, confuse logical FLOPs with latency, allocate the wrong KV shape, or optimize an implementation detail that the selected architecture does not have.

This module builds a mechanistic decoder-only model while refusing the common shortcut that every modern model is “basically Llama.” Llama-style pre-norm RMSNorm, RoPE, GQA, and SwiGLU form a useful current reference, not a universal definition. Sparse experts and latent-attention families extend the model and change the accounting.

**Module orientation**

- **Engineering problem**: Translate a model configuration into correct tensor shapes, equations, parameter counts, leading FLOP terms, numerical behavior, and downstream systems constraints.
- **What you will do**: Implement a minimal decoder layer, prove its shape invariants, test causal isolation and RoPE, compare MHA/MQA/GQA, reconcile analytical counts with real parameters, profile deviations from the model, and trace a pinned Transformers Llama forward path.
- **Research cutoff**: 2026-09-25. Current implementation claims are pinned to exact source revisions.

## 01 Prerequisites and Scope

Prerequisites: linear algebra, matrix multiplication, softmax, basic PyTorch, and Module 00 claim discipline.

This module owns architecture semantics and logical accounting. It cross-references but does not replace:

- KV-cache lifecycle, paging, reuse, and eviction — Module 03;
- GPU execution, memory hierarchy, and profiler reasoning — Module 02;
- kernel optimization, quantization, and speculative decoding — Module 05;
- tensor/pipeline/expert/context parallelism — Module 20.

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
  statistical: SELECTIVE
  production_reasoning: REQUIRED
  failure_analysis: REQUIRED
  falsification: REQUIRED
  security: NOT_APPLICABLE
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: REQUIRED

estimated_effort:
  instruction: 6h
  guided_practice: 3h
  labs: 12h
  assessment: 3h
  source_trace: 2h
  total: 26h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Core Mental Model

```text
token IDs [B,S]
  → embeddings [B,S,d]
  → repeated decoder blocks
       RMSNorm
       Q/K/V projections + position transform
       causal attention + output projection
       residual add
       RMSNorm
       gated or expert MLP
       residual add
  → final norm
  → vocabulary projection
  → logits [B,S,V]
```

Each arrow has four views that must agree:

1. **Semantics**: what dependency or transformation is intended?
2. **Shape**: what are the axes, sizes, strides, and broadcast rules?
3. **Accounting**: which weights and operations are logically required?
4. **Implementation**: what source path, dtype, backend, workspace, and fusion actually realize it?

A mismatch between these views is a diagnostic signal, not something to paper over with a familiar asymptotic formula.

## 04 Lessons

### Lesson 1.1 — Decoder Block and Residual Stream

**Engineering question:** How does information move from input tokens to logits through a pre-normalized decoder?

Let input token IDs have shape $[B,S]$, model width $d$, and vocabulary size $V$. Embedding lookup produces

$$X_0\in\mathbb{R}^{B\times S\times d}.$$

A Llama-style pre-normalized layer can be written as

$$U_l=X_l+\mathrm{Attention}(\mathrm{RMSNorm}(X_l)),$$
$$X_{l+1}=U_l+\mathrm{MLP}(\mathrm{RMSNorm}(U_l)).$$

After $L$ layers, a final normalization and language-model head produce logits:

$$Z=\mathrm{LMHead}(\mathrm{RMSNorm}(X_L))\in\mathbb{R}^{B\times S\times V}.$$

This is a reference architecture. Other decoders may use post-norm, parallel attention/MLP branches, additional norms, convolution or state-space blocks, different residual scaling, or shared layers.

**Invariants**

- Both residual additions require identical logical shapes.
- The causal dependency of position $i$ must not change when only tokens at positions $>i$ change.
- Vocabulary projection and input embedding may be tied or independent; parameter counting must inspect configuration and actual tensors.

**Guided practice:** Install hooks around every layer of a small model. Record shape, dtype, device, contiguity, min/max, finite count, and residual norm. Compare the trace with the equations before profiling speed.

---

### Lesson 1.2 — Causal Attention and MHA/MQA/GQA Geometry

**Engineering question:** Which axes are shared, and which remain per query head?

For hidden states $X\in\mathbb{R}^{B\times S\times d}$, define query heads $H_q$, key/value heads $H_{kv}$, and head width $d_h$, with $d=H_qd_h$ for the standard case:

$$Q\in\mathbb{R}^{B\times H_q\times S\times d_h},$$
$$K,V\in\mathbb{R}^{B\times H_{kv}\times S_k\times d_h}.$$

If $g=H_q/H_{kv}$ is an integer, query head $h$ uses K/V group $\lfloor h/g\rfloor$. The logical attention for a query head is

$$A_h=\mathrm{softmax}\left(\frac{Q_hK_{group(h)}^\top}{\sqrt{d_h}}+M\right),$$
$$O_h=A_hV_{group(h)}.$$

For a simple no-prefix causal mask,

$$M_{ij}=\begin{cases}0,&j\le i\\-\infty,&j>i.\end{cases}$$

With cached prefixes, padding, packed sequences, sliding windows, or special attention patterns, absolute positions and valid-key boundaries require a richer mask.

- **MHA**: $H_{kv}=H_q$.
- **MQA**: $H_{kv}=1$.
- **GQA**: $1<H_{kv}<H_q$ in the intermediate case.

Fewer KV heads reduce K/V projection width and logical KV payload. They do not remove query heads, output projection work, softmax, or attention over context. A runtime can map groups without physically repeating K/V; a naive `repeat` may create avoidable memory traffic.

**Break test:** Flip the mask orientation or permute head and sequence axes while preserving the element count. Random-output smoke tests may pass. A prefix-invariance test and comparison against a reference attention implementation should fail.

---

### Lesson 1.3 — Rotary Position Embedding

**Engineering question:** How can a position transformation alter attention scores without adding a position vector to the residual stream?

For each paired feature coordinate, RoPE applies a rotation at position $p$:

$$R(p\theta)=\begin{bmatrix}\cos(p\theta)&-\sin(p\theta)\\\sin(p\theta)&\cos(p\theta)\end{bmatrix}.$$

Queries and keys are rotated before their dot product. Orthogonality gives

$$\left(R(p\theta)q\right)^\top\left(R(r\theta)k\right)=q^\top R((r-p)\theta)k,$$

so the score can depend on relative displacement. Real models choose a frequency schedule, rotary fraction, coordinate pairing convention, maximum training positions, and possibly a scaling method.

**Do not infer:** “RoPE supports unlimited context.” Mathematical rotations can be evaluated at new positions, but quality and numerical behavior beyond training length are empirical properties of the trained model and scaling scheme.

**Guided practice:** Test norm preservation of rotated vectors, relative-shift behavior, dtype sensitivity at large positions, and parity with the pinned reference. Include an intentionally wrong pairing convention.

---

### Lesson 1.4 — RMSNorm, SwiGLU, and Numerical Details

**RMSNorm** over the final feature axis:

$$\mathrm{RMSNorm}(x)=\gamma\odot\frac{x}{\sqrt{\frac{1}{d}\sum_{j=1}^{d}x_j^2+\epsilon}}.$$

Unlike LayerNorm, the expression does not subtract the feature mean. The current pinned Transformers Llama implementation converts hidden states to float32 for the mean-square and reciprocal-square-root calculation, casts normalized values back to the input dtype, and multiplies by the learned weight. A substituted fused kernel may realize equivalent semantics through a different execution path.

**SwiGLU** with intermediate width $m$:

$$\mathrm{SwiGLU}(x)=W_{down}\left(\mathrm{SiLU}(W_{gate}x)\odot(W_{up}x)\right).$$

Bias and orientation notation depend on the framework. In row-major PyTorch notation, `nn.Linear(d,m)` stores a weight shaped $[m,d]`; the scalar parameter count is still $dm$.

**Failure modes**

- applying epsilon outside the square root;
- reducing across the wrong axis;
- computing low-precision squares that overflow or underflow;
- swapping gate/up branches when the activation is not symmetric;
- applying normalization after rather than before the sublayer;
- comparing outputs without aligning dtype and tolerance.

---

### Lesson 1.5 — Parameter and FLOP Accounting

**Engineering question:** Which parts are exact logical counts, and which parts are performance hypotheses?

Assume dense, bias-free projections, $d=H_qd_h$, and no padding or adapters.

**Attention projection parameters**

$$P_Q=d(H_qd_h)=d^2,$$
$$P_K=P_V=d(H_{kv}d_h),\qquad P_O=d^2,$$
$$P_{attn}=2d^2+2dH_{kv}d_h=2d^2\left(1+\frac{H_{kv}}{H_q}\right).$$

**SwiGLU parameters**

$$P_{mlp}=dm+dm+md=3dm.$$

Two RMSNorm weights contribute $2d$ per standard block. Add embeddings, final norm, LM head, biases, adapters, expert/router tensors, and any untied output head separately.

**Worked parameter example**

For $d=4096$, $H_q=32$, $H_{kv}=8$, $d_h=128$, and $m=11008$:

- attention projections: $41,943,040$ parameters;
- dense SwiGLU: $135,266,304$ parameters;
- two RMSNorm weights: $8,192$ parameters;
- reference block subtotal: $177,217,536$ parameters.

This subtotal excludes embeddings, final norm, LM head, biases, storage padding, and all runtime memory.

**FLOP convention**

State whether one fused multiply-add counts as one or two operations. This module uses two:

$$\mathrm{GEMM}(m\times k,k\times n)\approx2mkn\text{ FLOPs}.$$

For a dense full-sequence attention calculation, QK and AV together have a nominal leading term

$$4BH_qS^2d_h=4BS^2d.$$

A causal-aware implementation may avoid part of the nominal square, and a fused kernel may never materialize the score matrix. Projections and MLP scale linearly with $S$ but can dominate at common dimensions. Logical FLOPs do not determine latency without shapes, backend, precision, achieved compute/bandwidth, and memory traffic.

**OOM discipline:** Logical tensors provide component estimates, not an exact OOM boundary. Measure allocated and reserved peaks across fresh processes and record workspaces, backend, cache state, graph capture, padding, and concurrent allocations.

---

### Lesson 1.6 — Sparse Experts and Latent Attention

**Sparse MoE** replaces a dense MLP with a router and multiple expert MLPs. For $E$ bias-free SwiGLU experts of width $m$, a simplified total expert count is

$$P_{experts,total}=3Edm.$$

If top-$k$ experts execute per token, the raw selected expert weight use is proportional to $3kdm$, plus router and shared components. This is not a wall-clock model. Expert imbalance, capacity, padding, token dropping/rerouting, locality, batching, and communication matter.

Always distinguish:

- total parameters;
- resident parameters per device/process;
- parameters selected per token;
- operations actually executed after padding/capacity decisions.

**MLA** uses architecture-specific latent projections to compress attention state. It is not merely GQA with a smaller $H_{kv}$. Study its projection/decompression equations separately and defer concrete cache layout to Module 03.

**Currentness classification**

- **REFERENCE / BASELINE**: scaled causal MHA and dense decoder blocks.
- **COMMON CURRENT PATTERN, NOT UNIVERSAL**: pre-norm RMSNorm, RoPE, SwiGLU, and GQA in many open decoder families.
- **WORKLOAD- AND MODEL-DEPENDENT**: MQA versus GQA versus MHA; dense versus sparse experts.
- **FRONTIER / ARCHITECTURE-SPECIFIC**: MLA and newer hybrid attention/state-space designs.
- **REJECT AS A MODEL**: “all current LLMs are Llama with different weights.”

---

### Lesson 1.7 — Production Source Trace and Diagnosis

At Transformers commit `11c16613d93911300c38ec8c9c2460567be53281`, trace:

```text
LlamaForCausalLM.forward
  → LlamaModel.forward
      → embed_tokens
      → create_causal_mask
      → LlamaRotaryEmbedding
      → each LlamaDecoderLayer.forward
          → input_layernorm
          → LlamaAttention.forward
              → q_proj / k_proj / v_proj
              → apply_rotary_pos_emb
              → optional Cache.update
              → ALL_ATTENTION_FUNCTIONS backend interface
              → o_proj
          → residual add
          → post_attention_layernorm
          → LlamaMLP.forward
          → residual add
      → final LlamaRMSNorm
  → lm_head
```

The source file defines orchestration and an eager reference path, but configured attention may dispatch to a different backend. Record the selected backend before making a kernel claim.

Use a diagnostic chain:

```text
SYMPTOM
  → competing shape / mask / position / dtype / backend / memory hypotheses
  → missing tensor and timeline evidence
  → discriminating invariant or intervention
  → ranked explanation
  → fix
  → parity and workload remeasurement
```

Examples of discriminating tests:

- Future-token perturbation changes earlier logits → mask or packing hypothesis.
- FP32 norm path fixes non-finite outputs → precision hypothesis strengthened.
- Eager backend matches reference but optimized backend diverges → backend path strengthened.
- Analytical parameters differ from named unique storage → tying, sharing, padding, or omitted tensors.
- OOM boundary shifts across fresh processes with unchanged shapes → allocator/workspace/state hypothesis.

## 05 Literature and Source Map

**REFERENCE / BASELINE**

- Vaswani et al. (2017), *Attention Is All You Need* — scaled dot-product and multi-head attention.
- Zhang and Sennrich (2019), *Root Mean Square Layer Normalization* — RMSNorm mechanism.
- Shazeer (2020), *GLU Variants Improve Transformer* — SwiGLU family.
- Su et al. (2021), *RoFormer* — rotary position embedding.

**CURRENT ARCHITECTURE EXTENSIONS**

- Shazeer (2019), *Fast Transformer Decoding: One Write-Head is All You Need* — MQA.
- Ainslie et al. (2023), *GQA* — grouped-query attention and uptraining study.
- Fedus, Zoph, and Shazeer (2022), *Switch Transformers* — sparse routing reference.
- DeepSeek-AI (2024), *DeepSeek-V2* — MLA and a modern MoE family; empirical results remain setup-specific.

**CURRENT SOURCE SNAPSHOT**

- Hugging Face Transformers commit `11c16613d93911300c38ec8c9c2460567be53281`, `src/transformers/models/llama/modeling_llama.py`, verified 2026-09-25.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

### LAB A — Minimal Decoder and Numerical Parity

- **Objective**: Implement RMSNorm, RoPE, causal attention, GQA mapping, SwiGLU, residuals, final norm, and logits without using a high-level decoder block.
- **Controls**: fixed tiny configuration, deterministic weights/inputs, explicit dtype and tolerance, no dropout.
- **Required tests**: shape assertions, finite outputs, future-token prefix invariance, norm/rotation properties, gradient check where applicable, and output parity with a transparent reference.
- **Break**: wrong mask orientation, wrong RMSNorm axis, epsilon outside the root, and a head/sequence transpose that preserves element count.

### LAB B — MHA/MQA/GQA and RoPE Boundary Study

- **Objective**: Compare parameter geometry, logical KV dimensions, output parity at equivalent weights where meaningful, and measured runtime across MHA/MQA/GQA.
- **Independent variables**: $H_{kv}$, batch, prompt length, decode context, backend, and position range.
- **Evidence**: projection counts, tensor shapes, backend identity, allocated/reserved memory, kernel timeline, and quality/parity limitations.
- **Falsify**: find a workload where fewer KV heads do not improve end-to-end latency, or a position range where a naive RoPE extrapolation claim fails.

### LAB C — Accounting Versus Real Execution

- **Objective**: Generate an analytical manifest for every parameter group and leading matmul, then reconcile it with `named_parameters`, profiler shapes, and peak memory.
- **Design**: sweep $B$, $S$, $d$, $m$, $H_q$, and $H_{kv}$ on configurations small enough to reproduce.
- **Break**: enable biases, tie/untie embeddings, add an adapter, change backend/workspace, and introduce an MoE layer.
- **Falsification**: demonstrate at least one case where fewer logical FLOPs or parameters does not yield proportional latency or memory savings.

### LAB D — Pinned Transformers Source Trace

- **Objective**: Execute the pinned Llama path with hooks and map every source symbol to the mathematical operation and tensor contract.
- **Required trace**: entry point, configuration fields, causal-mask construction, RoPE, layer order, cache call boundary, attention backend dispatch, final norm, and LM head.
- **Artifact**: repository, commit, verification date, file, symbol, entry point, execution path, selected backend, exact commands, and `TODO_VERIFY` for paths not run.

## 07 Break / Incident Scenario

### Incident 01.1 — Plausible Logits, Corrupt Long Context

A converted model matches short-prompt top-1 tokens but loses quality beyond 8K tokens and occasionally produces non-finite activations in low precision. Parameter count appears close to the source checkpoint.

Investigate competing explanations:

1. wrong RoPE frequencies, scaling, pairing, or position IDs;
2. incorrect $H_{kv}$ grouping or reshape layout;
3. causal/padding-mask broadcast error;
4. RMSNorm epsilon or accumulation dtype mismatch;
5. tied versus untied output weights or missing parameters;
6. optimized attention backend discrepancy;
7. checkpoint/config mismatch unrelated to the suspected mechanism.

Required response:

- localize the first layer and position where parity diverges;
- compare eager and optimized backends;
- run future-token perturbation and relative-position invariants;
- inspect norms, finite counts, shapes, dtypes, and configuration;
- rank explanations and state what remains unresolved;
- fix one mechanism and rerun the same short- and long-context tests.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

Given an unfamiliar decoder configuration and a pinned implementation, produce:

1. a forward-pass graph with exact logical shapes and residual boundaries;
2. attention, MLP, norm, embedding, head, router/expert, and total parameter counts with exclusions;
3. leading FLOP equations with a declared FMA convention;
4. MHA/MQA/GQA or MLA classification without forcing incompatible formulas;
5. a causal-mask and position-encoding invariant test suite;
6. a minimal executable reference for one layer;
7. a source trace showing backend dispatch and numerical dtype choices;
8. a profiler comparison explaining where analytical work and measured latency diverge;
9. an OOM prediction expressed as a component estimate plus empirical boundary, not fake exactness;
10. downstream implications for Modules 02–05 and 20 with scope boundaries preserved.

## 09 Required Evidence and Rubric

- **Mechanistic reasoning**: Strong work explains axes, grouping, residual order, mask semantics, and position transforms without relying on architecture names alone.
- **Quantitative reasoning**: Strong work derives counts from matrices, states bias/tie/padding/FMA conventions, and reconciles formulas with actual tensors.
- **Implementation**: Strong work passes invariants and parity checks and records dtype/backend details.
- **Failure analysis**: Strong work uses competing hypotheses and localizes the first divergence instead of guessing from final logits.
- **Transfer**: Strong work explains which conclusions generalize across decoders and which are Llama-, MoE-, MLA-, backend-, or workload-specific.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Decoder forward path | 1.1 | Labs A/D | Mastery 1, 6-7 | Shape/source trace |
| Attention geometry | 1.2 | Labs A/B | Mastery 1, 4-5 | Tensor invariants |
| RoPE mechanics | 1.3 | Labs A/B | Mastery 5 | Rotation/position tests |
| RMSNorm and SwiGLU | 1.4 | Lab A | Mastery 6-7 | Numerical parity |
| Parameter/FLOP accounting | 1.5 | Lab C | Mastery 2-3, 8-9 | Analytical manifest |
| MoE and MLA distinctions | 1.6 | Labs B/C | Mastery 4, 10 | Architecture comparison |
| Source-based diagnosis | 1.7 | Lab D / Incident | Mastery 7-8 | Pinned source trace |

## 11 Exit Criteria and Final Mental Model

A learner can exit Module 01 when they can:

1. reconstruct an unfamiliar decoder from configuration and source;
2. preserve token, head, sequence, and feature axes through every operation;
3. detect future-token leakage and position-transform errors with invariants;
4. distinguish MHA, MQA, GQA, sparse experts, and latent attention;
5. derive scoped parameter and FLOP estimates and state every exclusion;
6. explain why logical work, active parameters, memory footprint, and latency are different quantities;
7. localize numerical or backend divergence before proposing a fix;
8. hand accurate architectural inputs to GPU, KV-cache, serving, optimization, and distributed-inference modules.

The final invariant: **model architecture is an executable tensor contract, not a bag of component names.**

## 12 Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

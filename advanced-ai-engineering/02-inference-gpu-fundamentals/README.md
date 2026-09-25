# Module 02 — Inference GPU Fundamentals

## 00 Why This Module Exists

An LLM forward pass is a graph of asynchronous host submissions, device kernels, memory transfers, allocations, and synchronization. A model-level FLOP count cannot tell you whether the live execution is limited by launch latency, exposed dependencies, insufficient parallelism, a compute pipeline, a memory boundary, resource residency, throttling, or an interaction among them.

This module builds the hardware/software measurement model needed before KV-cache engineering, serving, or optimization. It teaches the learner to define the measured boundary, calculate a scoped upper bound, collect timeline and kernel evidence, break one assumption at a time, and reject a clean bottleneck story when the telemetry does not discriminate it.

**Module orientation**

- **Engineering problem**: connect tensor shapes and operations to GPU execution, memory traffic, time, profiler observations, and falsifiable performance explanations.
- **What you will do**: implement reference kernels, induce coalescing and divergence failures, prove why unsynchronized timing is invalid, construct qualified Roofline bounds, profile at system and kernel scope, trace a pinned benchmark implementation, and defend a diagnosis against alternatives.
- **Research cutoff**: 2026-09-25. Current implementation claims are pinned to exact source revisions where source behavior is asserted.

## 01 Prerequisites and Scope

Prerequisites: Module 00 measurement discipline; Module 01 tensor shapes, matrix multiplication, parameter/FLOP conventions, and numerical invariants; basic Python and either CUDA C++ or a GPU kernel DSL.

This module owns:

- CUDA-style execution hierarchy, SIMT, warps, divergence, and synchronization;
- registers, shared memory, caches, device memory, memory transactions, and coalescing;
- asynchronous execution, streams, events, timing boundaries, warmup, and replication;
- arithmetic intensity, effective bandwidth, basic and hierarchical Roofline reasoning;
- occupancy/resource residency and why occupancy is not a performance objective;
- application-timeline versus kernel-counter profiling;
- profiler overhead, replay, cache control, and measurement intrusion;
- shape-, precision-, backend-, and hardware-dependent bottleneck transitions.

It cross-references but does not replace:

- KV-cache layout, paging, fragmentation, and eviction — Module 03;
- TTFT, ITL/TPOT, queueing, batching, goodput, and serving capacity — Module 04;
- FlashAttention, fusion, quantization, speculative decoding, and production kernel tuning — Module 05;
- collective communication and multi-GPU topology — Module 20;
- fleet observability, SLOs, and incident operations — Module 23.

## 02 Target Mastery

```yaml
module: 02-inference-gpu-fundamentals
mastery:
  understand:
    - execution hierarchy and memory hierarchy
    - asynchronous timing and synchronization boundaries
    - Roofline assumptions and resource-residency limits
  model:
    - work, bytes, time, throughput, intensity, and attainable ceilings
    - shape-dependent compute, memory, and latency regimes
  measure:
    - synchronized device and wall time
    - useful versus measured traffic and achieved throughput
    - timelines, launch geometry, occupancy limits, stalls, clocks, and variance
  implement:
    - explicit reference kernels with correctness and shape assertions
    - a reproducible benchmark and profiler manifest
  break:
    - coalescing, alignment, divergence, synchronization, occupancy, and numerical assumptions
    - profiler representativeness through replay-sensitive or concurrent workloads
  diagnose:
    - launch, latency, compute, memory, resource, throttling, and measurement hypotheses
  defend:
    - a scoped performance intervention with falsifying evidence and remeasurement
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Core Mental Model

```text
host code / framework
  ↓ asynchronous submission, allocation, dispatch, synchronization
CUDA streams and dependencies
  ↓ grids of thread blocks
SM residency and warp issue
  ↓ instructions, tensor/math pipelines, load/store pipelines
registers ↔ shared memory / L1 ↔ L2 ↔ device memory
  ↓
completed results observed at an explicit boundary
```

The diagnostic feedback loop is:

```text
shape / layout / precision / backend changes
  → kernel choice and launch geometry change
  → reuse, traffic, registers, shared memory, and parallelism change
  → stalls and attainable throughput change
  → elapsed time and downstream serving behavior change
```

That loop is a hypothesis template, not a universal mechanism. It applies only when the selected path actually changes those quantities. It is falsified when the kernel path and device evidence remain stable while latency changes elsewhere, for example in a CPU gap, synchronization, queue, or allocation.

Keep three knowledge types separate:

- **O — Source observation**: a profiler counter, timeline event, documented semantic, or pinned source behavior.
- **D — Derivation**: a result following from named units, formulas, and assumptions.
- **H — Engineering hypothesis**: a causal explanation that predicts what a discriminating intervention will change.

## 04 Lessons

### Lesson 2.1 — Execution Hierarchy, SIMT, and Resource Residency

**Engineering question:** What can the program control, and what is only observed after the device schedules work?

A kernel launch defines a grid of thread blocks. Blocks are scheduled onto streaming multiprocessors (SMs); correctness cannot depend on the order in which blocks run. Threads within an SM execute in groups of 32 called warps on current CUDA devices. Threads have individual state, but a warp is most efficient when active lanes follow the same instruction path.

Do not collapse these concepts:

- **grid size**: total blocks and exposed work;
- **block size**: threads that cooperate and share block-scoped resources;
- **warp execution**: active lanes, divergence, reconvergence, and instruction issue;
- **resident blocks/warps**: work that fits concurrently on an SM;
- **occupancy**: active warps divided by the architectural maximum;
- **utilization/throughput**: activity or work rate during a named interval.

Residency is jointly constrained by architectural block/thread limits and per-block registers and shared memory. More occupancy can help hide latency, but maximum occupancy is not a theorem of maximum performance. Reducing registers to increase occupancy can spill data; shrinking tiles can lower reuse; changing blocks can reduce instruction-level parallelism.

**Break test:** sweep block size and artificial register/shared-memory use. Record theoretical occupancy, achieved active warps, spills, stall mix, and kernel time. The intended falsification is a case where higher occupancy is slower.

---

### Lesson 2.2 — Memory Hierarchy, Transactions, and Coalescing

**Engineering question:** How many bytes did the program need, and how many bytes did the hardware move at each boundary?

A useful simplified path is registers and shared memory/L1, then L2, then device memory. The exact topology, capacities, cache behavior, and instructions vary by architecture. Never substitute an unqualified “GPU memory” byte count for all levels.

When lanes in a warp access adjacent aligned values, hardware can serve requests with fewer memory transactions. Strided, scattered, or misaligned patterns can transfer sectors containing unused data. Coalescing therefore concerns the ratio of requested bytes to transferred bytes, not merely whether addresses are contiguous somewhere in source code.

For useful bytes read $B_r$, useful bytes written $B_w$, and synchronized elapsed time $t$ seconds:

$$BW_{useful}=\frac{B_r+B_w}{t}\quad\text{bytes/s}.$$

Use $10^9$ for decimal GB/s and $2^{30}$ for GiB/s; do not mix the label and divisor. This value is not DRAM traffic. Compare it with profiler traffic at a named boundary to test transaction waste or reuse hypotheses.

**Necessary but insufficient signals**

- low useful bandwidth may mean poor coalescing, insufficient parallelism, dependency stalls, or simply little memory work;
- high DRAM throughput supports a DRAM-pressure hypothesis but does not prove every load is efficient;
- high L2 hit rate does not reveal whether L1/shared/register use is optimal;
- many transactions do not prove they are on the critical path.

---

### Lesson 2.3 — Asynchrony and Honest Timing

**Engineering question:** What exactly starts and stops the clock?

Kernel launches normally return before device completion. A host timer around a launch can therefore measure submission time, not execution time. Three common boundaries are different measurements:

1. **device interval**: events recorded in the relevant stream;
2. **synchronized host interval**: wall time with completion enforced at the end points;
3. **application interval**: end-to-end work including host logic, allocation, dispatch, transfers, and required synchronization.

Record stream, start/end events, synchronization call, warmup, repetitions, aggregation, and whether transfers or setup are included. An event on one stream does not automatically bound unrelated work in another stream.

Warmup can include context creation, library initialization, memory-pool growth, code generation, autotuning, cache state, and clock ramp. Do not erase cold-start behavior when cold start is the question; report cold and steady-state separately.

**Anti-patterns**

- `time.time()` around an asynchronous launch with no completion boundary;
- synchronizing after every operator and then claiming the result represents production overlap;
- selecting the minimum of many runs without documenting the intended estimator;
- timing under a heavy profiler and treating the host duration as uninstrumented latency.

---

### Lesson 2.4 — Roofline as a Qualified Bound

Define:

- $W$: executed or modeled work in operations;
- $Q$: bytes transferred at one named memory-hierarchy boundary;
- $I=W/Q$: operational/arithmetic intensity in operations per byte;
- $P_{peak}$: attainable compute ceiling for the selected precision/instruction path;
- $\beta$: attainable bandwidth at the same selected memory boundary.

The basic Roofline bound is

$$P\le\min(P_{peak},\beta I),$$

with ridge intensity

$$I^*=\frac{P_{peak}}{\beta}.$$

Equivalent time lower bounds are

$$T_{compute}\ge\frac{W}{P_{peak}},\qquad T_{memory}\ge\frac{Q}{\beta},$$

and under the basic overlap abstraction,

$$T\ge\max\left(\frac{W}{P_{peak}},\frac{Q}{\beta}\right).$$

This is an analytical bound, not a latency guarantee. State all assumptions:

- FMA counting convention and which operations are counted;
- precision and instruction path, including whether tensor instructions execute;
- DRAM, L2, L1, or another byte boundary;
- measured sustainable versus specification/nameplate ceiling;
- sufficient parallelism and problem size;
- cache/reuse model and whether bytes are algorithmic or measured;
- excluded launch, synchronization, allocation, dependency, and control costs.

A point below both ceilings is not automatically “neither compute nor memory.” It may reflect a lower unmodeled ceiling, poor instruction mix, dependencies, divergence, occupancy/resource limits, insufficient waves, frequency/power state, or invalid counts. Hierarchical Roofline and profiler evidence refine the hypothesis.

---

### Lesson 2.5 — Shape-Dependent Regime Transitions

For $C_{M\times N}=A_{M\times K}B_{K\times N}$ and FMA counted as two operations:

$$W_{GEMM}\approx2MNK.$$

If each element occupies $s$ bytes and an optimistic screen assumes one read of $A$, one read of $B$, one write of $C$, and $\beta C=0$:

$$I_{one-pass}=\frac{2MNK}{s(MK+KN+MN)}.$$

This is not measured intensity. Tiles reuse operands through registers/shared memory/cache, while imperfect implementations can reload them. The chosen $M,N,K$, layout, dtype, alignment, epilogue, library version, workspace, and concurrent work affect kernel selection and realized traffic.

Consequences:

- matrix-vector-like shapes often expose far less reuse than large matrix-matrix shapes;
- a larger batch can raise reuse and parallelism, then later hit another ceiling;
- padding can improve an instruction path while adding work;
- a nominally high-intensity kernel can remain latency-limited when the grid is too small;
- the same model phase can contain kernels in different regimes.

Therefore “prefill is compute-bound” and “decode is memory-bound” are useful hypotheses for some shapes, not architecture-independent facts. Module 04 measures their serving consequences; this module verifies the device path.

Numerical behavior remains part of the experiment. Floating-point addition is not generally associative, and fused or reordered reductions may differ. Define tolerances and downstream quality checks before declaring a faster path correct.

---

### Lesson 2.6 — Profiler Ladder and Diagnostic Reasoning

Use the least intrusive evidence that can answer the current question:

1. establish a synchronized unprofiled baseline with distributions;
2. use an application timeline to locate CPU gaps, launches, copies, streams, kernels, and synchronization;
3. select representative kernel instances, preserving shapes and context;
4. collect targeted kernel metrics for launch geometry, resource limits, achieved throughput, traffic, and stalls;
5. inspect source or SASS only when the hypothesis requires instruction-path evidence;
6. change one causal variable and remeasure the baseline.

Nsight Systems and Nsight Compute are complementary. Systems traces application scheduling and the CPU/GPU timeline. Compute collects detailed metrics for selected kernels. A kernel can be efficient while the application is slow; an application timeline can show a long kernel without explaining its internal limiter.

Profilers are interventions. Nsight Compute may replay kernels or ranges, save/restore memory, control caches/clocks, serialize launches, or patch instructions depending on the metric set. Keep replay mode, cache control, metric set, filtering, and concurrent activity in the evidence record. If the workload is nondeterministic or relies on concurrency, verify that the capture method preserves the behavior of interest.

Apply this chain:

```text
SYMPTOM
  → competing hypotheses
  → missing evidence
  → discriminating measurement or intervention
  → ranked explanation
  → change
  → unprofiled remeasurement
```

Example: “GPU utilization fell” can be explained by host launch gaps, smaller grids, dependency stalls, a faster kernel, throttling, memory faults, synchronization, or another process. Utilization alone cannot rank them.

---

### Lesson 2.7 — Pinned Runtime Source Trace

At PyTorch commit `7ee5406f6686d190efc6571475f074ba1bc9a8c0`, inspect:

```text
torch/utils/benchmark/utils/timer.py
  Timer.blocked_autorange
    → Timer._estimate_block_size
        → Timer._timeit
            → timer
                → torch.accelerator.synchronize
                → timeit.default_timer
    → repeated measurement loop
    → Measurement(number_per_run, raw_times, task_spec)
```

Observed behavior at that revision:

- the module's default timer synchronizes an available accelerator before reading the host timer;
- block-size estimation acts as warmup and attempts to amortize timer overhead;
- `blocked_autorange` collects repeated timed blocks and returns raw times with the repetition count.

Generalizable? **PARTIAL.** Synchronization, warmup, and replicates are general measurement concerns. The exact thresholds, call graph, and accelerator abstraction are PyTorch-revision-specific. This path does not measure an end-to-end serving request and its synchronization suppresses overlap outside the timed statement.

## 05 Literature and Source Map

**REFERENCE / BASELINE**

- Williams, Waterman, and Patterson (2009), *Roofline: An Insightful Visual Performance Model for Multicore Architectures* — original bound and optimization model.
- NVIDIA, *CUDA Programming Guide* — current execution, synchronization, and architecture semantics.
- NVIDIA, *CUDA C++ Best Practices Guide* — correctness, timing, bandwidth, coalescing, and execution-configuration guidance.

**CURRENT OPERATIONAL DOCUMENTATION**

- NVIDIA, *Nsight Systems User Guide* — CPU/GPU application timelines and CUDA tracing.
- NVIDIA, *Nsight Compute Profiling Guide* — metrics, replay, overhead, reproducibility, and Roofline analysis.
- NVIDIA, *GPU Performance Background* and *Matrix Multiplication Background* — shape-dependent math/memory/latency intuition with explicitly historical hardware examples.
- PyTorch, *CUDA semantics* — framework-facing asynchronous execution and timing boundaries.

**CURRENT SOURCE SNAPSHOT**

- PyTorch commit `7ee5406f6686d190efc6571475f074ba1bc9a8c0`, `torch/utils/benchmark/utils/timer.py`, symbols `timer`, `Timer._estimate_block_size`, and `Timer.blocked_autorange`, verified 2026-09-25.

**Currentness classification**

- **REFERENCE / BASELINE**: SIMT reasoning, explicit synchronization, effective bandwidth, and basic Roofline.
- **CURRENT DEFAULT PRACTICE**: synchronized/warmed replicated benchmarks followed by timeline-first and targeted-kernel profiling; this is a workflow, not one tool mandate.
- **WORKLOAD-DEPENDENT**: compute-, memory-, latency-, launch-, occupancy-, or dependency-limited behavior; useful batch/shape; overlap; cache benefit.
- **FRONTIER / GENERATION-SPECIFIC**: new tensor instructions, asynchronous copy/memory engines, cluster-level features, graph/device launch, and hierarchy-specific optimizations.
- **LEGACY WHEN UNIVERSALIZED**: fixed transaction rules from old compute capabilities, implicit warp-synchronous assumptions, and a single device's nameplate ridge used for every kernel.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

### LAB A — Transactions, Layout, and Useful Bandwidth

- **Build**: implement copy/add and a tiled matrix multiply with shape and bounds assertions.
- **Measure**: sweep contiguous, offset, and strided accesses; calculate useful bytes/time and collect targeted L1/L2/DRAM traffic.
- **Break**: preserve element count and arithmetic while permuting layout so adjacent lanes access a large stride.
- **Competing hypotheses**: transaction waste, cache reuse, insufficient grid size, alignment, compiler vectorization, or timing error.
- **Falsification**: if measured transaction/traffic efficiency and kernel time do not respond to layout while the same path executes, the coalescing explanation weakens.
- **Artifact**: source, correctness oracle, hardware/software manifest, byte equations, raw timings, targeted profile, and a boundary-labeled conclusion.

### LAB B — Asynchronous Timing Trap

- **Build**: time identical GPU work with an unsynchronized host timer, synchronized host timer, device events, and `torch.utils.benchmark`.
- **Break**: add work on a second stream and move synchronization boundaries.
- **Measure**: submission time, device interval, application interval, warm/cold distributions, and timeline.
- **Falsification**: demonstrate which timer excludes queued work and which synchronization destroys overlap.
- **Artifact**: a timing contract that makes start/end, streams, setup, and aggregation explicit.

### LAB C — Qualified Roofline Matrix

- **Build**: create elementwise, reduction, GEMV-like, and GEMM-like cases across shapes and dtypes.
- **Derive**: $W$, algorithmic $Q$, $I$, attainable ceilings, ridge, and time lower bounds with units.
- **Measure**: selected instruction path, executed work where available, traffic at multiple hierarchy levels, throughput, grid size, and duration.
- **Break**: choose a tiny high-intensity problem that underfills the GPU and a large low-intensity problem with high bandwidth.
- **Falsification**: revise any classification whose selected precision/path or traffic boundary does not match the model.
- **Artifact**: analytical-versus-measured table with every exclusion and no universal model-phase label.

### LAB D — Competing Bottleneck Diagnosis

- **Inject separately**: host launch gaps, divergence, strided loads, register pressure, synchronization, small grids, background GPU activity, and a constrained power/clock state where safe.
- **Blind diagnose**: start from the same symptom—higher elapsed time or lower aggregate utilization.
- **Required evidence**: unprofiled baseline, Systems timeline, selected Compute metrics, clocks/power, shape/backend, and controlled intervention.
- **Scoring rule**: no credit for naming a bottleneck without ruling down at least two alternatives.
- **Artifact**: symptom → hypotheses → missing evidence → discrimination → ranked cause → intervention → remeasurement.

### LAB E — Pinned Benchmark Source Trace

- **Trace**: PyTorch `Timer.blocked_autorange` at the pinned commit through block-size estimation and synchronization.
- **Compare**: direct events, manual synchronized wall timing, and the utility on one stable operator.
- **Break**: add asynchronous work outside the timed statement and explain why each boundary reports a different result.
- **Artifact**: repository, commit, verification date, file, symbols, entry point, execution path, exact run commands, and `TODO_VERIFY` for any path not executed locally.

## 07 Break / Incident Scenario

### Incident 02.1 — Latency Doubled While “GPU Utilization” Fell

After a framework upgrade, a fixed-shape inference benchmark is twice as slow and a dashboard GPU-utilization average is lower. A teammate concludes that memory bandwidth is the bottleneck and proposes kernel fusion.

Competing hypotheses:

1. the selected kernel/backend changed;
2. host compilation or launch gaps entered the measured window;
3. synchronization or copies became serialized;
4. the grid underfills the GPU after a shape/padding change;
5. register/shared-memory changes reduced residency or caused spills;
6. memory layout increased transferred bytes;
7. clocks/power/thermal state changed;
8. the benchmark boundary or warmup changed;
9. another process or profiler perturbed execution.

Required response:

- freeze inputs, shape, dtype, seed, correctness tolerance, and software/hardware manifest;
- reproduce with synchronized unprofiled distributions;
- align old/new Systems timelines and identify changed gaps, kernels, copies, and sync;
- profile only representative changed kernels with the smallest discriminating metric set;
- verify source/backend selection;
- run one-variable interventions;
- remeasure unprofiled latency and numerical parity.

The aggregate utilization signal is useful for detecting change but insufficient to establish cause.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

The learner receives an unfamiliar GPU trace, selected kernel profiles, tensor shapes, and a regression report. They must:

1. define all timing and byte boundaries;
2. reconstruct launch geometry and resource constraints;
3. derive a unit-consistent Roofline screen with stated exclusions;
4. identify at least three competing explanations;
5. request the minimum discriminating measurements;
6. reproduce one failure in a controlled kernel;
7. propose an intervention with a falsifying prediction;
8. demonstrate numerical parity and unprofiled improvement;
9. state which conclusions are O, D, and H;
10. refuse unsupported translation from kernel behavior to TTFT/TPOT or capacity.

## 09 Required Evidence and Rubric

| Dimension | Evidence required | Failure condition |
|---|---|---|
| Correctness | oracle, shape assertions, dtype/tolerance, numerical-difference distribution | speed reported without parity |
| Measurement | clocks, boundaries, warmup, repetitions, raw samples, sync/stream semantics | asynchronous submission timed as execution |
| Modeling | work/byte equations, units, precision/path, hierarchy boundary, ceilings | Roofline assembled from mismatched quantities |
| Profiling | unprofiled baseline, application timeline, targeted kernel metrics, capture settings | counter dump without a question |
| Diagnosis | alternatives, discriminating tests, ranked explanation, falsifier | exactly-one bottleneck from utilization |
| Reproducibility | hardware, compute capability, driver/toolkit/framework, source revision, commands | environment cannot be reconstructed |
| Remeasurement | same representative workload after intervention, including variance | profiled improvement only |

## 10 Capability Traceability Matrix

| Capability | Lessons | Lab / assessment | Evidence claims |
|---|---|---|---|
| Explain GPU execution and residency | 2.1 | LAB A, LAB D | M02-CLM-001, 007 |
| Diagnose memory transactions | 2.2 | LAB A | M02-CLM-002, 010 |
| Time asynchronous work correctly | 2.3, 2.7 | LAB B, LAB E | M02-CLM-003, 011 |
| Build and qualify a Roofline model | 2.4, 2.5 | LAB C | M02-CLM-004, 005, 006 |
| Select profiler scope and control intrusion | 2.6 | LAB D | M02-CLM-008, 009, 012 |
| Preserve numerical validity | 2.5, 2.6 | LAB C, LAB D | M02-CLM-013 |

## 11 Exit Criteria and Final Mental Model

A learner can exit Module 02 when they can:

1. connect tensor shapes to grids, blocks, warps, resource residency, and memory traffic;
2. distinguish useful bytes from traffic at named hierarchy levels;
3. time asynchronous work with an explicit, defensible boundary;
4. derive Roofline bounds without presenting them as latency guarantees;
5. explain why occupancy and aggregate utilization are not unique bottleneck classifiers;
6. show a workload that transitions among latency, memory, and compute regimes;
7. use timeline and kernel profilers in a hypothesis-driven sequence;
8. account for profiler replay and measurement intrusion;
9. preserve correctness and numerical tolerance during performance experiments;
10. hand Module 04 measured device evidence without confusing it with serving metrics.

The final invariant: **a GPU bottleneck is a scoped causal claim supported by aligned work, byte, time, timeline, and intervention evidence—not a label read from one counter.**

## 12 Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

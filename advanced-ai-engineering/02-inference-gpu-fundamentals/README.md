# Module 02 — Inference GPU Fundamentals

## 00 Why This Module Exists

An LLM forward pass is a graph of asynchronous host submissions, device kernels, memory transfers, allocations, and synchronization. A model-level FLOP count cannot tell you whether the live execution is limited by launch latency, exposed dependencies, insufficient parallelism, a compute pipeline, a memory boundary, resource residency, throttling, or an interaction among them.

This module builds the hardware/software measurement model needed before KV-cache engineering, serving, or optimization. It teaches the learner to define the measured boundary, calculate a scoped upper bound, collect timeline and kernel evidence, break one assumption at a time, and reject a clean bottleneck story when the telemetry does not discriminate it.

**Module Orientation**

- **Engineering Problem**: Connect tensor shapes and operations to GPU execution, memory traffic, time, profiler observations, and falsifiable performance explanations.
- **What You Will Do**: Implement reference kernels, induce coalescing and divergence failures, prove why unsynchronized timing is invalid, construct qualified Roofline bounds, profile at system and kernel scope, trace a pinned benchmark implementation, and defend a diagnosis against alternatives.
- **Environment**: Python 3.10+ plus CUDA C++ or a GPU kernel DSL; an NVIDIA GPU and current profiling tools are required for device-counter labs, while analytical exercises can run without them.
- **Research Cutoff**: 2026-09-27. Current implementation claims remain pinned to the verified revisions below.

## 01 Baseline Assumptions

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
  security: NOT_APPLICABLE
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 6h
  guided_practice: 3h
  labs: 15h
  assessment: 3h
  source_trace: 2h
  total: 29h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

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

**Engineering Question:** What can the program control, and what is only observed after the device schedules work?

**Concepts & Definitions:**

A kernel launch defines a grid of thread blocks. Blocks are scheduled onto streaming multiprocessors (SMs); correctness cannot depend on the order in which blocks run. Threads within an SM execute in groups of 32 called warps on current CUDA devices. Threads have individual state, but a warp is most efficient when active lanes follow the same instruction path.

Do not collapse these concepts:

- **grid size**: total blocks and exposed work;
- **block size**: threads that cooperate and share block-scoped resources;
- **warp execution**: active lanes, divergence, reconvergence, and instruction issue;
- **resident blocks/warps**: work that fits concurrently on an SM;
- **occupancy**: active warps divided by the architectural maximum;
- **utilization/throughput**: activity or work rate during a named interval.

Residency is jointly constrained by architectural block/thread limits and per-block registers and shared memory. More occupancy can help hide latency, but maximum occupancy is not a theorem of maximum performance. Reducing registers to increase occupancy can spill data; shrinking tiles can lower reuse; changing blocks can reduce instruction-level parallelism.

**Worked Example:** Two kernels may launch the same grid while different register or shared-memory footprints permit different resident blocks per SM; occupancy alone does not predict which finishes first.

**Knowledge Check:** Which launch quantities are chosen by the program, and which residency/utilization quantities must be observed?

**Guided Practice:** Sweep block size and artificial register/shared-memory use. Record theoretical occupancy, achieved active warps, spills, stall mix, and kernel time. Seek a case where higher occupancy is slower.

**Feedback Contract:** Require launch geometry, resource limits, achieved activity, time, and a falsifier; reject “maximize occupancy” as an objective without workload evidence.

**Learning Outcome:** Connect execution hierarchy and resource residency to measured performance without treating occupancy as a universal target.

*(Effort: 45m instruction, 25m practice)*

---

### Lesson 2.2 — Memory Hierarchy, Transactions, and Coalescing

**Engineering Question:** How many bytes did the program need, and how many bytes did the hardware move at each boundary?

**Concepts & Definitions:**

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

**Worked Example:** A kernel requesting 4 MiB and taking 1 ms has 4 GiB/s useful bandwidth under binary units; profiler-reported DRAM bytes may be larger or smaller depending on reuse and the named boundary.

**Knowledge Check:** Why are useful bytes, L2 traffic, and DRAM traffic different, and what does coalescing change?

**Guided Practice:** Sweep aligned contiguous, offset, and strided accesses while preserving arithmetic; compare useful bandwidth with hierarchy-specific traffic.

**Feedback Contract:** Require byte equations, units, boundary names, timing synchronization, transaction evidence, and competing explanations.

**Learning Outcome:** Distinguish requested work from measured traffic and diagnose layout-dependent transaction waste.

*(Effort: 45m instruction, 25m practice)*

---

### Lesson 2.3 — Asynchrony and Honest Timing

**Engineering Question:** What exactly starts and stops the clock?

**Concepts & Definitions:**

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

**Worked Example:** A host launch returning in 20 microseconds while a synchronized boundary completes at 200 microseconds shows submission time and completed-work time, not conflicting measurements.

**Knowledge Check:** Why can an event on one stream fail to bound work on another, and when is per-operator synchronization unrepresentative?

**Guided Practice:** Time identical work using unsynchronized wall time, synchronized wall time, device events, and an application boundary; then add a second stream.

**Feedback Contract:** Report clocks, streams, synchronization, warmup, repetitions, inclusion rules, raw samples, and profiler status.

**Learning Outcome:** Measure asynchronous execution at an explicit boundary without destroying the concurrency being studied.

*(Effort: 45m instruction, 25m practice)*

---

### Lesson 2.4 — Roofline as a Qualified Bound

**Engineering Question:** Under which work, byte, hierarchy, precision, and ceiling assumptions does a Roofline bound discriminate a regime?

**Concepts & Definitions:**
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

**Worked Example:** For declared $P_{peak}$ and $\beta$, compute $I^*=P_{peak}/\beta$, then compare a modeled intensity with that ridge. The result is a screening bound, not a latency guarantee.

**Knowledge Check:** Which byte boundary defines $I$, and why can a point below both ceilings have an unmodeled limiter?

**Guided Practice:** Derive bounds using specification peaks and then measured sustainable ceilings; explain how the conclusion changes.

**Feedback Contract:** Require units, FMA convention, selected instruction path, hierarchy boundary, attainable ceilings, exclusions, and profiler validation.

**Learning Outcome:** Build and qualify a Roofline model without promoting it to a measured bottleneck or latency guarantee.

*(Effort: 55m instruction, 30m practice)*

---

### Lesson 2.5 — Shape-Dependent Regime Transitions

**Engineering Question:** Which shape and backend changes move execution among launch-, latency-, bandwidth-, and compute-limited regimes?

**Concepts & Definitions:**
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

**Worked Example:** A tiny high-intensity GEMM can underfill the GPU, whereas a larger shape can expose enough parallelism to approach another ceiling; arithmetic intensity alone does not encode grid size.

**Knowledge Check:** Why is the one-pass GEMM intensity not measured intensity, and how can padding both add work and improve execution?

**Guided Practice:** Sweep GEMV-like and GEMM-like shapes, dtype, alignment, and backend; record selected kernels, traffic, grid size, time, and numerical error.

**Feedback Contract:** Identify transitions from aligned evidence rather than assigning one universal regime to prefill or decode.

**Learning Outcome:** Explain and measure shape-dependent bottleneck transitions while preserving numerical validity.

*(Effort: 50m instruction, 30m practice)*

---

### Lesson 2.6 — Profiler Ladder and Diagnostic Reasoning

**Engineering Question:** What is the least intrusive evidence needed to distinguish competing GPU-performance explanations?

**Concepts & Definitions:**
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

**Worked Example:** A Systems trace showing a CPU gap before an unchanged kernel weakens an internal-kernel bottleneck claim; targeted kernel counters are unnecessary until the changed interval is localized.

**Knowledge Check:** When should a timeline precede kernel metrics, and how can replay invalidate a concurrency-sensitive capture?

**Guided Practice:** Diagnose the same latency symptom after separately injecting a host gap, strided access, and register pressure.

**Feedback Contract:** Require an unprofiled baseline, minimal discriminating capture, capture settings, alternatives, intervention, and unprofiled remeasurement.

**Learning Outcome:** Select profiler scope from a hypothesis and account for the profiler as an intervention.

*(Effort: 50m instruction, 30m practice)*

---

### Lesson 2.7 — Pinned Runtime Source Trace

**Engineering Question:** What timing semantics does the pinned benchmark utility implement, and what does it exclude?

**Concepts & Definitions:** A source trace distinguishes implementation-specific synchronization, warmup, repetition, aggregation, and returned raw measurements from general benchmarking principles.

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

**Worked Example:** `blocked_autorange` returning raw block times and repetitions supports operator timing at this boundary; it does not include request queueing or prove production overlap.

**Knowledge Check:** Where does synchronization occur, and which surrounding asynchronous work falls outside the statement?

**Independent Practice:** Execute the pinned trace on one operator, compare it with device events and manual synchronized wall time, and mark unexecuted paths `TODO_VERIFY`.

**Feedback Contract:** Require revision, path, symbols, call path, configuration, static-versus-executed status, exact commands, and generalizability limits.

**Learning Outcome:** Verify benchmark timing behavior from source without turning utility-specific semantics into a universal definition.

*(Effort: 35m instruction, 30m source trace)*

## 05 Literature & Production Source Map

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

All labs follow $\text{PREDICT}\to\text{BUILD}\to\text{MEASURE}\to\text{EXPLAIN}\to\text{BREAK}\to\text{IMPROVE}\to\text{FALSIFY}$.

### LAB A — Transactions, Layout, and Useful Bandwidth

- **Objective**: Determine when layout changes alter useful and physical traffic at named memory boundaries.
- **Pre-Registered Hypothesis**: Strided lane access will increase transferred traffic or reduce useful bandwidth on the selected path relative to aligned contiguous access.
- **Independent Variables**: Layout, offset, stride, problem size, and launch geometry.
- **Dependent Variables**: Correctness, useful bandwidth, hierarchy traffic, transactions, stalls, and kernel time.
- **Build**: implement copy/add and a tiled matrix multiply with shape and bounds assertions.
- **Measure**: sweep contiguous, offset, and strided accesses; calculate useful bytes/time and collect targeted L1/L2/DRAM traffic.
- **Break & Falsify**: Preserve element count and arithmetic while permuting layout so adjacent lanes access a large stride; if traffic efficiency and time do not respond on the same path, weaken the coalescing explanation.
- **Competing hypotheses**: transaction waste, cache reuse, insufficient grid size, alignment, compiler vectorization, or timing error.
- **Artifact**: source, correctness oracle, hardware/software manifest, byte equations, raw timings, targeted profile, and a boundary-labeled conclusion.
- **Alignment**: Lessons 2.1–2.2.
- **Effort Estimate**: 3h implementation, 1h analysis.

### LAB B — Asynchronous Timing Trap

- **Objective**: Establish which timing boundaries include queued device work and which perturb production overlap.
- **Pre-Registered Hypothesis**: Unsynchronized host timing will measure submission rather than completion for asynchronous work.
- **Independent Variables**: Timer, synchronization boundary, stream count, cold/warm state, and workload size.
- **Dependent Variables**: Submission, device, synchronized-wall, and application durations plus overlap visible in the timeline.
- **Build**: time identical GPU work with an unsynchronized host timer, synchronized host timer, device events, and `torch.utils.benchmark`.
- **Break & Falsify**: Add work on a second stream, move synchronization boundaries, and demonstrate which timer excludes queued work or destroys overlap.
- **Measure**: submission time, device interval, application interval, warm/cold distributions, and timeline.
- **Artifact**: a timing contract that makes start/end, streams, setup, and aggregation explicit.
- **Alignment**: Lesson 2.3.
- **Effort Estimate**: 2h implementation, 1h analysis.

### LAB C — Qualified Roofline Matrix

- **Objective**: Compare analytical bounds with measured execution across shape and arithmetic-intensity regimes.
- **Pre-Registered Hypothesis**: Regime classification will change for at least one shape when measured traffic and sustainable ceilings replace optimistic one-pass/specification values.
- **Independent Variables**: Operation family, shape, dtype, layout, backend, and hierarchy boundary.
- **Dependent Variables**: Work, traffic, intensity, selected path, grid size, throughput, duration, and numerical error.
- **Build**: create elementwise, reduction, GEMV-like, and GEMM-like cases across shapes and dtypes.
- **Derive**: $W$, algorithmic $Q$, $I$, attainable ceilings, ridge, and time lower bounds with units.
- **Measure**: selected instruction path, executed work where available, traffic at multiple hierarchy levels, throughput, grid size, and duration.
- **Break & Falsify**: Choose a tiny high-intensity problem that underfills the GPU and a large low-intensity problem with high bandwidth; revise classifications whose precision/path or traffic boundary does not match.
- **Artifact**: analytical-versus-measured table with every exclusion and no universal model-phase label.
- **Alignment**: Lessons 2.4–2.5.
- **Effort Estimate**: 3h implementation, 1h analysis.

### LAB D — Competing Bottleneck Diagnosis

- **Objective**: Distinguish multiple causes that present as slower elapsed time or lower aggregate utilization.
- **Pre-Registered Hypothesis**: Timeline, targeted counters, and a one-variable intervention can rule down at least two alternatives for each injected fault.
- **Independent Variables**: Injected fault and evidence available to the blinded diagnostician.
- **Dependent Variables**: Diagnosis rank, evidence requested, time-to-discrimination, correction effect, and false attribution.
- **Inject separately**: host launch gaps, divergence, strided loads, register pressure, synchronization, small grids, background GPU activity, and a constrained power/clock state where safe.
- **Blind diagnose**: start from the same symptom—higher elapsed time or lower aggregate utilization.
- **Required evidence**: unprofiled baseline, Systems timeline, selected Compute metrics, clocks/power, shape/backend, and controlled intervention.
- **Scoring rule**: no credit for naming a bottleneck without ruling down at least two alternatives.
- **Break & Falsify**: Include two faults with similar aggregate utilization and reject any diagnosis that cannot distinguish them through aligned evidence and intervention.
- **Artifact**: symptom → hypotheses → missing evidence → discrimination → ranked cause → intervention → remeasurement.
- **Alignment**: Lessons 2.1, 2.2, 2.5, and 2.6.
- **Effort Estimate**: 3h injection and diagnosis, 1h report.

### LAB E — Pinned Benchmark Source Trace

- **Objective**: Verify timing and synchronization semantics of the pinned benchmark utility.
- **Pre-Registered Hypothesis**: Different timing boundaries will report different values when asynchronous work exists outside the timed statement.
- **Independent Variables**: Timing method, external asynchronous work, block size, and warmup state.
- **Dependent Variables**: Raw times, repetitions, synchronization placement, and disagreement among timing boundaries.
- **Trace**: PyTorch `Timer.blocked_autorange` at the pinned commit through block-size estimation and synchronization.
- **Compare**: direct events, manual synchronized wall timing, and the utility on one stable operator.
- **Break & Falsify**: Add asynchronous work outside the timed statement and explain why each boundary reports a different result.
- **Artifact**: repository, commit, verification date, file, symbols, entry point, execution path, exact run commands, and `TODO_VERIFY` for any path not executed locally.
- **Alignment**: Lessons 2.3 and 2.7.
- **Effort Estimate**: 2h source trace, 1h comparison.

## 07 Break / Incident Scenarios

### Incident 02.1: Latency Doubled While “GPU Utilization” Fell

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

**Diagnostic Protocol (Task):**

Required response:

- freeze inputs, shape, dtype, seed, correctness tolerance, and software/hardware manifest;
- reproduce with synchronized unprofiled distributions;
- align old/new Systems timelines and identify changed gaps, kernels, copies, and sync;
- profile only representative changed kernels with the smallest discriminating metric set;
- verify source/backend selection;
- run one-variable interventions;
- remeasure unprofiled latency and numerical parity.
- separate immediate rollback from the long-term correction and define quantitative recovery criteria.

The aggregate utilization signal is useful for detecting change but insufficient to establish cause.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Transfer Problem — Defend a GPU Performance Diagnosis

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

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit the pinned PyTorch benchmark trace from Lesson 2.7/Lab E, including repository, revision, verification date, path, symbols, call path, runtime configuration, static-versus-executed status, commands, and `TODO_VERIFY` markers.

### Rubric Dimensions

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

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Explain GPU execution and residency | Lesson 2.1 | LAB A / LAB D | Incident / Mastery | M02-CLM-001, 007; launch/resource trace |
| Diagnose memory transactions | Lesson 2.2 | LAB A | Mastery | M02-CLM-002, 010; traffic report |
| Time asynchronous work correctly | Lessons 2.3, 2.7 | LAB B / LAB E | Mastery | M02-CLM-003, 011; timing/source trace |
| Build and qualify a Roofline model | Lessons 2.4–2.5 | LAB C | Mastery | M02-CLM-004–006; bound/measurement table |
| Select profiler scope and control intrusion | Lesson 2.6 | LAB D | Incident | M02-CLM-008, 009, 012; diagnosis matrix |
| Preserve numerical validity | Lessons 2.5–2.6 | LAB C / LAB D | Mastery | M02-CLM-013; parity report |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

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

### Module Wrap-Up (Final Mental Model Reconstruction)

The final invariant: **a GPU bottleneck is a scoped causal claim supported by aligned work, byte, time, timeline, and intervention evidence—not a label read from one counter.**

## 12 Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Analyze -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

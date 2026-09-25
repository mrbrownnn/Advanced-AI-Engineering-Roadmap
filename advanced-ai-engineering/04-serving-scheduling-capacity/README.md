# Module 04 — Serving, Scheduling, & Capacity

## 00 Why This Module Exists
An LLM model weight checkpoint cannot serve user traffic on its own. Deploying a model into production requires a serving engine that manages request ingress, batches heterogeneous requests, schedules GPU compute across conflicting prefill and decode execution phases, and dynamically arbitrates constrained GPU High Bandwidth Memory (HBM).

When traffic surges or prompt distributions shift, static batching can waste capacity, long prefill work can delay active decodes, and KV growth can force queuing, rejection, preemption, migration, or recomputation. Which mechanism dominates is workload- and runtime-dependent and must be established from telemetry.

Mastering serving, scheduling, and capacity planning is what transforms an engineer from someone who merely runs inference scripts into a systems engineer capable of designing, provisioning, and defending production-grade LLM runtimes under rigorous Service Level Objectives (SLOs) and real-world economics.

**Module Orientation**
- **Engineering Problem**: Maximizing inference goodput and capacity utilization while enforcing strict bounds on Time-To-First-Token (TTFT) and Inter-Token Latency (ITL) under dynamic, multi-tenant workloads.
- **What You Will Do**: Implement a discrete-event continuous batching scheduler simulator, evaluate chunked prefill mechanics under heterogeneous prompt distributions, model queueing saturation knees using Little's Law and M/G/1 theory, diagnose a multi-factor production latency incident, trace vLLM scheduler execution paths, and design an enterprise serving platform capacity plan.
- **Environment**: A standard Python 3.10+ development environment for the scheduler simulator and queueing test harnesses (Labs A–D). Access to a single NVIDIA GPU (Ampere/Ada/Hopper) is optional but valuable for running live inference benchmarks.

## 01 Baseline Assumptions
- **Transformer Forward Pass & Dimensions**: You understand the sequence dimensions, self-attention tensor shapes, and linear projection FLOP counts for standard autoregressive decoders (Module 01).
- **GPU Execution Model & Memory Hierarchy**: You understand Streaming Multiprocessors (SMs), Tensor Cores, High Bandwidth Memory (HBM) bandwidth constraints, and the distinction between compute-bound and memory-bandwidth-bound regimes via the Roofline model (Module 02).
- **KV Cache Memory Footprint & Paged Management**: You know how to derive the logical K/V payload per token ($2 \times \text{layers} \times \text{kv\_heads} \times \text{head\_dim} \times \text{bytes per element}$), state what that estimate excludes, and understand how logical sequences map to physical blocks in paged KV memory pools (Module 03).

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
  research_connection: SELECTIVE

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 12h
  assessment: 3h
  source_trace: 2h
  total: 25h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map
Request Arrival $\to$ Ingress Control $\to$ Admission / Waiting Queue $\to$ Iteration-Level Scheduler $\to$ Prefill and Decode Work $\to$ GPU Compute, Memory-Bandwidth, and KV-Capacity Constraints $\to$ Optional Preemption / Migration / Recomputation $\to$ Output Streaming $\to$ Completion or Cancellation $\to$ Telemetry Feedback. Queueing, rate limiting, backpressure, load shedding, and capacity planning act at different points in this lifecycle; their effectiveness depends on workload, runtime, hardware, and client behavior.

## 04 Lessons

### Lesson 4.1 — The Serving Request Lifecycle & Metrics Decomposition

**Engineering Question:**
How do we define client-visible latency components precisely enough to narrow competing explanations for an SLO violation?

**Concepts & Definitions:**
- **Request Lifecycle**: An incoming request can pass through client/network transit, gateway processing, tokenization and validation, admission, scheduler waiting, prompt processing, first-token sampling and emission, autoregressive decode, output buffering/streaming, and completion or cancellation. Implementations may overlap or reorder some stages, so timestamps must be defined at explicit boundaries.
- **Time-to-First-Token (TTFT)**: The elapsed wall-clock duration between a declared request-arrival boundary and receipt of the first output event. Client-observed TTFT can include ingress and egress transit, gateway and host work, scheduler queueing, prompt processing, sampling, serialization, and buffering; engine-reported TTFT may use narrower boundaries.
- **Inter-Token Latency (ITL)**: The elapsed duration between consecutive output events at a declared observation boundary. Client-observed ITL can include scheduling, execution, sampling, buffering, and transport; an engine-side token timestamp uses a narrower definition. In a streaming interface, the upper tail affects perceived generation smoothness.
- **Time-Per-Output-Token (TPOT)**: The average decode latency across all generated tokens for a request: $(T_{e2e} - T_{TTFT}) / (N_{out} - 1)$. While TPOT represents the mean, ITL tracks the per-token distribution, capturing tail jitter and stalls.
- **End-to-End Latency ($T_{e2e}$)**: Total wall-clock time from initial request arrival to final token emission.

**Quantitative Model / Derivation:**
For a request arriving at time $t_0$, scheduled for prefill at $t_1$, emitting token 1 at $t_2$, and emitting subsequent tokens at $t_3, t_4, \dots, t_{N_{out}}$:
$$T_{queue} = t_1 - t_0$$
$$T_{first\_output\_after\_queue} = t_2 - t_1$$
$$T_{TTFT} = t_2 - t_0 = T_{queue} + T_{first\_output\_after\_queue}$$
Let $o_i$ be the client-observed arrival time of output token or output event $i$, with $o_1=t_2$:
$$ITL_i = o_i-o_{i-1} \quad \text{for } i \in [2,N_{out}]$$
$$T_{e2e} = o_{N_{out}}-t_0 = T_{TTFT}+\sum_{i=2}^{N_{out}}ITL_i$$
$$TPOT = \frac{T_{e2e}-T_{TTFT}}{N_{out}-1}=\frac{1}{N_{out}-1}\sum_{i=2}^{N_{out}}ITL_i$$
These identities assume one observed event per token. If a runtime emits multiple tokens per event, event-level ITL and token-amortized TPOT differ.

*Assumptions*: $t_0$, $t_1$, and the output timestamps share a clock and explicitly defined observation boundary; generation produces $N_{out} \ge 2$ output events. A narrower engine-side decomposition must separately account for host, sampling, serialization, buffering, and network time before comparing it with client-observed TTFT.

**Worked Example:**
A client issues a request with a 2,048-token prompt.
- Queue wait time: $T_{queue} = 120\text{ ms}$.
- Measured first-output work after queue admission: $T_{first\_output\_after\_queue} = 65\text{ ms}$.
- Output length: $N_{out} = 200\text{ tokens}$.
- Total autoregressive decode iterations take $4,975\text{ ms}$.
- $T_{TTFT} = 120\text{ ms} + 65\text{ ms} = 185\text{ ms}$.
- $TPOT = 4,975\text{ ms} / (200 - 1) = 25.0\text{ ms/token}$.
- $T_{e2e} = 185\text{ ms} + 4,975\text{ ms} = 5,160\text{ ms}$.
If the target SLO requires $T_{TTFT} < 100\text{ ms}$, the system is violating the SLO. The breakdown demonstrates that $T_{queue}$ alone ($120\text{ ms}$) exceeds the entire $100\text{ ms}$ budget before compute ever begins. Optimizing the prefill kernel will not restore compliance; the queue delay must be resolved.

**Knowledge Check:**
1. A monitoring dashboard reports that mean $T_{e2e}$ increased by $400\text{ ms}$, but mean $TPOT$ remained completely unchanged at $20\text{ ms/token}$. Assuming output token count is constant, which metric component accounts for the entire regression?
2. Why is tracking P99 ITL critical for conversational chat applications even if mean TPOT meets the 30 ms target?

**Guided Practice:**
An inference node serves a steady stream of requests with $N_{in} = 1,024$ and $N_{out} = 128$.
Under load, telemetry reveals:
- $T_{queue} = 240\text{ ms}$
- $T_{prefill} = 40\text{ ms}$
- 126 decode steps take $22\text{ ms}$ each.
- Step 45 experiences an anomalous pause of $410\text{ ms}$ before subsequent steps resume at $22\text{ ms}$.
Assume an output contains 128 tokens: 126 inter-token gaps are $22\text{ ms}$ and one gap is $410\text{ ms}$. Calculate: (a) $T_{TTFT}$, (b) mean $TPOT$, (c) maximum ITL, and (d) $T_{e2e}$. Then calculate empirical P99 under a stated quantile convention rather than silently equating P99 with the maximum.

**Feedback Contract:**
- *Expected Evidence*: (a) $T_{TTFT}=280\text{ ms}$. (b) Total post-first-output time $=(126\times22)+410=3,182\text{ ms}$ and mean $TPOT=3,182/127\approx25.06\text{ ms/token}$. (c) Maximum ITL is $410\text{ ms}$. With only 127 gaps, common empirical P99 conventions need not return the maximum; the convention and sample size must be reported. (d) $T_{e2e}=3,462\text{ ms}$. The post-first-output anomaly leaves TTFT unchanged but affects TPOT and the upper tail/max of ITL.
- *Common Failure*: Blaming the 410 ms spike on prefill queueing or adding 410 ms to TTFT.
- *Diagnostic Hint*: At what step index did the 410 ms pause occur? Was token 1 already emitted?
- *Concept to Revisit*: Latency Decomposition Invariants.

**Learning Outcome:**
Decompose end-to-end serving latency into mathematically precise lifecycle stages and attribute SLO breaches to queueing, prefill compute, or decode jitter.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 4.2 — Static vs. Continuous Batching Mechanics

**Engineering Question:**
When does fixed batch membership waste execution slots in autoregressive workloads, and what scheduler mechanisms make iteration-level admission possible?

**Concepts & Definitions:**
- **Static Batching**: Requests are grouped into a fixed batch of size $B$ at arrival. The entire batch executes synchronously through prefill and all decode steps until every single request in the batch emits its end-of-sequence (`<eos>`) token or hits `max_tokens`.
- **Execution Bubbles (Wasted Slots)**: When requests in a fixed-membership batch have uneven completion lengths, finished slots cannot be replaced until the batch boundary. Some implementations execute masked or padded work for inactive slots; others avoid part of that work but still lose the opportunity to admit waiting requests.
- **Continuous Batching (Iteration-Level Scheduling)**: Batch composition is re-evaluated dynamically at each discrete token-generation step. Terminated sequences immediately exit the batch and release their resources, while waiting requests are admitted into empty batch slots on the very next iteration.

**Mechanism Explanation:**
In static batching, batch execution duration is governed by the maximum output length: $T_{batch} = \max_{j \in [1, B]} (N_{out, j}) \times \tau_{step}$. The total useful token compute is $\sum_{j=1}^B N_{out, j}$, while the expended compute is $B \times \max_{j} (N_{out, j})$.
Continuous batching decouples individual request lifecycles. At iteration $k$:
1. Check termination condition (`<eos>` or length limit) for each running sequence.
2. Evict finished sequences and release their KV cache blocks to the free pool.
3. If free slots and KV memory exist, select requests from the waiting queue and insert them into the active execution batch.
4. Execute one forward pass across all active sequences.

**Quantitative Model / Derivation:**
As an analytical toy model, let independent generation lengths be continuous $N_{out,j}\sim\mathrm{Uniform}(0,M)$. This is not a production output-length model.
Under static batching:
$$\mathbb{E}[\max_{j \in [1, B]} N_{out, j}] = \frac{B}{B + 1} M$$
$$\text{Efficiency}_{static} = \frac{\mathbb{E}[\sum_{j=1}^B N_{out, j}]}{B \cdot \mathbb{E}[\max_j N_{out, j}]} = \frac{B \cdot (M/2)}{B \cdot \frac{B}{B+1} M} = \frac{B + 1}{2B}$$
For large $B$, this toy-model slot-occupancy ratio approaches $50\%$. It does not prove that half of GPU compute is wasted: masked execution, kernel shapes, memory traffic, admission gaps, and batching overhead determine measured resource waste.
Continuous batching can refill eligible slots and improve occupancy, but it does not imply $100\%$ useful hardware utilization or a universal throughput multiplier. Admission frequency, prefill interference, KV availability, scheduler overhead, and workload shape must be measured.

**Worked Example:**
Consider a static batch of 4 requests with generation lengths:
- Req 1: 10 tokens
- Req 2: 50 tokens
- Req 3: 20 tokens
- Req 4: 100 tokens
Total useful tokens: $10 + 50 + 20 + 100 = 180\text{ tokens}$.
Static batch duration: $100\text{ iterations}$.
Total slots processed: $4 \times 100 = 400\text{ token slots}$.
Bubble waste: $400 - 180 = 220\text{ wasted slots}$ ($55\%$ waste).
Under continuous batching, Req 1 vacates its slot at step 10, permitting Req 5 from the queue to start immediately.

**Knowledge Check:**
1. Under what theoretical workload distribution would static batching achieve identical efficiency to continuous batching?
2. Why can continuous batching introduce higher variance in decode ITL compared to static batching?

**Feedback Contract:**
- *Expected Evidence*: (1) A workload where all requests have strictly identical prompt and generation lengths. (2) Because new requests entering the batch introduce prefill compute or change the active batch size dynamically, altering step execution time.
- *Diagnostic Hint*: When does static batching have zero padding bubbles?
- *Concept to Revisit*: Static Batch Bubble Derivation.

**Independent Practice:**
Proceed to **LAB A** to construct the discrete-event scheduler simulator and measure throughput differences between static and continuous batching.

**Learning Outcome:**
Quantify the compute bubble waste of static batching and explain the iteration-level state transitions of continuous batching.

*(Effort: 40m instruction, Lab A integration)*

---

### Lesson 4.3 — Prefill/Decode Interference & Chunked Prefill

**Engineering Question:**
When does long prefill work interfere with active decodes, and how should chunk size and token budget be chosen without assuming chunking improves every metric?

**Concepts & Definitions:**
- **Prefill vs. Decode Regimes**: Prefill exposes parallel work across prompt tokens; decode advances active sequences incrementally. Prefill often reaches higher arithmetic intensity and decode often becomes sensitive to weight/KV traffic, but model, batch size, sequence length, precision, kernels, and hardware can change either regime.
- **Inter-Phase Interference**: Long prefill work can delay decode through launch ordering or contention for SM, cache, HBM, and runtime budgets. A time-aligned scheduler trace plus GPU counters is required to identify the actual cause.
- **Chunked Prefill**: Dividing an input prompt of length $N_{in}$ into multiple chunks of size at most $C$ (e.g., $C = 512$). In a simple fixed-size model the prompt needs $\lceil N_{in}/C\rceil$ chunks; block alignment and runtime-specific constraints can alter boundaries. Chunking creates opportunities to schedule decode work between or alongside chunks, but does not guarantee that every iteration contains both phases.

**Mechanism Explanation:**
In a hypothetical scheduler that executes an arriving 4,096-token prompt as one non-overlapped prefill step, a measured $150\text{ ms}$ step can add a comparable delay to decodes queued behind it. Actual overlap and contention must be established from a timeline.
In the Sarathi-Serve scheduling pattern:
1. The scheduler maintains a total token budget per iteration: $T_{budget}$ (e.g., 512 tokens).
2. Active decodes are scheduled first: $B_{dec}$ tokens.
3. The next prefill allocation is bounded by both the configured chunk limit and remaining token budget, e.g. $C_{next}\le\min(C_{max},T_{budget}-B_{dec})$ in a simplified token-count model.
4. The long prompt executes chunk 1 ($C$ tokens), appends intermediate KV states to its block table, and yields execution.
5. In iteration 2, the next decode tokens execute alongside chunk 2.
The work budget limits scheduled token work, but does not create a strict wall-clock bound: token cost, cache state, kernel choice, synchronization, and hardware state can still vary.

**Quantitative Model / Derivation:**
Use a measured step-cost function rather than treating tokens as equal-cost work:
$$T_{step}=f(N_{prefill},N_{decode},\text{lengths},\text{cache state},\text{kernels},\text{hardware})$$
The scheduler enforces a work constraint such as $N_{prefill}+N_{decode}\le T_{budget}$, while experiments estimate $f$. Smaller chunks can shorten individual decode interruptions but add more scheduling steps and may reduce kernel efficiency. Larger chunks can improve prefill efficiency while worsening ITL. TTFT, ITL, throughput, fairness, and goodput must all be remeasured.

**Worked Example:**
Treat the following values as exercise inputs, not hardware facts: a decode-only step measures $18\text{ ms}$, an unchunked 4,096-token prefill step measures $120\text{ ms}$, and eight 512-token chunks measure $35\text{ ms}$ each when mixed with the same decode load. Compare the longest injected step and cumulative prompt-processing time. Exact request TTFT and ITL require the arrival order, batch composition, and scheduling timeline, so construct that timeline before calculating them. Then test whether the measurements reproduce across randomized runs and include a counterexample workload with short prompts or high chunk overhead.

**Knowledge Check:**
1. Does chunking necessarily increase mathematical model FLOPs? Which implementation costs can nevertheless increase measured work or elapsed time?
2. If your workload consists exclusively of short prompts ($N_{in} \le 64$) and long decodes ($N_{out} \ge 1,024$), is chunked prefill necessary?

**Feedback Contract:**
- *Expected Evidence*: (1) No: causal-attention arithmetic can be organized without a universal increase in mathematical FLOPs, while kernel shapes, repeated scheduling/launch work, block boundaries, and cache traffic can increase measured cost. (2) Chunking may be unnecessary or harmful for short prompts, but the decision requires measured ITL and throughput rather than prompt length alone.
- *Diagnostic Hint*: How does attention work when prompt tokens are split across iterations?
- *Concept to Revisit*: Causal Attention Across Chunk Boundaries.

**Independent Practice:**
Proceed to **LAB B** to measure ITL variance under heterogeneous prompt arrivals with and without chunked prefill.

**Learning Outcome:**
Diagnose prefill/decode interference and select chunk budgets from workload-faithful TTFT, ITL, throughput, and goodput measurements.

*(Effort: 40m instruction, Lab B integration)*

---

### Lesson 4.4 — Queueing Dynamics, Little's Law, & The Saturation Knee

**Engineering Question:**
Under which assumptions does queueing delay grow nonlinearly near capacity, and how should analytical mean-value checks be combined with empirical load sweeps to select operating headroom?

**Concepts & Definitions:**
- **Arrival Rate ($\lambda$)**: Mean number of requests arriving per second.
- **Service Rate ($\mu$)**: In a single-server queue model with independent service times $S$, $\mu=1/\mathbb{E}[S]$. A batching LLM runtime has a state- and workload-dependent completion curve, so an empirical "requests/s" capacity is not automatically this model parameter.
- **Traffic Intensity / Utilization ($\rho$)**: In the stated single-server model, $\rho=\lambda\mathbb{E}[S]=\lambda/\mu$ and stationarity requires $\rho<1$. GPU utilization counters and token throughput are different quantities.
- **Little's Law**: Fundamental queueing theorem:
  $$L = \lambda W$$
  The average number of requests in the system ($L$) equals arrival rate ($\lambda$) multiplied by average residence time ($W = T_{e2e}$).
  Similarly, for the waiting queue alone: $L_q = \lambda W_q$, where $L_q$ is average queue depth and $W_q$ is average queue wait time ($T_{queue}$).
- **Saturation Knee**: An empirically observed workload-specific region where additional offered load causes queue delay, rejection, or SLO misses to rise sharply relative to useful completions. There is no universal safe-utilization interval.

**Quantitative Model / Derivation:**
Approximating the serving system as an M/G/1 queue (Poisson arrivals, general service time distribution with mean $1/\mu$ and variance $\sigma^2$):
According to the **Pollaczek-Khinchine (P-K) formula**:
$$W_q = \frac{\lambda (\sigma^2 + 1/\mu^2)}{2(1 - \rho)} = \frac{\rho \cdot \frac{1}{\mu} \left(1 + C_v^2\right)}{2(1 - \rho)}$$
where $C_v = \sigma / (1/\mu) = \sigma \mu$ is the coefficient of variation of service time.
Notice two critical dynamics:
1. **The $(1 - \rho)$ Denominator**: In the M/G/1 model, mean waiting diverges as $\rho \to 1$. This is model behavior, not a percentile guarantee for a batching LLM server.
2. **The Variance Term ($C_v^2$ or $\sigma^2$)**: Under the model assumptions, higher service-time variance increases mean queue wait. Real LLM service demand depends on prompt/output lengths, batching, cache state, and scheduler decisions, so the distribution must be measured rather than inferred from output length alone.

**Worked Example:**
A serving instance has maximum capacity $\mu = 10\text{ req/s}$ with mean service time $1/\mu = 100\text{ ms} = 0.1\text{ s}$.
Workload has standard deviation $\sigma = 0.1\text{ s}$ ($C_v = 1.0$).
- At $\lambda = 5\text{ req/s}$ ($\rho = 0.50$):
  $$W_q = \frac{0.50 \cdot 0.1 \cdot (1 + 1)}{2(1 - 0.50)} = \frac{0.10}{1.0} = 0.10\text{ s} = 100\text{ ms}$$
  $$L_q = \lambda W_q = 5 \times 0.10 = 0.5\text{ requests in queue}$$
- At $\lambda = 8\text{ req/s}$ ($\rho = 0.80$):
  $$W_q = \frac{0.80 \cdot 0.1 \cdot (1 + 1)}{2(1 - 0.80)} = \frac{0.16}{0.40} = 0.40\text{ s} = 400\text{ ms}$$
  $$L_q = 8 \times 0.40 = 3.2\text{ requests in queue}$$
- At $\lambda = 9.5\text{ req/s}$ ($\rho = 0.95$):
  $$W_q = \frac{0.95 \cdot 0.1 \cdot (1 + 1)}{2(1 - 0.95)} = \frac{0.19}{0.10} = 1.90\text{ s} = 1,900\text{ ms}$$
  $$L_q = 9.5 \times 1.90 = 18.05\text{ requests in queue}$$
Notice: Moving from $\rho = 0.80$ to $\rho = 0.95$ (an 18% increase in arrival rate) causes queue wait time to jump by nearly $5\times$ ($400\text{ ms} \to 1,900\text{ ms}$).

**Knowledge Check:**
1. If your average response time is $W = 2.5\text{ seconds}$ and your target peak arrival rate is $\lambda = 40\text{ req/s}$, what is the average concurrency $L$ that your cluster must maintain in steady state?
2. If an initially empty deterministic fluid queue receives $\lambda=15$ req/s while departures remain fixed at $\mu=10$ req/s for 20 seconds, with no drops or cancellations, what backlog does that model predict?

**Feedback Contract:**
- *Expected Evidence*: (1) By Little's Law, $L = \lambda W = 40 \times 2.5 = 100$ concurrent requests. (2) Unstable queue growth: $dL_q/dt = \lambda - \mu = 15 - 10 = 5\text{ req/s}$. Over 20 seconds, accumulated queue depth $= 5 \times 20 = 100$ requests waiting.
- *Diagnostic Hint*: Can Little's Law be applied during the transient 20-second burst when $\lambda > \mu$?
- *Concept to Revisit*: Steady-State Assumption in Little's Law vs Transient Accumulation.

**Guided Practice:**
A capacity engineer observes P50 TTFT of $80\text{ ms}$ and P99 TTFT above $3,500\text{ ms}$. Use the P-K formula only to predict how the **mean** wait changes under an explicit M/G/1 approximation. Then state why the formula cannot explain or guarantee P99, and design an open-loop load experiment that measures the actual tail under the changed workload.

**Learning Outcome:**
Apply Little's Law and qualified M/G/1 intuition as mean-value checks, then determine SLO-constrained headroom empirically.

*(Effort: 45m instruction, 15m practice)*

---

### Lesson 4.5 — Runtime Scheduling Policies & Preemption Under Memory Pressure

**Engineering Question:**
How do serving runtimes respond when scheduled work cannot obtain KV capacity, and which parts of that behavior are general mechanisms versus version-specific policy?

**Concepts & Definitions:**
- **General mechanisms**: Delay admission, reject work, preempt and recompute, pause while retaining state, migrate/offload state, or reserve capacity. Each has latency, fairness, compute, bandwidth, and memory costs.
- **Current pinned vLLM V1 observation**: At commit `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`, the scheduler maintains `running` and request queues. When `KVCacheManager.allocate_slots()` fails, it can select a victim and call `_preempt_request()`, which frees KV blocks, marks the request `PREEMPTED`, resets `num_computed_tokens`, increments its counter, and prepends it to the waiting queue.
- **Historical vLLM V0 observation**: Older source/paper versions described SWAP versus RECOMPUTE modes. That path is useful as a design comparison, not as the current vLLM definition.
- **Thrashing hypothesis**: Repeated preemption or migration can reduce completion goodput, but counters and traces must show the loop before it is diagnosed.

**Mechanism Explanation:**
At the pinned V1 revision, the relevant path is conceptually:
```python
new_blocks = self.kv_cache_manager.allocate_slots(request, num_new_tokens, ...)
if new_blocks is None:
    victim = select_victim(policy, running)
    self._preempt_request(victim, scheduled_timestamp)
```
This pseudocode is explanatory; the required artifact must cite the actual pinned symbols and execution path. Victim selection depends on configured scheduling policy and queue state.

**Quantitative Model / Trade-off Comparison:**
- *Idealized cost of transferring state*:
  $$T_{swap\_roundtrip} = \frac{2 \times \text{KV\_Size\_Bytes}}{\text{PCIe\_Effective\_Bandwidth}} + 2 \times \text{DMA\_Overhead}$$
  For an illustrative $4\text{ GiB}$ state and a measured $25\text{ GiB/s}$ effective transfer path, the payload-only round-trip lower bound is:
  $$T_{swap\_roundtrip} \ge \frac{2 \times 4\text{ GiB}}{25\text{ GiB/s}} = 0.32\text{ s} = 320\text{ ms}$$
- *Idealized cost of recomputation*:
  $$T_{recompute} = T_{prefill}(N_{in} + N_{gen\_so\_far}) \approx \frac{2 \cdot P \cdot (N_{in} + N_{gen\_so\_far})}{\text{Peak\_Compute\_FLOPs}}$$
These are lower-bound comparisons. Effective bandwidth, overlap, serialization, current load, kernel efficiency, and timeout/cancellation behavior must be measured.

**Worked Example:**
Given a hypothetical 500 MiB state payload and a **measured** 25 GiB/s effective transfer path, calculate the ideal round-trip transfer lower bound. Compare it with a separately measured recomputation time. Do not substitute peak link bandwidth or peak FLOPs for those measurements.

**Knowledge Check:**
1. Under what condition does enabling CPU KV swapping degrade total cluster throughput worse than immediately aborting preempted requests?
2. How would FCFS, priority scheduling, youngest-first victim selection, and largest-memory-first selection change wasted work and fairness?

**Feedback Contract:**
- *Expected Evidence*: (1) A transfer policy is harmful when transfer/restore work consumes scarce bandwidth without increasing SLO-compliant completions. (2) No victim policy dominates universally: minimizing discarded work can conflict with fairness, priority, memory reclaimed, and deadlines.
- *Diagnostic Hint*: How much work has a request that just started done versus one that is at token 1,900 of 2,000?
- *Concept to Revisit*: LIFO Preemption Victim Selection.

**Guided Practice:**
Trace `vllm/v1/core/sched/scheduler.py` at commit `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`: `Scheduler.schedule()` → `KVCacheManager.allocate_slots()` failure → victim selection → `Scheduler._preempt_request()`. Contrast this verified V1 path with the older V0 SWAP/RECOMPUTE design.

**Learning Outcome:**
Analyze scheduler priority arbitration and evaluate when delay, rejection, recomputation, transfer, or reservation policies reduce useful work or create repeated preemption.

*(Effort: 45m instruction, 15m practice)*

---

### Lesson 4.6 — Admission Control, Backpressure, & Load Shedding

**Engineering Question:**
How do we prevent a serving system from entering goodput collapse when incoming request volume exceeds physical hardware capacity?

**Concepts & Definitions:**
- **Throughput vs. Goodput**:
  - *Throughput*: Total tokens or requests processed by the GPU per second, regardless of whether they meet SLOs or whether the client abandoned the connection.
  - *Goodput*: The rate of work that satisfies a declared usefulness predicate. In this module's exercises that predicate includes successful completion and stated latency SLOs; a production definition must also specify the measured population and treatment of rejections, cancellations, retries, and output-quality failures.
- **Goodput Collapse**: Under some timeout, retry, and cancellation policies, overload makes the system spend resources on work that no longer produces user-visible value. Goodput can fall even while raw throughput or device activity remains high.
- **Admission Control**: A gating mechanism at the gateway or scheduler boundary that evaluates whether sufficient system capacity exists to service a request within its SLO before admitting it into the queue.
- **Backpressure**: A signal or blocking mechanism that causes an upstream producer to slow down. HTTP 429 or gRPC `RESOURCE_EXHAUSTED` is rejection/rate-limit feedback; it becomes effective backpressure only if upstream behavior actually reduces offered load.
- **Load Shedding**: The deliberate, early rejection of excess requests when queues exceed a safe threshold, preserving full service capacity for already-admitted requests.

**Mechanism Explanation:**
Admission control algorithms employ three primary strategies:
1. **Concurrency Limits ($L_{max}$)**: Bound the total active requests in the serving engine:
   $$\text{If } |Queue_{waiting}| + |Queue_{running}| \ge L_{max} \implies \text{apply the configured admission outcome}$$
2. **Time-in-Queue Bounding (Virtual Deadline)**:
   A simple estimate $\widehat{W}_q=|Queue_{waiting}|/\mu$ requires homogeneous work and approximately constant service rate. Treat it as a calibrated predictor with uncertainty rather than proof that an SLO violation is guaranteed.
3. **Sojourn-Time Queue Control**: Use measured queue residence time as an overload signal and shed according to a declared policy. CoDel is one reference algorithm from packet queues; applying it to heterogeneous LLM jobs requires validation rather than copying its constants.
4. **Rate Limiting**: A token-bucket or related limiter bounds accepted arrivals over a time window and permits a configured burst. It controls ingress rate; it is not the same mechanism as sojourn-time-based queue dropping.

**Quantitative Model / Trade-off Comparison:**
For an initially empty deterministic fluid queue with fixed rates and no drops:
$$\lambda>\mu \implies Q(t)=(\lambda-\mu)t,\qquad \widehat{W}_q(t)=Q(t)/\mu$$
Client timeouts can reduce user-visible goodput when stale work is not cancelled, but neither the fluid model nor $\lambda>\mu$ alone proves goodput goes to zero. In a simplified steady-rate exercise, admitted rate cannot sustainably exceed measured completion capacity. Real goodput additionally depends on SLO attainment, failures, cancellations, retries, and output quality. Load shedding trades rejected-user utility for protection of selected objectives; it does not create 100% reliability.

**Worked Example:**
A fluid-model exercise assumes fixed completion capacity $\mu=100\text{ req/s}$, an initially empty queue, no service-rate change with batching, and a client timeout of $2.0\text{ s}$.
A traffic spike arrives at $\lambda = 250\text{ req/s}$ for 60 seconds.
- *Scenario A (No Admission Control, FIFO queue)*:
  Queue accumulates $(250 - 100) = 150\text{ req/s}$.
  After 15 seconds, queue depth reaches $15 \times 150 = 2,250\text{ requests}$.
  Wait time in queue for newly scheduled requests $= 2,250 / 100 = 22.5\text{ seconds}$.
  Requests at the tail of this modeled backlog exceed the timeout. Exact goodput requires the arrival/departure timeline and cancellation semantics; it is not automatically zero.
- *Scenario B (Strict Admission Control with Load Shedding)*:
  Gateway limits queue size to 50 requests ($0.5\text{ s}$ max queue delay).
  Gateway sheds the offered excess using the configured rejection contract.
  The admitted $100\text{ req/s}$ are processed with $T_{queue} < 500\text{ ms}$.
  The model predicts up to $100\text{ req/s}$ completions; measured SLO-compliant goodput may be lower.

**Knowledge Check:**
1. Under what timeout, retry, cancellation, and user-utility assumptions is early rejection preferable to queueing?
2. What client-side mechanism must accompany HTTP 429 load shedding to prevent an immediate retry storm (thundering herd)?

**Feedback Contract:**
- *Expected Evidence*: (1) Early rejection before engine admission avoids KV and model-execution work for that request, though gateway work remains. Queueing is harmful only when it later times out, is cancelled, or displaces more valuable work. (2) Retry budgets plus exponential backoff with jitter, or another policy that reduces synchronized retry load.
- *Diagnostic Hint*: What happens when 1,000 clients simultaneously retry an HTTP 429 after exactly 1.000 seconds?
- *Concept to Revisit*: Thundering Herd and Jittered Backoff.

**Guided Practice:**
Design an admission policy for an enterprise RAG endpoint. Prompt length is 3,000 tokens, generation length is 200 tokens, and the client timeout is 1,200 ms. The queue has 12 requests. Explain why a single aggregate "1,500 prompt tokens/sec" value is insufficient to decide admission, list the missing service/scheduling information, and propose a calibrated prediction with an uncertainty margin.

**Learning Outcome:**
Design, configure, and defend admission control and load shedding mechanisms to safeguard serving goodput under overload.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 4.7 — Capacity Planning & Bottleneck Discrimination

**Engineering Question:**
How do we rank interacting queueing, compute, host, network, and memory constraints, and how do we size a deployment without mistaking a lower-bound estimate for a capacity guarantee?

**Concepts & Definitions:**
- **Competing Constraint Hypotheses**:
  1. *Queueing pressure*: arrivals, bursts, or a reduced effective service rate create backlog; this describes system state, not a physical resource by itself.
  2. *Device execution pressure*: one or more kernels are limited by compute throughput, memory bandwidth, synchronization, launch behavior, or inefficient shapes. Aggregate SM activity alone does not identify which.
  3. *KV-capacity pressure*: allocation limits block admission or trigger runtime-specific preemption even when some compute counters appear low.
  4. *Host/network/control-plane pressure*: tokenization, serialization, communication, scheduler overhead, or downstream backpressure delays work outside the main GPU kernels.
Multiple constraints can interact or transition during one incident.

**Systematic Discrimination Matrix:**
To diagnose an underperforming cluster, collect the following telemetry metrics and cross-reference against the discrimination signatures:

| Signal | Supports a hypothesis when correlated with | Why it is insufficient alone |
|---|---|---|
| **Queue residence / depth** | arrival trace, completion rate, request-state transitions | backlog can be caused by any reduction in effective service rate |
| **Prefill/decode spans** | token counts, batch composition, kernel timeline | longer spans can reflect scheduler delay, shapes, contention, or host gaps |
| **GPU counters** | per-kernel arithmetic intensity, achieved bandwidth/FLOPs, occupancy | aggregate utilization hides idle gaps and mixed kernels |
| **Free KV blocks / allocation failures** | admission blocks, preemption events, sequence footprints | low free blocks may be an intentional steady state without harm |
| **Preemption rate** | recompute/transfer work and latency changes | correlation does not establish that preemption initiated the incident |
| **Host and network timeline** | CPU queues, serialization, transport, client timestamps | engine-only metrics omit end-to-end delays |

**Quantitative Capacity Planning Model:**
Given:
- Target peak arrival rate: $\lambda_{peak}\text{ req/s}$
- Mean prompt length: $\bar{N}_{in}$; Mean decode length: $\bar{N}_{out}$
- TTFT SLO: $T_{TTFT\_SLO}$; ITL SLO: $T_{ITL\_SLO}$
- Model parameters: $P\text{ billion}$; Precision: $B_{param}\text{ bytes/param}$
- GPU specs: HBM capacity $M_{gpu}$, Memory bandwidth $BW_{mem}$, FP16 Peak Tensor Compute $C_{peak}$

*Step 1: Weight and Static Memory Footprint*:
$$M_{weights} = P \times B_{param}$$
$$M_{KV\_available} = M_{device,usable} - M_{weights,resident} - M_{nonKV,measured} - M_{headroom}$$
The non-KV term and allocator/headroom policy are measured for the selected runtime and configuration; they are not universal constants.

*Step 2: Concurrency Sizing via Little's Law*:
Average request duration:
$$W = T_{TTFT} + (\bar{N}_{out} - 1) \times T_{ITL}$$
Target average concurrency:
$$L = \lambda_{peak} \times W$$

*Step 3: KV Cache Concurrency Ceiling*:
Per-request peak KV memory demand:
$$KV_{req} = 2 \times N_{layers} \times N_{kv\_heads} \times d_{head} \times B_{elem} \times (\bar{N}_{in} + \bar{N}_{out})$$
Maximum concurrent sequences supported by memory:
$$L_{KV\_max} = \frac{M_{KV\_available}}{KV_{req}}$$
If the rough $L_{KV\_max}$ estimate is below required resident concurrency, KV capacity is a candidate constraint. Mean sequence length, allocator overhead, weights, activations, prefix reuse, batching, and routing can invalidate the estimate; confirm with runtime measurements before selecting an intervention.

**Worked Example:**
- Exercise assumption: $80\text{ GiB}$ usable device memory after any platform reservation.
- Model payload estimate: $8\times 10^9$ parameters at 2 bytes each $=16\times10^9$ bytes $\approx14.90\text{ GiB}$; real resident weight memory must be measured.
- Measured non-KV allocations plus reserved headroom: $5.10\text{ GiB}$.
- Exercise KV budget: $M_{KV\_available}=80-14.90-5.10=60.00\text{ GiB}$.
- KV footprint for 8B model (32 layers, 8 KV heads, $d_{head}=128$, FP16):
  $$\text{Bytes/token} = 2 \times 32 \times 8 \times 128 \times 2 = 131,072\text{ bytes} = 128\text{ KiB/token}$$
- Workload: $\bar{N}_{in} = 2,048$, $\bar{N}_{out} = 512$. Total tokens $= 2,560$.
- Logical KV payload per request: $2,560 \times 128\text{ KiB}=320\text{ MiB}=0.3125\text{ GiB}$, before allocator metadata, block rounding, and prefix sharing.
- Maximum memory concurrency:
  $$L_{KV\_max,logical}=\left\lfloor\frac{60\text{ GiB}}{0.3125\text{ GiB}}\right\rfloor=192\text{ concurrent requests}$$
If expected workload has $\lambda = 50\text{ req/s}$ and $W = 5\text{ s}$, then $L = 50 \times 5 = 250\text{ concurrent requests}$.
Because the mean-value concurrency estimate exceeds the simplified KV ceiling, this configuration fails the analytical screening check. It does not prove a crash or prescribe one remedy; run a workload-faithful load sweep and compare admission, replication, model placement, representation, and SLO trade-offs.

**Knowledge Check:**
1. A cluster exhibits elevated P99 TTFT, low sampled SM activity, and abundant free KV blocks. Give at least three competing hypotheses and the next discriminating measurement.
2. If doubling batch size doubles ITL while compute counters are high, what additional evidence distinguishes compute saturation from synchronization, queueing, or host-side delay?

**Feedback Contract:**
- *Expected Evidence*: A ranked hypothesis set—not an exactly-one classifier—using queue residency, request states, host/GPU timelines, kernel counters, KV state, and client/network timing.
- *Diagnostic Hint*: If SM utilization is 35% and memory is free, is the GPU doing work while the request waits?
- *Concept to Revisit*: Serving Bottleneck Discrimination Matrix.

**Guided Practice:**
For a 70B exercise with a stated $140\text{ GiB}$ resident weight footprint, $\lambda=20\text{ req/s}$, mean response time $W=8\text{ s}$, and a stated logical $KV_{req}=2.0\text{ GiB}$, compute the mean-concurrency requirement and memory lower bounds. Then list the missing placement, parallelism, per-replica service curve, tail-latency, burst, allocator, failure-headroom, and workload-distribution measurements that prevent these averages from determining an exact H100 count.

**Learning Outcome:**
Discriminate queueing, compute, and memory bottlenecks from production telemetry and calculate hardware provisioning requirements from workload parameters.

*(Effort: 45m instruction, 15m practice)*

---

## 05 Literature & Production Source Map

**CANONICAL**
- *Orca: A Distributed Serving System for {Transformer-Based} Generative Models* (Yu et al., OSDI 2022). [Mechanism: Continuous Batching & Iteration-Level Scheduling]
- *Why it matters*: Established iteration-level scheduling and selective batching as reference serving mechanisms.
  - *Key Sections*: Section 3 (Iteration-Level Scheduling), Section 4 (Selective Batching).
- *Efficient Memory Management for Large Language Model Serving with PagedAttention* (Kwon et al., SOSP 2023). [Mechanism: Paged KV Cache & vLLM Architecture]
  - *Why it matters*: Demonstrated paged KV management and memory-aware serving; its scheduler implementation is historical, not the current vLLM V1 path.
  - *Key Sections*: Section 4.3 (Memory-Aware Scheduling), Section 4.4 (Preemption via Swapping/Recomputation).

**PRODUCTION**
- `vllm/v1/core/sched/scheduler.py` and `vllm/v1/core/kv_cache_manager.py`. Pinned commit: `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`; statically verified 2026-09-25.
  - *Inspection Focus*: `Scheduler.schedule()`, `KVCacheManager.allocate_slots()`, and `Scheduler._preempt_request()`.
- Other production runtimes should be traced only at a pinned revision. Their queue, budgeting, and preemption semantics are comparison points, not assumed copies of vLLM; unpinned implementation claims remain `TODO_VERIFY`.

**FRONTIER**
- *Taming Throughput-Latency Tradeoff in LLM Inference with Sarathi-Serve* (Agrawal et al., OSDI 2024).
  - *Why it matters*: Provides measured evidence for chunked-prefill/decode-piggyback trade-offs on evaluated configurations.
- *DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving* (Zhong et al., OSDI 2024).
  - *Why it matters*: Evaluates phase disaggregation under explicit workloads and exposes placement and KV-transfer trade-offs.
- *Splitwise: Efficient Generative LLM Serving Using Phase Splitting* (Patel et al., ISCA 2024).
  - *Reading Priority*: DIRECTED_READ for understanding phase separation economics and network KV transfer overheads.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs require following the rigorous scientific loop:
$$\text{PREDICT} \to \text{BUILD} \to \text{MEASURE} \to \text{EXPLAIN} \to \text{BREAK} \to \text{IMPROVE} \to \text{FALSIFY}$$

### LAB A — Serving Scheduler Simulator: Static vs. Continuous Batching
- **Objective**: Build a discrete-event scheduler simulator in Python that processes heterogeneous prompt and generation workloads, comparing static batching against iteration-level continuous batching.
- **Pre-Registered Hypothesis**: Under the lab's declared execution-cost model, sustained backlog, and heterogeneous output lengths, continuous batching will reduce unfillable fixed-membership slot-time. State a minimum practically important throughput effect before running the experiment; do not treat a supplied multiplier as a fact.
- **Independent Variables**: Batching policy (`STATIC` vs `CONTINUOUS`), batch size $B \in [4, 8, 16, 32]$, generation length distribution variance ($\sigma \in [10, 100, 500]$ tokens).
- **Dependent Variables**: Token throughput (tokens/sec), execution bubble percentage, P50/P99 TTFT, P50/P99 TPOT.
- **Break & Falsify**: Inject a degenerate workload where all requests have strictly identical prompt and generation lengths ($N_{in}=128, N_{out}=128$). Falsify the claim that continuous batching is universally superior by measuring the scheduling loop overhead.
- **Alignment**: Explicitly exercises Lesson 4.2.
- **Effort Estimate**: 3h implementation, 1h analysis (4h total).

### LAB B — Chunked Prefill & ITL Variance Under Mixed Workloads
- **Objective**: Implement chunked prefill mechanics within the scheduler simulator and measure inter-token latency stability during long prompt arrivals.
- **Pre-Registered Hypothesis**: For the selected cost model and mixed workload, an intermediate chunk size will reduce the upper tail of ITL relative to monolithic prefill, while very small chunks can sacrifice prefill efficiency. Declare the expected effect and acceptable TTFT/throughput cost before measurement.
- **Independent Variables**: Prefill chunk size $C \in [128, 256, 512, 1024, \infty]$ (where $\infty$ denotes unchunked), prompt length distribution.
- **Dependent Variables**: P50/P95/P99 ITL, TTFT, throughput, scheduled step count, and simulator cost components. If an optional GPU experiment is run, add achieved FLOPs/bandwidth and kernel timelines rather than inventing hardware efficiency in the simulator.
- **Break & Falsify**: Lower chunk size to $C=32$ and test whether extra steps or implementation overhead reduce throughput. A result with no degradation falsifies the proposed overhead mechanism for that setup.
- **Alignment**: Explicitly exercises Lesson 4.3.
- **Effort Estimate**: 2.5h implementation, 1h analysis (3.5h total).

### LAB C — Queueing Dynamics, Little's Law, & Saturation Breakdown
- **Objective**: Subject the serving simulator to varying arrival rates modeled as Poisson processes, experimentally locating the saturation knee and validating Little's Law.
- **Pre-Registered Hypothesis**: For a stable run measured after warm-up with consistent arrival and residence boundaries, $L\approx\lambda W$ within a tolerance justified by finite-sample uncertainty. Near overload, the steady-state estimator becomes unusable if the run never equilibrates; finite queues, drops, or cancellations require reporting admitted/completed populations rather than claiming Little's Law itself failed.
- **Independent Variables**: Arrival rate $\lambda \in [0.2 \mu, 0.4 \mu, 0.6 \mu, 0.8 \mu, 0.95 \mu, 1.2 \mu]$, prompt length variance ($C_v \in [0.2, 1.0, 2.5]$).
- **Dependent Variables**: Mean queue depth $L_q$, queue wait time $W_q$, system occupancy $L$, goodput (requests meeting a 500 ms TTFT SLO).
- **Statistical Rigor**: Use independently seeded trials, predeclare warm-up and stopping rules, report the estimator and confidence-interval method, and increase trial count until uncertainty is adequate for the chosen effect size. Ten trials may be a starting point, not a universal guarantee.
- **Alignment**: Explicitly exercises Lesson 4.4 and Lesson 4.7.
- **Effort Estimate**: 2h implementation, 1h analysis (3h total).

### LAB D — Preemption Under Memory Pressure & Admission Control
- **Objective**: Implement physical KV block accounting in the scheduler simulator. Simulate high-concurrency traffic that exceeds GPU HBM, triggering preemption (swapping vs recomputation), and integrate an admission control gateway.
- **Pre-Registered Hypothesis**: Under a declared timeout/cancellation/retry model, bounded admission will protect selected SLO-compliant goodput during overload at the cost of rejected utility. Sweep offered load and queue limits; a result where unbounded queueing preserves equal or better declared utility falsifies the proposed policy for that workload.
- **Action**: Compare at least two victim/admission policies, parameterize measured or explicitly synthetic transfer and recomputation costs, and report completions, SLO-goodput, rejections, cancellations, wasted work, and fairness. Keep the simulator's response contract generic unless a specific gateway protocol is part of the experiment.
- **Alignment**: Explicitly exercises Lesson 4.5 and Lesson 4.6.
- **Effort Estimate**: 2.5h implementation, 1h analysis (3.5h total).

---

## 07 Break / Incident Scenarios

### Incident 04.1: The Midnight Latency Collapse
- **Incident Symptoms**:
  At 00:14 UTC, production alerts fire for a multi-tenant enterprise LLM cluster serving customer support chatbots and backend document indexing jobs.
  - Overall P99 TTFT spikes from a baseline of $210\text{ ms}$ to $14,800\text{ ms}$.
  - Customer chat clients report widespread HTTP 504 Gateway Timeouts.
  - GPU compute metrics show that all 16 node GPUs remain pinned at $100\%$ SM utilization.
  - Total output token generation rate drops by $82\%$, despite the cluster being at 100% compute load.
  - Ingress traffic volume (requests per second) increased by only $12\%$ over normal midnight baseline.
- **Diagnostic Protocol (Task)**:
  The learner must act as the primary on-call systems engineer and execute the following investigation:
  1. *Formulate Competing Hypotheses*: Formulate at least 3 distinct, technically rigorous hypotheses that could explain the symptoms (e.g., Hypothesis A: Ingress arrival burst causing pure queueing saturation; Hypothesis B: Document indexing job submitted long-context prompts triggering prefill/decode interference; Hypothesis C: Dynamic sequence length growth caused KV cache exhaustion, triggering a preemption recomputation thrashing cascade).
  2. *Rank Hypotheses by Initial Plausibility*: Evaluate the reported symptoms against each hypothesis. Do not assume a proportional response: a modest offered-load increase can cross timeout, retry, cancellation, or batching thresholds and cause a much larger goodput change even without KV thrashing.
  3. *Identify Missing Evidence*: Specify exactly what telemetry metrics and logs must be inspected to discriminate the mechanism (e.g., scheduler preemption counters, KV allocation failures, transfer/offload bytes when that feature is enabled, prompt-length histograms, and queue-residency timers).
  4. *Design Discriminating Test*: Formulate a targeted query or controlled intervention whose predicted observations distinguish the leading hypotheses; state what result would falsify each explanation within the test's scope.
  5. *Execute Causal Diagnosis*: Rank the supported mechanism or interacting mechanisms and state which alternatives the evidence rules out.
  6. *Prescribe Immediate Mitigation & Long-Term Intervention*: Define the immediate remediation (e.g., shedding the background indexing queue via admission control) and the architectural prevention (e.g., separate tenant pools or chunked prefill with strict max batched tokens).
  7. *Remeasurement & Post-Mortem Defense*: State the quantitative verification criteria proving the incident is resolved.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Architecture Transfer Problem: Multi-Tenant Voice & Document Platform
You are the Principal Inference Architect for an enterprise AI platform that serves two drastically different workloads on a shared cluster of 8 $\times$ NVIDIA H100 (80 GiB HBM3) GPUs running a 70B parameter model:
1. **Interactive Real-Time Voice Agent**: Requires strict streaming latency SLOs: P95 TTFT $< 300\text{ ms}$, P95 ITL $< 30\text{ ms}$. Average prompt: 400 tokens; average generation: 150 tokens. Traffic: 40 requests/sec with high burstiness.
2. **Asynchronous Document Summarization**: Long-context batch jobs. Average prompt: 16,000 tokens; average generation: 800 tokens. Traffic: 2 requests/sec steady background volume.

Under the current default single-engine deployment, whenever a document summarization request arrives, the voice agent experiences severe audio stuttering (P99 ITL spikes to $180\text{ ms}$), and conversational users frequently disconnect.

You must design a comprehensive serving architecture, scheduling configuration, and capacity allocation plan that targets the voice agent's strict SLOs while maximizing document summarization throughput, and define the load and failure tests required before making a production guarantee.

**Required Deliverables**:
1. **Quantitative Workload & Resource Model**:
   - Calculate parameter memory footprint, available KV cache memory per GPU, and per-request KV demand for both workloads.
   - Apply Little's Law to calculate steady-state concurrency for both workloads.
2. **Bottleneck Prediction & Theoretical Analysis**:
   - Formulate a formal hypothesis identifying the primary hardware bottleneck causing the voice agent ITL spikes.
   - Explain why standard continuous batching fails to isolate these workloads.
3. **Three Candidate Architectural Interventions**:
   - Formulate three technically distinct candidate interventions (e.g., (A) Aggressive Chunked Prefill with token budgeting; (B) Prefill-Decode Disaggregation with dedicated voice and summary pools; (C) Strict Priority Preemption with CPU KV Swapping).
4. **Quantitative Derivation of Expected Effects**:
   - For each intervention, derive the expected effect on voice agent TTFT, voice agent ITL, and document summarization throughput. State all explicit assumptions.
5. **Trade-Off & Failure Mode Analysis**:
   - Detail the operational risks, hardware utilization costs, and failure modes of each intervention.
6. **Rejection of Inappropriate Interventions**:
   - Explicitly analyze and reject at least one seemingly plausible intervention (e.g., explaining why simply increasing the global continuous batch size or relying on CPU swapping worsens the voice agent's latency collapse).
7. **Final Architecture Defense**:
   - Defend your recommended architecture, detailing exact scheduler settings (`max_num_batched_tokens`, chunk sizes, queue limits, admission control rules).
8. **Uncertainty, Falsification, & Rollback Strategy**:
   - Identify what workload change would falsify your design assumptions (e.g., voice prompts growing to 4,000 tokens).
   - Define exact operational metrics and automated rollback triggers that would reverse the architectural changes in production.

---

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace
The learner must submit a verified code trace document analyzing the scheduling loop of a production serving runtime. The reference trace uses vLLM commit `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`.
The trace must map out:
1. Entry point into `Scheduler.schedule()`.
2. How running work is budgeted and where `KVCacheManager.allocate_slots()` is called.
3. The path from allocation failure through victim selection to `Scheduler._preempt_request()`.
4. How chunked prefill slices waiting requests into the scheduled token budget.
5. One concrete policy trade-off where the production scheduler's heuristic favors one objective over another; do not assert a universally optimal queue discipline without an objective and queue model.

### Rubric Dimensions
- **Mechanistic Reasoning**: *Insufficient* describes high-level concepts without execution mechanisms. *Competent* explains step-by-step scheduler decisions and hardware contention. *Strong* links scheduler loop logic directly to GPU hardware execution regimes (Tensor Core saturation vs HBM bandwidth).
- **Quantitative Reasoning**: *Insufficient* quotes formulas without assumptions. *Competent* applies Little's Law, qualified queueing models, and memory estimates with units and boundaries. *Strong* compares predictions with load measurements and explains where state-dependent service invalidates the analytical approximation.
- **Experiment Design & Rigor**: *Insufficient* runs uninstrumented tests. *Competent* isolates variables and validates hypotheses with controlled workloads. *Strong* constructs rigorous falsification experiments and accounts for statistical variance across Monte Carlo trials.
- **Failure Diagnosis**: *Insufficient* guesses root causes from high-level symptoms. *Competent* distinguishes queueing, compute, and memory bottlenecks using metric signatures. *Strong* executes a systematic elimination protocol during multi-factor incidents without leaking hypotheses.
- **Architecture Defense**: *Insufficient* recommends a popular library without justification. *Competent* balances throughput vs latency trade-offs using workload models. *Strong* provides a mathematically defended architecture plan with explicit rejection of inappropriate alternatives and rollback triggers.

---

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| **Latency Metrics Decomposition** | Lesson 4.1 | Lesson 4.1 Guided Practice | LAB A / Mastery Deliverable 1 | Latency Profiler Trace / Workload Model |
| **Continuous Batching Dynamics** | Lesson 4.2 | Lesson 4.2 Independent Practice | LAB A | Scheduler Simulator Code & Benchmark Report |
| **Chunked Prefill & ITL Stabilization** | Lesson 4.3 | Lesson 4.3 Guided Practice | LAB B / Mastery Deliverable 3 | ITL Variance Measurement & Comparison Artifact |
| **Queueing Theory & Little's Law** | Lesson 4.4 | Lesson 4.4 Guided Practice | LAB C / Mastery Deliverable 1 | M/G/1 Model Derivation & Empirical Plot |
| **Preemption & Memory Arbitration** | Lesson 4.5 | Lesson 4.5 Guided Practice | LAB D / Incident 04.1 | Production Source Trace / Diagnostic Report |
| **Admission Control & Load Shedding** | Lesson 4.6 | Lesson 4.6 Guided Practice | LAB D / Mastery Deliverable 7 | Gateway Simulator & Goodput Curve |
| **Bottleneck Discrimination** | Lesson 4.7 | Lesson 4.7 Guided Practice | Incident 04.1 / Mastery Deliverable 2 | Root Cause Analysis Protocol & Defense |

---

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria
A learner successfully completing Module 04 must be able to:
1. Derive logical memory estimates, state their exclusions, and measure the saturation knee for a specified workload distribution.
2. Build and instrument a discrete-event continuous batching scheduler simulator.
3. Evaluate chunked prefill parameters across TTFT, ITL, throughput, fairness, and goodput, including a workload where chunking is harmful.
4. Trace execution flow through the scheduler loop of a production serving engine.
5. Diagnose complex serving incidents and systematically isolate queueing, compute, and memory exhaustion failures.
6. Design, defend, and specify an enterprise serving architecture under conflicting multi-tenant latency SLOs.

### Module Wrap-Up (Final Mental Model Reconstruction)
- **The Core Invariant**: An LLM serving engine is a state-dependent queueing and resource-arbitration system. Prefill and decode frequently present different execution shapes, but their actual bottlenecks must be measured for the selected model, workload, runtime, and hardware.
- **The Path to High Goodput**:
  $$\text{Arrivals} \to \text{Overload Controls} \to \text{Queue/Admission} \to \text{Scheduling and Batching} \to \text{GPU/KV Constraints} \to \text{Completion, Rejection, or Cancellation}.$$
- When degradation occurs, align client, queue, scheduler, host, GPU, KV-allocation, and network timelines; rank competing explanations; intervene; then repeat the same measurements. No single ordering or signal identifies every incident.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

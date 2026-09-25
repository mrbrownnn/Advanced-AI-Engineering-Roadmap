# Module 03 — KV Cache Engineering

## 00 Why This Module Exists
KV cache is often one of the dominant sources of dynamic memory consumption in LLM serving and can become a primary concurrency constraint, particularly under long-context or high-concurrency workloads. How you manage KV memory determines how many requests you can serve simultaneously, how you handle memory pressure, and how you exploit sharing across requests. This is where OS-level systems thinking meets ML inference.

**Module Orientation**
- **Engineering Problem**: Solving memory exhaustion and latency spikes under continuous batching.
- **What you will do**: Build a paged KV allocator, simulate external vs internal fragmentation, diagnose a latency incident, and design an architecture transfer plan.
- **Environment**: A basic Python environment for the simulator (Labs A-C). Access to a single GPU is optional but recommended for latency bounds testing (Lab D).

## 01 Prerequisites and Scope
- **Architecture**: You understand how multi-head attention (MHA), grouped-query attention (GQA), multi-query attention (MQA), and multi-head latent attention (MLA) differ in KV footprint (Module 01).
- **Hardware**: You understand the GPU memory hierarchy, HBM bandwidth constraints, and basic CUDA memory allocation concepts (Module 02).

This module owns KV state geometry, allocation, block tables, sharing, prefix identity, lifecycle/eviction, representation, and tiering. It exposes feasibility and pressure signals to the scheduler, but request queueing, admission objectives, fairness, preemption policy, TTFT/TPOT, and capacity belong to Module 04. General quantization/kernel optimization belongs to Module 05, and distributed placement/transport belongs to Module 20.

Research cutoff: **2026-09-25**. Runtime behavior is claimed only against pinned source revisions.

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
  guided_practice: 2h
  labs: 13h
  assessment: 3h
  source_trace: 2h
  total: 25h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map
KV state in autoregressive inference → KV tensor dimensions → analytical KV memory model → prefill vs decode lifecycle → dynamic sequence growth → continuous batching pressure → naive / contiguous allocation → reservation waste + fragmentation → logical vs physical KV address space → paged KV management → block tables → allocation / deallocation → block-size trade-offs → reference counting → shared blocks → copy-on-write → scheduler interaction → memory pressure → preemption / recomputation / swapping → prefix caching → cache identity / matching → partial blocks → cache lifecycle → eviction → radix / tree-based reuse → cache locality → cache-aware routing → KV quantization → KV offloading / memory tiering → distributed KV implications → prefill-decode disaggregation / KV transfer.

## 04 Lessons

### Lesson 3.1 — KV State & Quantitative Model

**Engineering question:**
What is the logical KV payload per retained token under explicit architecture assumptions, and why is it not exact process memory?

**Concepts & Definitions:**
- **Autoregressive KV Reuse**: In LLM generation, previous tokens' Key (K) and Value (V) representations do not change when computing the next token. KV caching avoids recomputing K/V representations for previously processed tokens during autoregressive decoding, while each new query still attends over the cached prefix and KV reads/attention work still grow with context length.
- **Prefill vs. Decode Lifecycle**: *Prefill* computes the KV cache for the entire prompt in one highly parallel forward pass. *Decode* generates token-by-token. Prefill often tends toward compute-heavy regimes, and decode often tends toward memory-bandwidth-sensitive regimes. However, the actual bottleneck depends on hardware, batch size, architecture, and kernel implementation. The learner must PREDICT THE BOTTLENECK → PROFILE → COMPARE PREDICTION WITH OBSERVATION.

**Quantitative Model / Derivation:**
`Logical KV payload bytes per retained token = 2 (K and V) × bytes_per_element × cached_layers × num_kv_heads × head_dim`
*Assumptions*: Uniform uncompressed decoder self-attention with one stored K and V vector per cached layer/head/token. This excludes sharding/replication, block rounding, alignment, scales/metadata, allocator state, latent attention, sliding windows, recurrent/state-space state, and hybrid layouts. `bytes_per_element` is 2 for FP16/BF16.

**Worked Example:**
Example Transformer configuration: 32 layers, 32 query heads, 32 KV heads (MHA), head_dim=128, FP16.
`Bytes/token = 2 × 2 × 32 × 32 × 128 = 524,288 bytes (~0.5 MiB)`.
A 2048-token sequence requires ~1 GiB of KV cache.
80 such sequences would require approximately 80 GiB for KV alone and therefore cannot fit on an 80 GiB device once model weights and runtime memory are included.
*Validation hierarchy*: analytical KV estimate → runtime KV allocator metrics → framework allocator metrics → process/device VRAM.

**Knowledge Check:**
1. Derive the bytes/token for a model with 64 layers, 8 KV heads, head_dim 128, in FP8.
2. Why is the analytical KV estimate not equal to `nvidia-smi` usage?

**Guided Practice:**
For a screening exercise, assume an 80 GiB device, the standard formula above, every request exactly 4096 retained tokens, 15 GiB of parameters, 2 GiB of other fixed buffers, and no headroom, allocator rounding, workspace growth, or failures. Calculate the payload-only upper bound on concurrent requests, then list why it is unsafe as a deployment limit.

**Feedback Contract:**
- *Expected Evidence*: Floor of available VRAM divided by per-request KV payload, labeled as an optimistic upper bound rather than safe capacity.
- *Common Failure*: Using the full 80 GiB or forgetting to multiply by 4096.
- *Diagnostic Hint*: After subtracting weights and buffers, how much memory is actually free for KV?
- *Concept to Revisit*: Total device memory vs KV-only analytical requirement.

**Learning outcome:**
Derive KV memory from a model architecture and predict theoretical batch limits.

*(Effort: 30m instruction, 15m practice)*

---

### Lesson 3.2 — Memory Allocation & Fragmentation

**Engineering question:**
What are the different types of memory waste in LLM inference, and how do they manifest under continuous batching?

**Concepts & Definitions:**
- **Abstraction Hierarchy**: Request/logical KV demand → Inference-engine allocation policy → KV block/page allocation → Framework allocator → CUDA/device allocator → Physical HBM.
- **Reservation Waste (Logical)**: Over-allocating memory based on a predicted `max_sequence_length` that the request never reaches.
- **Internal Fragmentation (Block level)**: Wasted space *inside* an allocated block.
- **External Fragmentation (Engine level)**: Free memory scattered in non-contiguous chunks, preventing large contiguous logical allocations.
- **Allocator-level Fragmentation**: Waste caused by the OS/CUDA memory manager failing to coalesce freed segments.

**Mechanism Explanation:**
Naive systems allocate contiguous tensors of size `max_length`. If requests finish early or at different times, the inference engine cannot allocate a new contiguous chunk. Paged KV changes how the inference runtime maps logical sequence growth onto physical KV storage; it does NOT magically defragment all GPU memory layers.

**Worked Example:**
Request A (needs 1000 tokens), Request B (needs 100), Request C (needs 1000).
Naive continuous contiguous allocation reserves 2048 tokens per request. Total requested: 2100. Total allocated: 6144. Reservation waste: 4044 tokens.

**Knowledge Check:**
1. At which layer of the abstraction hierarchy does reservation waste occur?
2. If your inference engine uses Paged KV, can you still experience allocator-level fragmentation from `cudaMalloc`?

**Feedback Contract:**
- *Expected Evidence*: Identifying the correct layer of abstraction.
- *Common Failure*: Believing Paged KV solves `cudaMalloc` fragmentation.
- *Diagnostic Hint*: Does Paged KV manage HBM directly or does it ask the framework allocator for memory?
- *Concept to Revisit*: Fragmentation Abstraction Hierarchy.

**Independent Practice:**
Proceed to **LAB A** to build the Allocation Simulator.

**Learning outcome:**
Distinguish fragmentation/waste mechanisms across the abstraction hierarchy.

*(Effort: 30m instruction, Lab A integration)*

---

### Lesson 3.3 — Paged KV Memory Management

**Engineering question:**
How can we store growing sequential data in non-contiguous physical memory blocks to mitigate external fragmentation?

**Concepts & Definitions:**
- **Logical Token Position**: The token's index in the logical sequence (e.g., token 15).
- **Logical Block**: A fixed-size grouping of logical token positions (e.g., tokens 0-15).
- **Physical Block**: Contiguous storage capacity allocated from the GPU KV memory pool.
- **Block Table**: Per-sequence mapping from logical block indices to physical blocks.
- **Block Size**: The number of tokens per block.
- **Allocator Invariants**: Every allocated logical block maps to exactly one valid physical block. A physical block cannot simultaneously be in the free pool and allocated. Mappings remain stable unless explicitly remapped/reclaimed.

**Mechanism Explanation:**
Paged KV separates the logical address space from physical storage.
`logical_block = floor(token_index / block_size)`
`offset = token_index % block_size`
`physical_block = block_table[logical_block]`

**Worked Example:**
Block size B = 16. A sequence has 34 tokens.
Logical blocks needed: ceil(34/16) = 3 blocks (Indices 0, 1, 2).
The Block Manager allocates physical blocks: `[7, 19, 4]`.
Token 33 (the 34th token): `logical_block = 2`. `offset = 1`. It resides at physical block `4`, offset `1`.

**Guided Practice:**
`block_size = 4`.
Current block table: `logical 0 → physical 7`, `logical 1 → physical 2`.
Free pool: `[4, 8, 11]`.
Sequence grows from 8 tokens to 9 tokens.
1. Determine whether a new logical block is required.
2. Allocate one physical block.
3. Update the block table.
4. Update the free pool.
5. State the allocator invariant after the transition.

**Feedback Contract:**
- *Expected Evidence*: Logical block 2 required. Physical block 4 (or 8/11) allocated. Table gets `logical 2 → 4`. Free pool drops to `[8, 11]`. Physical block 4 is no longer free.
- *Diagnostic Hint*: 8 tokens filled exactly how many 4-token blocks? What does the 9th token trigger?
- *Concept to Revisit*: Paged allocation thresholds and invariant bounds.

**Independent Practice:**
Proceed to **LAB B** to build the Minimal Paged KV Block Manager.

**Learning outcome:**
Explain paged KV management precisely, reason about block-size trade-offs, and track logical-to-physical state transitions.

*(Effort: 30m instruction, 15m practice, Lab B integration)*

---

### Lesson 3.4 — Sharing & Lifetime

**Engineering question:**
How do we safely share identical physical KV blocks across multiple distinct logical requests?

**Concepts & Definitions:**
- **Ownership & Reference Count**: Physical blocks carry a reference count indicating active logical ownership. `refcount(block) = number of active logical references to that block`.
- **Shared Block**: A physical block with `refcount > 1`.
- **Copy-on-Write (CoW)**: A shared mutable partial block must not be modified in-place if doing so would change state observed by another logical sequence. It must be duplicated.
- **Cache Residency vs Ownership**: Distinguish active ownership (`refcount > 0`) from cache residency. A block may have `refcount == 0` while still being cache-resident under a cache policy.

**Mechanism Explanation:**
Two sequences generated from the same prompt share the physical blocks. Their block tables point to the exact same physical indices.

**Worked Example:**
Prompt is 16 tokens (1 full block, block `9`). Seq A and Seq B share it. `refcount=2`.
Seq A generates token 17. Block `9` is full, so Seq A allocates a new block `14`. No CoW needed.
If the prompt was 14 tokens (a partial block), and Seq A appends token 15, modifying block `9` would corrupt Seq B. CoW is triggered: Block `9` is copied to `15`, Seq A appends to `15`, and Block `9`'s refcount drops to 1.

**Knowledge Check:**
1. Why is CoW unnecessary when appending to a full block?
2. Trace the refcount of a partial block shared by 3 parallel sampling sequences.

**Feedback Contract:**
- *Expected Evidence*: Explaining that full blocks are never mutated.
- *Diagnostic Hint*: Can a token be inserted into a block that is already full?
- *Concept to Revisit*: Block lifecycle invariants and mutability.

**Independent Practice:**
Implement CoW and invariant tests (refcount invariant, no free-and-allocated state overlap) in **LAB B**.

**Learning outcome:**
Reason about sharing, refcounts, and CoW mechanisms under strict invariants.

*(Effort: 30m instruction)*

---

### Lesson 3.5 — KV-Aware Scheduling & Memory Pressure

**Engineering question:**
How does the KV memory allocator interact with the request scheduler under severe memory pressure?

**Concepts & Definitions:**
- **Continuous Batching**: Iteration-level scheduling where new requests are admitted as soon as others finish or pause.
- **Admission**: The decision to allow a new request into the running batch.
- **Preemption / Swapping**: Pausing an active request and moving its KV blocks to CPU memory (or discarding for recomputation).
- **Scheduler Thrashing**: Spending all compute recomputing preempted caches rather than generating new tokens.

**Mechanism Explanation:**
The scheduler cannot decide feasibility from request count alone; it needs allocator state and projected demand. If active sequences exhaust allocatable blocks, a runtime may delay work, reject/admit differently, evict reusable state, preempt and recompute, swap/offload, or fail allocation. Which policy runs and its TTFT/TPOT effect are runtime- and workload-specific and belong to Module 04; this lesson focuses on the allocator signals and state transitions exposed at that boundary.

**Guided Practice:**
`free_blocks = 10`.
Running: `A` (expected growth = +4 blocks), `B` (expected growth = +8 blocks).
Queued: `C` (initial requirement = 6 blocks).
Compare:
1. Admit C now.
2. Delay C.
3. Preempt one running request.
Predict the TTFT effect, TPOT effect, memory-pressure risk, and recomputation cost for each scenario. What additional evidence is required before declaring the admission unsafe?

**Feedback Contract:**
- *Expected Evidence*: Demonstrating that admitting C consumes 6 blocks, leaving 4 free, creating high projected memory-pressure risk unless requests finish, blocks are reclaimed, or scheduling intervenes. Delaying C hurts TTFT but protects TPOT.
- *Diagnostic Hint*: What happens to A and B on the next few iterations if C takes 6 of the 10 free blocks?
- *Concept to Revisit*: KV growth uncertainty and scheduling risk.

**Independent Practice:**
Diagnose scheduler thrashing in the **Incident Scenario**.

**Learning outcome:**
Analyze scheduler-memory interaction and evaluate admission versus preemption trade-offs.

*(Effort: 30m instruction, 15m practice, Incident integration)*

---

### Lesson 3.6 — Prefix Cache, Radix & Eviction

**Engineering question:**
How do we persist and match previously computed KV blocks for future, unconnected requests?

**Concepts & Definitions:**
- **Cache Identity**: Two prefixes are safely reusable only if the runtime can establish that their cached KV state is equivalent under the relevant execution context (e.g., token sequence, model version, adapter, position semantics). Token-prefix hashing/tree lookup is an IMPLEMENTATION STRATEGY for identifying reusable state, not the semantic definition of reuse.
- **Eviction**: Reclaiming cache-resident KV according to a policy. LRU is one possible policy.
- **Lifecycle Transition**: `refcount == 0` means no active logical sequence currently references the block. Depending on policy, it MAY remain resident/evictable, be reclaimed, or participate in another cache policy. `refcount == 0` does NOT mean it must immediately be freed, nor does it mean it must remain cached.

**Mechanism Explanation:**
When a new request arrives, the scheduler searches the index for a safe match. If found, the scheduler initializes the request's block table with the matching blocks and increments their refcounts.

**Worked Example:**
Under a 16-token full-block reuse policy, System Prompt A has a reusable 96-token prefix mapped to physical blocks `[10...15]` plus a partial tail that is not indexed yet.
Request 1 finishes. Blocks `[10...15]` drop to `refcount=0`. The runtime policy leaves them indexed but physically reusable as eviction candidates.
Request 2 arrives with the same validated execution identity. The index finds `[10...15]`, removes them from the free/eviction queue, increments their reference counts, and skips recomputation of the 96-token full-block prefix. The tail behavior remains runtime-specific.

**Knowledge Check:**
1. If two requests have the exact same token prefix, does that guarantee their KV caches are mathematically identical? Name one context variable that could invalidate reuse.

**Feedback Contract:**
- *Expected Evidence*: Mentioning position semantics, adapters (LoRA), or model version mismatches.
- *Diagnostic Hint*: What if the exact same prompt is passed to two completely different model architectures or checkpoints?
- *Concept to Revisit*: Semantic Cache Identity.

**Independent Practice:**
Proceed to **LAB C** to simulate Prefix Sharing.

**Learning outcome:**
Model prefix-cache effectiveness and diagnose caching semantics.

*(Effort: 30m instruction, Lab C integration)*

---

### Lesson 3.7 — Cache-Aware Routing

**Engineering question:**
How does a multi-node load balancer know which worker holds the KV cache for a specific prompt?

**Concepts & Definitions:**
- **Cache Locality**: The probability a request routes to a worker possessing its prefix.
- **Load Skew**: When one worker receives disproportionately more requests.

**Mechanism Explanation:**
Pure round-robin load balancing often reduces cache locality, while perfect cache-aware routing can create severe load skew. The router tracks a heuristic view of downstream worker cache state to balance these concerns.

**Knowledge Check:**
1. Why might perfectly cache-aware routing degrade system goodput for a highly skewed workload?

**Feedback Contract:**
- *Expected Evidence*: Mention of worker saturation / bottlenecking.
- *Diagnostic Hint*: If 99% of requests use Prefix X, and Prefix X is only on Worker 1, what happens to Worker 2?
- *Concept to Revisit*: Load Skew.

**Independent Practice:**
Implement multi-worker routing simulation in **LAB C**.

**Learning outcome:**
Reason about routing/locality trade-offs.

*(Effort: 20m instruction, Lab C integration)*

---

### Lesson 3.8 — KV Quantization and Lossy Retention

**Engineering question:**
How does reducing the precision of the KV cache impact memory capacity, bandwidth, and generation quality?

**Concepts & Definitions:**
- **KV Quantization**: Storing physical KV blocks in FP8/INT8/INT4 rather than native FP16/BF16.
- **Scale / Quantization Metadata**: Extra scaling factors stored alongside the quantized blocks.

**Quantitative Model / Derivation:**
FP16 → FP8 approximately halves the NOMINAL KV PAYLOAD BYTES PER ELEMENT. This provides an analytical upper bound of ~2× KV capacity. It does NOT automatically imply 2× total throughput or exactly half measured HBM traffic, as this depends on metadata layout, kernel dequantization overhead, and model quality regressions.

Token-selective eviction is a different mechanism: it changes which positions attention can use and is therefore lossy unless the model/attention semantics already specify that window. Policies such as H2O provide important research evidence, but their quality and speed results remain model-, task-, kernel-, and workload-specific.

**Knowledge Check:**
1. Why doesn't INT8 quantization automatically double request concurrency?

**Feedback Contract:**
- *Expected Evidence*: Mentions of weights memory, context lengths, metadata overhead, or dequantization bottlenecks.
- *Diagnostic Hint*: Does quantizing the KV cache shrink the model weights?
- *Concept to Revisit*: Nominal vs System-level payload reduction.

**Independent Practice:**
Quantization trade-offs are evaluated in **LAB D**.

**Learning outcome:**
Evaluate quantization trade-offs explicitly based on empirical validation.

*(Effort: 30m instruction, Lab D integration)*

---

### Lesson 3.9 — KV Offloading & Memory Tiering

**Engineering question:**
When is it viable to move KV blocks off the GPU and into host memory?

**Concepts & Definitions:**
- **Memory Tiering**: HBM (fast/small) ↔ Pinned Host Memory (CPU DRAM, medium) ↔ NVMe (slow/large).
- **Pinned Host Memory**: Page-locked CPU memory enabling DMA-capable transfer paths. It avoids pageable-memory staging constraints but does NOT mean zero CPU orchestration/synchronization.

**Quantitative Model / Derivation:**
`transfer_serialization_time >= KV_payload_bytes / measured_effective_transfer_bandwidth`. Setup, synchronization, contention, conversion, and non-overlapped critical-path work add to completion time.

**Worked Example:**
Assume the 0.5 MiB/token configuration from Lesson 3.1.
If a 4096-token KV cache is offloaded, `KV_payload_bytes = 4096 * 0.5 MiB = 2,048 MiB = 2 GiB`.
For this exercise, assume measured effective host↔device transfer bandwidth is 50 GiB/s.
The ideal serialization lower bound for fetching 2 GiB over this link is `2 GiB / 50 GiB/s = 0.04 seconds (40 ms)`.
If restore is required before the first resumed computation, it adds at least 40 ms to that request's critical path before setup and contention. It is not automatically a per-output-token TPOT charge; placement and overlap determine which request metric moves.

**Knowledge Check:**
1. Calculate the ideal transfer time for a 16,384 token prompt under the exact same assumptions.

**Feedback Contract:**
- *Expected Evidence*: 16384 * 0.5 MiB = 8192 MiB = 8 GiB. 8 GiB / 50 GiB/s = 160 ms.
- *Diagnostic Hint*: Ensure units align before dividing by bandwidth.
- *Concept to Revisit*: Analytical transfer time.

**Independent Practice:**
Tiering latency is modeled in **LAB D**.

**Learning outcome:**
Evaluate offloading viability by quantitatively reasoning about host↔device transfer bandwidth.

*(Effort: 30m instruction, Lab D integration)*

---

### Lesson 3.10 — Production Source Trace and Cross-Instance KV

**Engineering question:**
How does a current runtime connect scheduler feasibility, physical blocks, and prefix-cache lifecycle, and what changes when KV crosses an instance boundary?

At vLLM commit `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`, the verified V1 path is:

```text
Scheduler.schedule
  → KVCacheManager.get_computed_blocks        # optional full-block prefix hit
  → KVCacheManager.allocate_slots
      → coordinator.get_num_blocks_to_allocate
      → headroom / watermark feasibility check
      → coordinator.allocate_new_computed_blocks
      → coordinator.allocate_new_blocks
          → BlockPool.get_new_blocks
  → execution consumes returned block IDs

request completion / release
  → KVCacheManager.free
  → coordinator.free
  → BlockPool.free_blocks
      → refcount update
      → non-cached blocks become early reuse candidates
      → cached zero-ref blocks become later eviction candidates
```

The same revision hashes only full blocks for prefix reuse. The chain includes parent-block hash, token IDs, and optional extra keys for LoRA, multimodal inputs, cache salt, and prompt embeddings. This is a current implementation contract, not a universal proof that token equality alone implies KV equivalence.

For cross-instance reuse or prefill/decode disaggregation, add ownership/validity metadata and a transfer path. The screening lower bound is

`T_transfer >= KV_payload_bytes / measured_effective_interconnect_bandwidth`.

Actual critical-path cost includes setup, contention, synchronization, layout conversion, and only the non-overlapped portion. The transport is not universally NCCL. Placement, routing, and distributed transport specialization belong to Module 20.

**Knowledge Check:**
1. Why can a vLLM block be both in the free queue and still indexed as a cached eviction candidate?
2. Which source-level extra keys prevent two token-identical requests with different execution state from sharing a block?

**Feedback Contract:**
- *Expected Evidence*: Distinguishes active ownership from reusable residency and identifies LoRA/multimodal/cache-salt/prompt-embedding keys.
- *Diagnostic Hint*: Read `BlockPool.touch`, `BlockPool.free_blocks`, and `generate_block_hash_extra_keys` at the pinned revision.
- *Concept to Revisit*: Runtime-specific lifecycle versus general mechanism.

**Independent Practice:**
Prefill→decode transfer latency modeled in **LAB D**.

**Learning outcome:**
Trace a current allocator/cache implementation without generalizing its objects or policies to every runtime.

*(Effort: 30m instruction, Lab D integration)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- Kwon et al. (SOSP 2023), *Efficient Memory Management for Large Language Model Serving with PagedAttention* — block tables, non-contiguous physical blocks, sharing, and paper-era evaluation.
- Zheng et al. (NeurIPS 2024), *SGLang: Efficient Execution of Structured Language Model Programs* — RadixAttention and structured-program prefix reuse.

**WORKLOAD-DEPENDENT EXTENSIONS**

- Liu et al. (ICML 2024), *KIVI* — asymmetric low-bit KV quantization; empirical results are configuration-specific.
- Zhang et al. (NeurIPS 2023), *H2O* — lossy attention-state retention based on recent/heavy-hitter tokens.
- Current TensorRT-LLM KV-cache documentation — paged/contiguous caches, reuse, prioritized eviction, offload, events, and distinct managers for hybrid layouts.

**CURRENT SOURCE SNAPSHOT**

- vLLM `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`, verified 2026-09-25:
  - `vllm/v1/core/sched/scheduler.py`: `Scheduler.schedule`;
  - `vllm/v1/core/kv_cache_manager.py`: `KVCacheManager.get_computed_blocks`, `allocate_slots`, `free`;
  - `vllm/v1/core/block_pool.py`: `BlockPool.get_new_blocks`, `touch`, `free_blocks`, `_maybe_evict_cached_block`;
  - `vllm/v1/core/kv_cache_utils.py`: `generate_block_hash_extra_keys`, `hash_block_tokens`, `generate_block_hashes`.

**Currentness classification**

- **REFERENCE / BASELINE**: exact KV reuse and logical payload accounting.
- **CURRENT COMMON PATTERN, NOT UNIVERSAL**: block/paged pools with dynamic per-request assignment.
- **WORKLOAD-DEPENDENT**: prefix/radix reuse, block size, eviction priority, cache-aware routing, and offload.
- **FRONTIER / ARCHITECTURE-SPECIFIC**: hybrid attention/state managers, low-bit/cold-page representations, token-selective retention, and cross-instance connectors.
- **LEGACY**: current claims based on removed `vllm/core/block_manager.py` or assuming the original 2023 vLLM architecture still defines current V1 internals.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

For every major experiment, follow the strict PREDICT → MEASURE loop:
1. Analytical prediction
2. Assumptions
3. Expected result
4. Instrumentation
5. Actual observation
6. Discrepancy analysis

### LAB A — KV Analytical Model + Allocation Simulator
- **Objective**: Implement a contiguous allocator simulator and expose fragmentation effects.
- **Alignment**: Explicitly exercises Lessons 3.1 and 3.2.
- **Effort Estimate**: 2h implementation, 1h experiments (3h total).

### LAB B — Minimal Paged KV Block Manager
- **Objective**: Implement a physical block pool and logical block table.
- **Action**: Implement allocation, growth, reference counting, and a CoW variant for shared partial blocks. Treat CoW as a reference mechanism from PagedAttention, not a claim that every current runtime shares partial blocks this way. Add invariant tests for ownership, free-pool membership, isolation, and stale metadata.
- **Alignment**: Explicitly exercises Lessons 3.3 and 3.4.
- **Effort Estimate**: 3h implementation, 1h analysis (4h total).

### LAB C — Prefix Sharing & Multi-Worker Routing
- **Objective**: Simulate Prefix tree lifecycles, cache identity policies, LRU eviction, and cache-aware routing.
- **Action**: Compare policies (round-robin vs prefix-affinity). Measure hit rate, load skew, and queue latency. Require repeated runs, p50/p95/p99 latency analysis, and variance measurement to avoid single-run conclusions (Statistical exercise).
- **Alignment**: Explicitly exercises Lessons 3.6 and 3.7.
- **Effort Estimate**: 2h simulation, 1h analysis (3h total).

### LAB D — Compression, Tiering, & Disaggregation Bounds
- **Objective**: Model latency constraints of tiering, quantization, and disaggregated transfers.
- **Action**: Compare exact paging, representation quantization, lossy token retention, and offload as distinct mechanisms. Extend analytical models to include simplified prefill→decode KV transfers. Reason about payload, metadata, quality, overlap, and critical-path impact without inventing a transport.
- **Alignment**: Explicitly exercises Lessons 3.8, 3.9, and 3.10.
- **Effort Estimate**: 2h modeling, 1h report (3h total).

### LAB E — Pinned vLLM V1 Lifecycle Trace
- **Objective**: Connect scheduler feasibility, prefix lookup, block allocation, release, and cached eviction candidates in a current source revision.
- **Action**: Trace the exact files and symbols in Lesson 3.10, then run a small workload with prefix caching enabled and disabled. Record cache groups/coordinator, block IDs, allocatable versus cached-evictable blocks, hits in reusable tokens, and any allocation refusal. Mark unexecuted branches `TODO_VERIFY`.
- **Falsification**: Find one behavior in the minimal simulator that is not valid for the pinned runtime, such as treating every free-queue block as semantically empty.
- **Effort Estimate**: 2h source trace, 1h instrumented comparison (3h total).

## 07 Break / Incident Scenarios

### Scenario A: The Undiagnosed Latency Collapse
- **Incident Symptoms**: Traffic patterns shift. Admission failures rise significantly. The reported "free capacity" metrics appear high, yet the system refuses to admit new requests. The TPOT latency distribution heavily tails. (Exercises Lesson 3.5: Scheduling / Memory Pressure).
- **Task**:
  - Formulate >= 3 plausible hypotheses for the failure.
  - Rank hypotheses based on system constraints.
  - Identify missing evidence/metrics.
  - Design a discriminating experiment.
  - Diagnose the root cause.
  - Intervene and re-measure.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment
**Architecture Transfer Problem**:
You are serving a massive Mixture-of-Experts (MoE) model where KV cache footprint matches dense models, but parameter weights consume 80% of your HBM. You have strict Time-Per-Output-Token (TPOT) latency SLOs. The workload is highly concurrent, conversational, with little prefix locality.

You must design an architecture that maximizes concurrency while meeting SLOs.
The learner should:
1. Construct a quantitative workload model.
2. Predict the dominant KV bottleneck.
3. Generate at least three technically distinct interventions.
4. Derive expected effect and state assumptions for each.
5. Identify required evidence and compare trade-offs.
6. Reject inappropriate interventions based on evidence (e.g., if you propose prefix caching, justify it against the low prefix locality).
7. Defend a final architecture decision.
8. State uncertainty and rollback/reversal conditions.

## 09 Required Evidence & Rubric
Evidence must demonstrate the following. Submissions are evaluated on engineering capability.

### Required Artifact: Production Source Trace
Trace the pinned vLLM V1 path beginning at `Scheduler.schedule`, through `KVCacheManager.allocate_slots`, coordinator allocation, and `BlockPool.get_new_blocks`; then trace prefix lookup/hash and release/eviction paths. Do not substitute the removed legacy `vllm/core/block_manager.py` architecture.
Evidence must include: repository, commit `25b0add7b8a1c944d5c4e364f2de6aa82497a2ad`, verification date, exact files and symbols, execution path, mapping from implementation objects to Module 03 concepts, selected coordinator/cache groups at runtime, and one observation where the production implementation differs from the learner's minimal Block Manager. Use `TODO_VERIFY` for any dynamic path not executed.

### Rubric Dimensions
- **Mechanistic Reasoning**: *Insufficient* describes surface behavior. *Competent* explains causal mechanisms. *Strong* connects mechanisms across boundaries (e.g., scheduler + block manager).
- **Quantitative Reasoning**: *Insufficient* uses formulas without assumptions. *Competent* derives estimates with explicit assumptions. *Strong* predicts behavior, measures it, and explains discrepancies.
- **Experiment Design**: *Insufficient* runs benchmarks without hypothesis. *Competent* controls variables and tests a hypothesis. *Strong* designs discriminating experiments between competing hypotheses.
- **Failure Diagnosis**: *Insufficient* guesses a root cause. *Competent* forms multiple hypotheses. *Strong* ranks hypotheses by information gain and systematically eliminates them.
- **Falsification**: *Insufficient* only seeks confirming evidence. *Competent* tests counterexamples. *Strong* constructs adversarial conditions capable of disproving the design.
- **Architecture Decision**: *Insufficient* selects a technology. *Competent* explains trade-offs using evidence. *Strong* states uncertainty, operational risks, rollback strategies, and evidence that would reverse the decision.
- **Source Reasoning**: *Insufficient* quotes implementation details. *Competent* traces relevant execution paths. *Strong* connects implementation to observed system behavior.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| KV Memory Footprint | 3.1 | KC + Lab A | Mastery | Derivation/Report |
| Fragmentation Reasoning | 3.2 | KC + Lab A | Incident | Experiment Trace |
| Paged KV Allocation | 3.3 | GP + Lab B | Lab B | Implementation/Tests |
| Refcount / CoW | 3.4 | KC + Lab B | Lab B | Implementation/Tests |
| Scheduling Pressure | 3.5 | GP + Incident | Incident | Diagnosis/Report |
| Prefix Caching / Radix | 3.6 | KC + Lab C | Lab C | Simulation/Benchmark |
| Cache-Aware Routing | 3.7 | KC + Lab C | Lab C / Mastery | Routing Experiment |
| Quantization / Lossy Retention | 3.8 | KC + Lab D | Lab D | Quantitative + Quality Report |
| Offloading Latency | 3.9 | KC + Lab D | Lab D | Transfer Model |
| Production Source / Cross-Instance KV | 3.10 | KC + Lab D/E | Mastery | Pinned Source Trace |

*(Key: KC = Knowledge Check, GP = Guided Practice)*

## 11 Exit Criteria & Module Wrap-Up
A learner completing Module 03 should be able to derive KV memory footprint, explain paged block tables, diagnose scheduler memory thrashing, evaluate quantization/offloading mathematically, trace production code, and defend an architecture design.

**Module Wrap-Up (Final Mental Model Reconstruction):**
- MODEL ARCHITECTURE → KV BYTES PER TOKEN → REQUEST LENGTH / CONCURRENCY → LOGICAL KV DEMAND → PHYSICAL KV MANAGEMENT → ALLOCATOR / SHARING / CACHE → SCHEDULER → ROUTING / LOCALITY → HARDWARE CAPACITY & BANDWIDTH → LATENCY / GOODPUT / COST.
- When KV becomes limiting, possible interventions may include: admission/scheduling changes, block-size/allocation policy changes, prefix reuse, eviction changes, routing/locality, KV representation/quantization, tiering/offloading, or placement/disaggregation.
- The correct intervention depends entirely on evidence. The final mental model is: PREDICT → OBSERVE → DIAGNOSE → INTERVENE → FALSIFY.

## 12 Competency Targets
```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

# Module 11 — Context & Memory Engineering

## 00 Why This Module Exists

A long context window and durable memory are different systems. The context window is the finite serialized input available to one inference. Application memory is persisted state that must be written, updated, selected, authorized, compacted, and reintroduced into a later context. Neither is reliable merely because it is large.

```text
events/messages/tool results
        |
        +--> immutable history -----------+
        +--> canonical structured state --+--> read/select policy
        +--> summaries/reflections --------+         |
        +--> episodic/semantic records ----+         v
                                            context allocator
system + tools + current input + selected memory + evidence + output reserve
                                            |
                                            v
                                     model inference
                                            |
                              outcome + memory write decision
```

This module owns model-aware context budgeting, effective-context testing, memory types, write/read/update/delete policy, summarization and compression, and memory-stage diagnosis. Module 03 owns KV-cache internals, Module 09 retrieval algorithms, Module 10 RAG evidence/context assembly, Module 12 agent-loop control, and Module 14 durable execution/checkpoint semantics.

**Research cutoff:** 2026-09-27.

**Module Orientation**

- **Engineering problem:** retain and expose the minimum sufficient, valid, authorized state for the next decision under finite tokens, latency, cost, privacy, and correctness constraints.
- **What you will do:** account for serialized context; map effective-context limits; design typed/versioned memory; implement write/read/correct/delete and conflict policies; compare compaction strategies; inject stale, poisoned, fragmented, and cross-tenant state; trace current helpers; and diagnose the earliest failing memory stage.
- **Environment:** Python 3.10+ with the target tokenizer or server usage accounting, a versioned state store/search fixture, and a reproducible model endpoint. Pin serialization protocol, model, prompt, memory schema, selectors, summarizers, and evaluator revisions.
- **Evidence rule:** distinguish source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**). Cognitive terms are software metaphors unless an implementation contract defines them.

## 01 Baseline Assumptions

- Module 00: measurands, experimental units, paired comparisons, and falsification.
- Module 01: tokenization, attention, positional mechanisms, and model input shape.
- Module 03: physical KV state is not application memory.
- Module 07: uncertainty, abstention, and behavioral regression.
- Modules 08–10: versioned data, retrieval contracts, RAG context assembly, citations, and freshness.
- Module 04: latency distributions, goodput, and capacity effects of longer prompts.

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
  security: REQUIRED
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 18h
  assessment: 3h
  source_trace: 2h
  total: 31h
```

The learner must be able to account for every context token; reserve output safely; measure effective context rather than advertised length; design typed, versioned memory; implement selective write/read/update/delete paths; quantify compaction loss and break-even; inject stale, poisoned, contradictory, and fragmented records; and attribute failure across memory stages.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
                 application state across requests
       +----------------+----------------+----------------+
       |                |                |                |
  raw events       canonical facts    summaries      procedures
  immutable        versioned state     derived/lossy  reviewed/reusable
       \                |                |                /
        +---------------+----------------+---------------+
                            |
                  eligibility + retrieval
                            |
                 context-budget allocator
                            |
        instructions | tools | recent turns | memory | evidence
                            |
                     output reserve
```

Keep these boundaries explicit:

1. **Stored:** was the needed state correctly extracted and persisted?
2. **Valid:** is it current, authorized, and not superseded?
3. **Selected:** did the read policy retrieve it for this request?
4. **Placed:** did it survive context budgeting and ordering?
5. **Used:** did the model apply it correctly?

## 04 Lessons

### Lesson 11.1 — Context Accounting and Admission

**Engineering Question:**
Will the exact serialized request and reserved output fit without breaking protocol invariants?

**Concepts & Definitions:**

For a declared model, tokenizer, message protocol, and endpoint:

$$
T_{system}+T_{tools}+T_{history}+T_{evidence}+T_{state}+T_{input}
+T_{framing}+T_{output\ reserve}\le T_{limit}.
$$

The inequality is an admission invariant, not a quality guarantee. Tool schemas, role/name fields, tool-call IDs, images, hidden delimiters, and provider framing can consume tokens even when a UI hides them. Input/output limits may be combined or separate. Pin the interface version and record the serialized request when policy permits.

Use the target tokenizer or server-reported usage for exact accounting where possible. If a hot path uses a heuristic, measure signed and absolute error by language, modality, tool shape, and message mix; reserve a calibrated margin. Define priority classes and atomic groups so truncation never leaves a tool result without its call or strips the instruction that interprets state.

**Worked Example:** Sum measured tokens for system, tools, history, evidence, state, input, framing, and reserve. If the total exceeds the declared limit, reject or compact before submission; silently reducing reserve changes the output contract.

**Knowledge Check:** Why can visible text undercount serialized tool/message tokens?

**Guided Practice:** Compare heuristic and exact/server counts across languages and tool schemas; test exact-boundary, over-limit, and atomic tool-call cases.

**Feedback Contract:** Expected evidence is estimator-error distribution, pinned protocol/tokenizer, category ledger, invariant tests, and fallback. A common failure is truncating individual messages without preserving call/result pairs.

**Learning Outcome:** Implement admission with explicit category budgets, valid-message invariants, fallback, and output reserve.

*(Effort: 45m instruction, 30m practice)*

### Lesson 11.2 — Advertised Window vs Effective Context

**Engineering Question:**
Can the model reliably use the required information across position, noise, multiplicity, and task complexity?

**Concepts & Definitions:**

Maximum accepted length asks “can the request be processed?” Effective context asks “can the model reliably use the required information for this task?” Measure a phase surface across:

- input length and evidence position;
- relevant-item count, distractor count, and similarity;
- exact lookup, multi-hop tracing, aggregation, long-dialogue, and code/state tasks;
- prompt/template and model revision;
- answer format, reasoning budget, latency, and cost.

RULER shows that simple needle retrieval can conceal failures on multi-needle, tracing, aggregation, and QA tasks as length grows. Lost in the Middle shows position sensitivity in evaluated models. Neither implies a universal degradation curve. A model upgrade can improve one slice and regress another.

**Worked Example:** A model can accept 128k tokens and retrieve one unique string near the end while failing multi-fact aggregation at a shorter length. Acceptance and one needle task therefore do not define the same envelope.

**Knowledge Check:** Which crossed variables are required to distinguish length from position effects?

**Guided Practice:** Sweep length, position, distractors, relevant count, hops, and model/prompt revision with paired seeds and no-answer controls.

**Feedback Contract:** Expected evidence is a multidimensional quality/latency/cost surface with uncertainty. A common failure is reporting maximum accepted length as usable memory.

**Learning Outcome:** Publish workload-specific effective-context envelopes, not a single model-card number.

*(Effort: 40m instruction, 30m practice)*

### Lesson 11.3 — Memory Types and Source of Truth

**Engineering Question:**
Which record is authoritative state, which is immutable evidence, and which is a lossy derived view?

**Concepts & Definitions:**

Use types to encode different semantics:

- **immutable events:** what happened, with source, sequence, and time;
- **working/recent context:** a bounded recency window for the current task;
- **canonical state:** current preferences, decisions, entities, constraints, and versions;
- **episodic records:** selected prior interactions or outcomes;
- **semantic records:** durable facts with source and validity;
- **procedural artifacts:** reviewed strategies, plans, examples, or code;
- **summaries/reflections:** derived, lossy views that never silently replace raw provenance.

MemGPT is a reference for moving information between bounded in-context and external tiers. Generative Agents is a reference for an event stream, retrieval, and reflection. These are mechanism families, not proof of infinite, lossless, safe, or human-like memory.

**Worked Example:** A summary saying “user prefers X” is derived from an event and can become stale after correction. Canonical state should supersede it while raw events preserve the audit trail.

**Knowledge Check:** Why must summaries retain derived-from lineage?

**Guided Practice:** Classify an event log, current preference, prior episode, procedure, and summary; define owner, authority, times, supersession, and deletion behavior.

**Feedback Contract:** Expected evidence is typed schemas and explicit source-of-truth rules. A common failure is allowing a reflection to silently overwrite canonical state.

**Learning Outcome:** Define record schemas, authority, ownership, valid/transaction time, and correction paths for every memory type.

*(Effort: 40m instruction, 25m practice)*

### Lesson 11.4 — Write, Read, Update, and Forget

**Engineering Question:**
How does memory converge under retries, concurrent corrections, expiry, deletion, and access constraints?

**Concepts & Definitions:**

A write policy asks whether an event is eligible, novel, durable, attributable, consented, and safe to retain. Store stable identity, source, confidence, privacy class, valid time, transaction time, TTL, supersession, and derived-from lineage. Append-only growth can retain prompt injection, transient mood, wrong tool output, or another user's data.

A read policy first applies hard access, validity, deletion, and task constraints; only then rank by workload-tested relevance, recency, importance, authority, confidence, and prior utility. Similarity alone does not prove applicability. Selected records need lineage in the request trace.

Concurrent or multi-writer updates need an explicit compare/version rule. Conditional writes, monotonic sequence or transaction versions, and a domain conflict policy make lost updates observable; last-writer-wins is valid only when its clock and product semantics are declared. Backfills, TTL expiry, tombstones, indexes, replicas, and caches must converge before physical garbage collection removes recovery evidence.

Forgetting is a capability: expiry, correction, user deletion, policy deletion, invalidation after failure, and capacity-based eviction differ. Tombstones and supersession must reach all indexes/caches. A 2025 study reports experience-following, error propagation, and misaligned replay in its evaluated agents—use this as a failure family, not a universal rate.

**Worked Example:** Two devices update one preference from the same base version. A conditional write can reject one stale update for reconciliation; an unconditional write may silently lose it. This is a state-policy choice, not a language-model judgment.

**Knowledge Check:** Why must access/validity filtering precede similarity ranking?

**Guided Practice:** Inject duplicate, reordered, concurrent, corrected, expired, deleted, and cross-tenant records; verify convergence across primary store, index, replica, cache, and context trace.

**Feedback Contract:** Expected evidence is version/conflict logs, lineage, authorization checks, tombstone/GC state, and convergence probes. A common failure is deleting only the primary record.

**Learning Outcome:** Survive duplicated/out-of-order writes, corrections, deletes, poisoning, and distribution shift without replaying invalid state.

*(Effort: 50m instruction, 35m practice)*

### Lesson 11.5 — Truncation, Summarization, and Compression

**Engineering Question:**
Which compaction policy preserves required semantics and lowers end-to-end cost for the target workload?

**Concepts & Definitions:**

Compaction options have different failure modes:

- sliding windows preserve recent verbatim turns but forget older dependencies;
- deterministic selection preserves chosen fields but fails on an incomplete schema;
- summaries save tokens but can omit qualifiers, time, negation, disagreement, and source;
- retrieval externalizes history but can miss or misrank it;
- learned/token compression adds another model, overhead, and corruption surface.

Treat summaries as versioned derived artifacts. Link each claim to source events, allow rebuild, and periodically compare with raw history. Recursive summarization can turn an interpretation into an apparent fact.

Define:

$$
r=\frac{T_{retained}}{T_{original}},\qquad
\Delta L_{e2e}=L_{compress}+L_{model,compressed}-L_{model,original}.
$$

Token reduction does not imply latency reduction. A 2026 study found operating regions where compression helped and regions where preprocessing erased the benefit. Measure quality, protocol validity, memory, queueing, and end-to-end latency at the target model/hardware/load.

**Worked Example:** A compressor that saves 500 ms of model time but adds 700 ms preprocessing has $\Delta L_{e2e}=+200$ ms and is slower at that operating point despite reducing tokens.

**Knowledge Check:** Why can recursive summary claims become false canonical facts?

**Guided Practice:** Compare full, windowed, structured, source-linked summary, retrieval, and learned compression across ratio, depth, hardware, and load.

**Feedback Contract:** Expected evidence is semantic preservation by slice, protocol validity, preprocessing/model/queue timing, cost, and raw replay. A common failure is reporting compression ratio as latency speedup.

**Learning Outcome:** Choose compaction from a semantic-loss and end-to-end frontier, with raw-state recovery.

*(Effort: 45m instruction, 30m practice)*

### Lesson 11.6 — Memory Evaluation and Diagnosis

**Engineering Question:**
At which stored → valid → selected → placed → used boundary did the required state disappear or become harmful?

**Concepts & Definitions:**

Static long-context recall is not a complete memory test. Include:

- write/extraction precision and recall at event time;
- persistence, correction, deletion, TTL, authorization, and version convergence;
- valid-record retrieval and false-memory selection at probe time;
- context placement/order and survival after compaction;
- utilization given the required record is present;
- downstream task outcome, calibration/abstention, privacy leakage, latency, and cost;
- long-range understanding, test-time experience reuse, and selective forgetting.

MemoryAgentBench explicitly expands evaluation toward incremental interaction and four competencies. IFCMemoryBench provides 2026 domain-specific evidence that topically relevant memory can still be incomplete or fragmented. Do not turn either benchmark into a universal leaderboard.

Diagnosis follows the earliest discriminated boundary:

```text
wrong response
 -> fact never written?
 -> written incorrectly or stale?
 -> valid fact not selected?
 -> selected but compacted/evicted/misordered?
 -> present but model ignored/misused it?
```

**Quantitative Model / Derivation:**
For each declared probe population, report stage opportunity conditionally: valid given stored, selected given valid, placed given selected, and correctly used given placed. Multiplying stage rates is justified only when the denominators and conditional chain match; retain raw counts and end-to-end success.

**Worked Example:** If a fact is valid and selected but absent from the serialized prompt, retrieval tuning cannot fix the placement loss. Replaying the same request with oracle placement discriminates the stage.

**Knowledge Check:** Which oracle replay separates selection failure from model non-use?

**Guided Practice:** Run baseline plus oracle stored-state, oracle selection, oracle placement, and memory-disabled replays on stale, fragmented, corrected, and poisoned cases.

**Feedback Contract:** Expected evidence is stage timestamps/identities, conditional counts, privacy/deletion checks, downstream outcome, and remeasurement. A common failure is one end-to-end score with no stage lineage.

**Learning Outcome:** Run oracle-stage replays and assign interventions to the failing stage, then remeasure.

*(Effort: 50m instruction, 30m practice)*

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [Lost in the Middle](https://arxiv.org/abs/2307.03172) — Liu et al. (TACL 2023).
- [RULER](https://arxiv.org/abs/2404.06654) — Hsieh et al. (COLM 2024).
- [MemGPT](https://arxiv.org/abs/2310.08560) — Packer et al. (2024 revision).
- [Generative Agents](https://arxiv.org/abs/2304.03442) — Park et al. (2023).

**CURRENT DEFAULT:** explicit context budgets/output reserve; model/protocol token accounting; valid-message boundaries; typed versioned records; selective authorized write/read/update/delete; recoverable summary provenance; stage-attributed evaluation.

**WORKLOAD-DEPENDENT:** recency window, memory taxonomy detail, scoring weights, summary cadence, token compressor, context order, TTL, reflection, and effective-context envelope.

**FRONTIER:** incremental memory-agent benchmarks, experience reuse, structured/graph/domain-linked memory, learned selective forgetting, and 2026 compression break-even modeling. No universal architecture or benchmark winner is established.

**LEGACY / INSUFFICIENT:** replay the entire transcript; keep the last N messages without token/protocol checks; store everything forever; retrieve by similarity alone; treat summaries as facts; equate context length with usable memory; report compression ratio as speedup.

**PRODUCTION SOURCE TRACE**

- Repository: `langchain-ai/langchain`
- Revision: `80b74090515a594cc8421fb2d82e98f4e256c29f`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `libs/core/langchain_core/messages/utils.py::{trim_messages,count_tokens_approximately}` and `libs/core/langchain_core/chat_history.py::InMemoryChatMessageHistory.{add_message,clear}`.
- Observed: trimming accepts first/last strategies, exact or approximate counters, partial-message behavior, and message-boundary controls; the approximate counter explicitly disclaims exact model counts; the in-memory history stores a process-local list.
- Scope: current pinned helpers, not a universal memory policy and not durable persistence.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Context Accountant and Admission Gate

- **Objective:** build a serializer-aware context ledger and admission gate with output reserve and message invariants.
- **Pre-Registered Hypothesis:** heuristic count error will vary by language/protocol shape, requiring measured safety margin rather than one universal constant.
- **Independent Variables:** counter, language, tool/JSON/image shape, protocol/tokenizer revision, input length, and reserve.
- **Dependent Variables:** signed/absolute count error, admission errors, protocol validity, fallback, latency, and unused margin.
- **Break & Falsify:** underestimate framing, orphan tool messages, over-reserve output, change tokenizer, and test exact boundaries.
- **Alignment:** Lesson 11.1.
- **Effort Estimate:** 4h.

### LAB B — Effective-Context Phase Surface

- **Objective:** measure a workload-specific effective-context surface beyond accepted-length and needle baselines.
- **Pre-Registered Hypothesis:** position, distractor, multiplicity, and task complexity will interact with length, so no one accepted-window number predicts all outcomes.
- **Independent Variables:** length, position, relevant/distractor count/similarity, facts/hops/aggregation, task, and model/prompt revision.
- **Dependent Variables:** accuracy/support, utilization, abstention, latency, tokens, cost, and protocol failures.
- **Break & Falsify:** include long dialogue, structured state, code, multi-fact aggregation, and no-answer cases.
- **Alignment:** Lesson 11.2.
- **Effort Estimate:** 4.5h.

### LAB C — Versioned Memory Lifecycle

- **Objective:** implement typed/versioned records and prove write/read/correct/delete/conflict convergence.
- **Pre-Registered Hypothesis:** similarity-only, unconditional-write, or primary-store-only deletion baselines will fail at least one validity, privacy, lost-update, or convergence test.
- **Independent Variables:** memory type, duplicate/order/concurrency, correction/delete/TTL, ownership, poison, and replica/cache state.
- **Dependent Variables:** write precision/recall, conflicts, convergence, valid selection, false-memory/utilization, privacy violations, and recovery.
- **Break & Falsify:** inject stale-base concurrent writes, cross-tenant facts, adversarial memory, fragmented knowledge, lagging indexes, and GC before convergence.
- **Alignment:** Lessons 11.3 and 11.4.
- **Effort Estimate:** 5h.

### LAB D — Compaction and Memory Regression

- **Objective:** compare compaction policies and attribute memory failure by stored/valid/selected/placed/used stage.
- **Pre-Registered Hypothesis:** no compaction method will preserve every semantic slice, and preprocessing/load will produce at least one point where fewer tokens do not reduce end-to-end latency.
- **Independent Variables:** method, ratio/depth, source age, dependency, protocol shape, hardware/load, and oracle-stage replay.
- **Dependent Variables:** semantic preservation, stage opportunity, downstream outcome, preprocessing/model/queue latency, memory, cost, privacy, and recovery.
- **Break & Falsify:** use negation, corrections, long-range dependencies, recursive summaries, tool protocols, poisoning, and loaded queues.
- **Alignment:** Lessons 11.5 and 11.6.
- **Effort Estimate:** 4.5h.

## 07 Break / Incident Scenarios

### Incident 11.1 — Helpful Memory Release Causes Repeated Wrong Actions

**Incident Symptoms:**
A memory release lowers average prompt tokens and improves repeat-task success, but some users see old preferences after correction, another tenant's detail appears in a response, tool-call histories become invalid after trimming, and one previously successful but wrong execution is repeatedly replayed.

**Diagnostic Protocol (Task):**
1. **Form Competing Hypotheses:** tokenizer/framing undercount, reserve exhaustion, invalid trimming, extraction error, ownership-filter failure, lost/out-of-order update, incomplete deletion/GC, summary drift, stale selection, experience replay, position loss, or model non-use.
2. **Identify Missing Evidence:** raw events, versions/owners/valid and transaction times, conflict/tombstone/index/cache state, compaction lineage, exact prompt, counter/version/error, selected order, protocol structure, reserve, model/prompt, and stage timings.
3. **Design Discriminating Measurements:** replay with oracle stored/valid state, oracle selection, oracle placement/uncompressed context, changed position, and memory disabled; probe every replica/cache.
4. **Rank Explanations:** locate the earliest stored → valid → selected → placed → used failure and retain interacting privacy/protocol causes.
5. **Intervene:** gate the route, repair ownership/version/conflict/delete propagation, rebuild summaries/indexes, change compaction/placement, or disable harmful experience reuse.
6. **Remeasure:** repeat stage opportunity, corrections/deletes, privacy, protocol validity, task utility, abstention, latency, cost, and rollback checks.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem — Multi-Session Technical Assistant

Design context and memory for a multi-session technical assistant that uses tools, honors concurrent corrections/deletion, remembers project decisions, and operates under strict context, latency, privacy, and cost constraints.

**Required Deliverables:**
1. serializer-aware category budgets, output reserve, admission, and protocol invariants;
2. effective-context phase surface across length, position, noise, multiplicity, and task complexity;
3. typed/versioned schemas with authority, ownership, valid/transaction time, lineage, and source of truth;
4. write/read/correct/conflict/forget policy with deletion, TTL, tombstone, index/cache convergence, and GC boundary;
5. poisoned, stale, fragmented, concurrent-update, protocol, and cross-tenant tests;
6. compaction semantic-loss and end-to-end performance frontier with raw recovery;
7. stored/valid/selected/placed/used metrics and oracle-stage replays;
8. current source trace with explicit non-durability/generalizability boundary;
9. canary, rollback, and Incident 11.1 diagnosis.

## 09 Required Evidence & Rubric

### Required Artifact: Context and Memory Lifecycle Trace

Submit an exact serialized request ledger, typed record/change history, conflict/delete convergence probes, selected-memory lineage, compaction comparison, oracle-stage diagnosis, source trace, and release/rollback decision.

### Rubric Dimensions

- **Budget:** *Insufficient* counts visible text only. *Competent* pins tokenizer/protocol/model and reserve. *Strong* measures estimator error, multimodal/tool framing, invariants, and fallback.
- **Effective Context:** *Insufficient* quotes window size. *Competent* crosses length/position/noise. *Strong* maps multiplicity/complexity/revisions with uncertainty and operational cost.
- **Memory Lifecycle:** *Insufficient* stores untyped text. *Competent* versions owner/time/TTL/update/delete. *Strong* proves concurrent conflict, index/cache convergence, GC safety, and lineage.
- **Selection/Compaction:** *Insufficient* uses similarity or token ratio alone. *Competent* applies hard validity/access and measures semantic loss. *Strong* includes recursive drift, protocol validity, preprocessing, load, and recovery.
- **Diagnosis:** *Insufficient* blames memory generally. *Competent* separates stages. *Strong* uses oracle replays to locate earliest failure and tests competing causes.
- **Operations:** *Insufficient* omits privacy/deletion. *Competent* demonstrates them. *Strong* includes poisoning, rollback, raw-event recovery, and scoped source behavior.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Context accounting and admission | 11.1 | LAB A | Incident / Mastery | Ledger and estimator-error report |
| Effective-context measurement | 11.2 | LAB B | Mastery | Length-position-complexity surface |
| Typed memory lifecycle | 11.3–11.4 | LAB C | Incident / Mastery | Version, delete, privacy, poison tests |
| Compaction and diagnosis | 11.5–11.6 | LAB D | Incident / Mastery | Semantic-loss/performance frontier |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner passes when they can account for serialized context and output reserve; distinguish accepted from effective length; reject human-memory analogies without software semantics; survive concurrent update/delete/TTL/privacy/index/cache and replay failures; prove summaries are recoverable derived state; identify compression break-even rather than assume it; trace current source without generalizing it; and locate failures across stored, valid, selected, placed, and used boundaries.

### Module Wrap-Up (Final Mental Model Reconstruction)

Context is scarce execution input; memory is a governed state system. Reliability comes from budgeting, typed/versioned records, explicit conflict and deletion convergence, selective authorized movement into context, recoverable compaction, and stage-level falsification—not from replaying more text.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

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

**Research cutoff:** 2026-09-26.

- **Engineering problem:** retain and expose the minimum sufficient, valid, authorized state for the next decision under finite tokens, latency, cost, privacy, and correctness constraints.
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
  falsification: REQUIRED
```

The learner must be able to account for every context token; reserve output safely; measure effective context rather than advertised length; design typed, versioned memory; implement selective write/read/update/delete paths; quantify compaction loss and break-even; inject stale, poisoned, contradictory, and fragmented records; and attribute failure across memory stages.

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

For a declared model, tokenizer, message protocol, and endpoint:

$$
T_{system}+T_{tools}+T_{history}+T_{evidence}+T_{state}+T_{input}
+T_{framing}+T_{output\ reserve}\le T_{limit}.
$$

The inequality is an admission invariant, not a quality guarantee. Tool schemas, role/name fields, tool-call IDs, images, hidden delimiters, and provider framing can consume tokens even when a UI hides them. Input/output limits may be combined or separate. Pin the interface version and record the serialized request when policy permits.

Use the target tokenizer or server-reported usage for exact accounting where possible. If a hot path uses a heuristic, measure signed and absolute error by language, modality, tool shape, and message mix; reserve a calibrated margin. Define priority classes and atomic groups so truncation never leaves a tool result without its call or strips the instruction that interprets state.

**Outcome:** implement admission with explicit category budgets, valid-message invariants, fallback, and output reserve.

### Lesson 11.2 — Advertised Window vs Effective Context

Maximum accepted length asks “can the request be processed?” Effective context asks “can the model reliably use the required information for this task?” Measure a phase surface across:

- input length and evidence position;
- relevant-item count, distractor count, and similarity;
- exact lookup, multi-hop tracing, aggregation, long-dialogue, and code/state tasks;
- prompt/template and model revision;
- answer format, reasoning budget, latency, and cost.

RULER shows that simple needle retrieval can conceal failures on multi-needle, tracing, aggregation, and QA tasks as length grows. Lost in the Middle shows position sensitivity in evaluated models. Neither implies a universal degradation curve. A model upgrade can improve one slice and regress another.

**Outcome:** publish workload-specific effective-context envelopes, not a single model-card number.

### Lesson 11.3 — Memory Types and Source of Truth

Use types to encode different semantics:

- **immutable events:** what happened, with source, sequence, and time;
- **working/recent context:** a bounded recency window for the current task;
- **canonical state:** current preferences, decisions, entities, constraints, and versions;
- **episodic records:** selected prior interactions or outcomes;
- **semantic records:** durable facts with source and validity;
- **procedural artifacts:** reviewed strategies, plans, examples, or code;
- **summaries/reflections:** derived, lossy views that never silently replace raw provenance.

MemGPT is a reference for moving information between bounded in-context and external tiers. Generative Agents is a reference for an event stream, retrieval, and reflection. These are mechanism families, not proof of infinite, lossless, safe, or human-like memory.

**Outcome:** define record schemas, authority, ownership, valid/transaction time, and correction paths for every memory type.

### Lesson 11.4 — Write, Read, Update, and Forget

A write policy asks whether an event is eligible, novel, durable, attributable, consented, and safe to retain. Store stable identity, source, confidence, privacy class, valid time, transaction time, TTL, supersession, and derived-from lineage. Append-only growth can retain prompt injection, transient mood, wrong tool output, or another user's data.

A read policy first applies hard access, validity, deletion, and task constraints; only then rank by workload-tested relevance, recency, importance, authority, confidence, and prior utility. Similarity alone does not prove applicability. Selected records need lineage in the request trace.

Forgetting is a capability: expiry, correction, user deletion, policy deletion, invalidation after failure, and capacity-based eviction differ. Tombstones and supersession must reach all indexes/caches. A 2025 study reports experience-following, error propagation, and misaligned replay in its evaluated agents—use this as a failure family, not a universal rate.

**Outcome:** survive duplicated/out-of-order writes, corrections, deletes, poisoning, and distribution shift without replaying invalid state.

### Lesson 11.5 — Truncation, Summarization, and Compression

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

**Outcome:** choose compaction from a semantic-loss and end-to-end frontier, with raw-state recovery.

### Lesson 11.6 — Memory Evaluation and Diagnosis

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

**Outcome:** run oracle-stage replays and assign interventions to the failing stage, then remeasure.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Context Accountant and Admission Gate

- Build a serializer-aware ledger for system messages, tools, history, evidence, state, current input, framing estimate, and output reserve.
- Compare heuristic counts with target tokenizer/server usage across languages, JSON/tool schemas, images, and long tool results.
- Break with underestimated framing, orphan tool messages, overlong output requests, tokenizer revisions, and boundary-exact inputs.
- Artifact: estimator-error distribution, admission policy, valid-message invariants, fallbacks, and source trace.

### LAB B — Effective-Context Phase Surface

- Sweep length, evidence position, relevant/distractor count, semantic similarity, number of required facts, hops, aggregation, and prompt/model revision.
- Include vanilla needle tasks only as a baseline; add long dialogue, structured state, code, and no-answer cases.
- Measure accuracy/support, utilization, abstention, latency, tokens, and cost with paired seeds.
- Artifact: effective-context envelope and counterexamples to a single context-length claim.

### LAB C — Versioned Memory Lifecycle

- Implement typed event, canonical state, episodic, semantic, procedural, and summary records with ownership, valid time, version, confidence, TTL, supersession, tombstone, and lineage.
- Inject duplicates, reordering, corrections, deletes, private cross-tenant facts, adversarial memories, stale successes, and fragmented multi-session knowledge.
- Measure write precision/recall, convergence, valid retrieval, false-memory rate, utilization, privacy violations, and recovery.

### LAB D — Compaction and Memory Regression

- Compare full context, recent-window, structured extraction, source-linked summary, retrieval, and learned compression at matched tasks.
- Sweep compression ratio, summarization depth, source age, long-range dependencies, protocol structures, hardware, and load.
- Measure semantic preservation by slice, end-to-end latency including preprocessing, memory, cost, queueing, and downstream outcome.
- Falsify “shorter is faster” and “summary preserves what matters”; retain raw-event replay and rollback.

## 07 Break / Incident Scenarios

### Incident 11.1 — Helpful Memory Release Causes Repeated Wrong Actions

A memory release lowers average prompt tokens and improves repeat-task success, but some users see old preferences after correction, another tenant's detail appears in a response, tool-call histories become invalid after trimming, and one previously successful but wrong execution is repeatedly replayed.

Competing explanations include tokenizer/framing undercount, output-reserve exhaustion, invalid message trimming, write extraction error, missing ownership filter, out-of-order overwrite, incomplete deletion, summary drift, stale/high-similarity selection, experience-following, context-position loss, or model non-use.

Recover raw events, record versions/owners/times, compaction lineage, exact serialized prompt, token estimator/version/error, selected memory/order, tool-call structure, output reserve, model/prompt revision, and stage timings. Replay with oracle written state, oracle selection, uncompressed context, changed position, and memory disabled. Rank explanations only after these discriminators; patch the earliest failing stage; remeasure task utility, privacy, latency, and abstention.

## 08 Mastery Assessment

Design context and memory for a multi-session technical assistant that uses tools, honors corrections/deletion, remembers project decisions, and operates under strict context, latency, privacy, and cost constraints. Deliver category budgets and admission; effective-context surface; typed/versioned schemas; write/read/forget policy; poisoned/stale/privacy tests; compaction frontier; stage metrics and oracle replays; current source trace; canary/rollback; and diagnosis of Incident 11.1.

## 09 Required Evidence & Rubric

- **Budget:** tokenizer/protocol/model are pinned; tool/framing/multimodal costs and output reserve are included.
- **Effective context:** length, position, distractors, multiplicity, complexity, and model/prompt revisions are crossed.
- **Memory lifecycle:** identity, provenance, owner, valid/transaction time, confidence, TTL, update, deletion, and derived lineage are explicit.
- **Selection:** access/validity are hard constraints; ranking and conflict policy are measured by slice.
- **Compaction:** semantic loss, recursive drift, protocol validity, preprocessing, and end-to-end performance are measured.
- **Diagnosis:** write, persistence, selection, placement, and use are separated with oracle-stage tests.
- **Operations:** privacy, deletion, canary, rollback, and raw-event recovery are demonstrated.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Context accounting and admission | 11.1 | LAB A | Incident / Mastery | Ledger and estimator-error report |
| Effective-context measurement | 11.2 | LAB B | Mastery | Length-position-complexity surface |
| Typed memory lifecycle | 11.3–11.4 | LAB C | Incident / Mastery | Version, delete, privacy, poison tests |
| Compaction and diagnosis | 11.5–11.6 | LAB D | Incident / Mastery | Semantic-loss/performance frontier |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to account for serialized context and output reserve; distinguish accepted from effective length; reject human-memory analogies without software semantics; survive update/delete/privacy and replay failures; prove summaries are recoverable derived state; identify compression break-even rather than assume it; trace current source without generalizing it; and locate failures across write, persistence, read, placement, and use.

**Final mental model:** context is scarce execution input; memory is a governed state system. Reliability comes from budgeting, typed/versioned records, selective and authorized movement into context, recoverable compaction, and stage-level falsification—not from replaying more text.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

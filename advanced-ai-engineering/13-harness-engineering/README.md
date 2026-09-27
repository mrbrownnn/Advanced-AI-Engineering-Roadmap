# Module 13 — Harness Engineering

## 00 Why This Module Exists

A model API is not an application contract. The harness is the typed boundary that converts application intent, context, tools, and output requirements into an effective model request, then converts provider events into a validated application outcome. A weak harness makes incompatible targets look interchangeable, treats valid JSON as truth, loses streaming state, retries permanent errors, and cannot explain which request actually ran.

```text
application request + policy + context + schemas
                         |
                  canonical manifest
                         |
               capability negotiation
                  /              \
          unsupported         provider adapter
                                  |
                    effective serialized request
                                  |
                    raw response/event stream
                                  |
        assembly -> normalization -> validation ladder
                                  |
              accepted | repair/retry | fallback | fail
```

This module owns model-I/O request and response contracts, prompt/template serialization, target capability negotiation, provider adapters, structured outputs, constrained decoding, streaming assembly, error normalization, bounded retry/fallback, attempt lineage, and replay/conformance evidence. Module 12 owns the agent loop and tool-action policy; Module 14 owns durable checkpointing and side-effect execution; Module 15 owns the full evaluation program; Module 17 owns compatibility and rollout as the harness evolves; Module 23 owns platform-wide observability.

**Research cutoff:** 2026-09-26.

**Module Orientation**
- **Engineering Problem**: Preserve required semantics across changing model endpoints while producing typed, observable, bounded outcomes.
- **What You Will Do**: Build canonical and effective request models, probe backend capabilities, validate structured output, assemble adversarial streams, bound retries/fallbacks, trace XGrammar, and defend a portable production harness.
- **Environment**: Python 3.10+, two mock or disposable model adapters, JSON Schema fixtures, a streaming-event replayer, and privacy-safe request/attempt storage.
- **Evidence Rule**: Distinguish source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**). Adapter convenience is not evidence of semantic equivalence.

## 01 Baseline Assumptions

- Module 00: experimental units, paired comparisons, uncertainty, and falsification.
- Modules 01 and 06: tokenization, sampling, stopping, and inference-time behavior.
- Module 04: latency distributions, deadlines, retries as load, and goodput.
- Module 07: behavioral uncertainty, abstention, and regression slicing.
- Modules 08–11: lineage, evidence, context accounting, and versioned state.
- Module 12: tool contracts, authority, effect evidence, and bounded agent loops.

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
  security: SELECTIVE
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 4h
  guided_practice: 3h
  labs: 12h
  assessment: 3h
  source_trace: 2h
  total: 24h
```

The learner must be able to define canonical and effective requests; reject silent capability loss; version prompts and serializers; separate parse, schema, semantic, authority, and effect checks; implement structured and streaming outputs; classify errors before retry/fallback; account for every attempt; pin and trace a structured-decoding implementation; and diagnose raw-versus-normalized protocol drift.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
                       application contract
                 task | content | tools | output
                              |
                    canonical request manifest
                              |
                  required capability set C_req
                              |
                 target capability set C_verified
                    /                       \
               reject/degrade          adapter compile
                                              |
                                  effective request bytes
                                              |
                                 provider/runtime/model
                                              |
                                    ordered raw events
                                              |
                         stream assembler + normalizer
                                              |
             parse -> schema -> invariants -> task/evidence
                         -> authority -> postcondition
```

Keep three representations:

1. **Canonical request:** application meaning before provider translation.
2. **Effective request:** exact transformed payload, resolved target, and adapter policy.
3. **Raw plus normalized response:** retain transport truth while exposing stable application types.

## 04 Lessons

### Lesson 13.1 — Harness Boundary and Effective Request

**Engineering Question:**
Which representations must be retained to explain what the application intended, what the adapter sent, and what the provider returned?

**Concepts & Definitions:**
- **Canonical request**: provider-neutral application intent and requirements.
- **Effective request**: exact resolved target, transformed payload, and adapter policy.
- **Normalized response**: stable application type with provenance to raw events.

**Mechanism Explanation:**

A canonical request manifest includes task/version, prompt template and rendered messages, typed content blocks, context and memory selections, tool and output schemas, model alias and resolved target, sampling/stopping parameters, timeout, metadata, and adapter policy. The adapter compiles it into an effective request; record both where policy permits.

Prompts and serializers are executable dependencies. A 2025 controlled study reports sensitivity to subtle phrasing and formatting changes across its evaluated tasks and models. Do not universalize its effect sizes; use it to justify versioning delimiters, ordering, examples, escaping, chat templates, and schema renderers and testing them on your workload.

The normalized response must not destroy the raw response. Stable application fields need provenance back to raw events and an explicit `missing`, `unsupported`, or `not reported` state rather than fabricated defaults.

**Quantitative Model / Derivation:**
Let $R_c$ be a canonical request and $A_p$ the adapter for provider $p$. The effective request is $R_e=A_p(R_c)$. A conformance check compares required semantic fields before and after this transformation; byte equality is neither required nor sufficient.

**Worked Example:**
Diff a canonical request requiring tool choice, strict schema, stop sequence, and usage reporting against an adapter that silently drops strictness and fabricates zero usage. Mark dropped, emulated, and unreported fields explicitly.

**Knowledge Check:**
1. Why must raw provider events survive normalization?
2. Which version identifiers are needed to reproduce prompt serialization?

**Guided Practice:**
Create canonical and effective manifests for the same task through two adapters, then explain every semantic difference.

**Feedback Contract:**
- *Expected Evidence*: Exact target, prompt/schema/serializer versions, transformation diff, raw response, and explicit missing/unsupported states.
- *Common Failure*: Recording only the friendly SDK object.
- *Diagnostic Hint*: Can the exact outbound payload be reconstructed?
- *Concept to Revisit*: Canonical vs. Effective Request.

**Learning Outcome:**
Replay which logical request was intended, which bytes/events were exchanged, and which transformations produced the application result.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 13.2 — Capability Negotiation and Portability

**Engineering Question:**
When is a fallback semantically compatible rather than merely API-shaped like the preferred backend?

**Concepts & Definitions:**
- **Required capability set**: features whose absence changes the application contract.
- **Verified capability set**: behavior demonstrated for a pinned target and adapter.
- **Degraded outcome**: explicit result when policy authorizes relaxing a requirement.

**Mechanism Explanation:**

Define required capabilities `C_req(r)` and verified target capabilities `C_verified(p)`. Dispatch only when:

$$
C_{req}(r)\subseteq C_{verified}(p),
$$

after recording any deliberate emulation. This is an exact policy rule; discovering capabilities is empirical and must be refreshed.

Probe at least content roles/types, context and output limits, tool-call semantics, schema dialect/subset, streaming event types, log probabilities, seeds, stop behavior, usage, cancellation, refusals, and errors. The same field name or “compatible API” label does not prove identical behavior.

A fallback is compatible only if it satisfies required capabilities and passes task-specific prompt/schema/serializer fixtures. If emergency policy relaxes a requirement, return an explicit degraded outcome. Routing for price/quality optimization belongs to Module 22; here the question is whether a call remains contract-compatible.

**Quantitative Model / Derivation:**
Dispatch is allowed only when $C_{req}(r)\subseteq C_{verified}(p)$ after declared emulation. This set rule is exact for a manifest; the membership evidence is empirical and can expire.

**Worked Example:**
A fallback supports JSON objects but not the required recursive schema or parallel tool-call identity. The endpoint is reachable yet incompatible, so the harness rejects or returns an explicitly degraded outcome.

**Knowledge Check:**
1. Why does an identical parameter name not prove identical behavior?
2. Which capability probes must rerun after a provider/model revision?

**Independent Practice:**
Build a two-target capability matrix and a fixture that catches one silent default and one unsupported feature.

**Feedback Contract:**
- *Expected Evidence*: Each required capability maps to a current probe, version, and failure policy.
- *Common Failure*: Treating successful HTTP transport as portability.
- *Diagnostic Hint*: Which required semantic was actually exercised?
- *Concept to Revisit*: Evidence-Backed Capability Negotiation.

**Learning Outcome:**
Reject silent field loss and demonstrate portability with conformance evidence rather than adapter claims.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 13.3 — Structured Output and Constrained Decoding

**Engineering Question:**
What does constrained decoding guarantee, and which semantic, authority, and effect checks remain outside the grammar?

**Concepts & Definitions:**
- **Dialect/subset**: the exact schema vocabulary implemented by the validator or decoder.
- **Constrained decoding**: masking invalid next tokens for an implemented language.
- **Validation ladder**: parse, schema, invariants, task/evidence, authority, and effect checks.

**Mechanism Explanation:**

Pin the schema dialect and vocabulary. JSON Schema Draft 2020-12 separates Core and Validation vocabularies; `format` is annotation by default unless assertion behavior is enabled. Hosted APIs and decoding libraries may implement different subsets, strictness, recursion, ordering, and extensions.

Use a validation ladder:

```text
bytes/text parse?
  -> declared schema valid?
    -> application invariants valid?
      -> task/factual/evidence checks pass?
        -> action authorized?
          -> external postcondition observed?
```

These predicates are not equivalent. Constrained decoding can keep generation inside an implemented formal language by masking invalid continuations; it cannot prove facts, business invariants absent from the grammar, permission, or side effects.

JSONSchemaBench evaluates schema-feature coverage, efficiency, and output quality separately across real-world schemas and the official test suite. Use the same dimensions for backend selection; valid-JSON rate alone hides unsupported keywords and semantic failure.

**Production source trace:** at XGrammar revision `4221346f3d26b8306b46de19cb40c8a525c4871c`, `GrammarCompiler.compile_json_schema` produces a tokenizer-aware compiled grammar, `GrammarMatcher.fill_next_token_bitmask` and `accept_token` expose stateful validity, and the Transformers `LogitsProcessor.__call__` accepts the last token, fills the next mask, and applies it to logits. Static inspection only; native code and benchmarks were not executed.

XGrammar-2's 2026 maintainer material extends the family with composable structural tags and batching/speculative-decoding integration. Treat it as frontier, implementation-specific evidence; do not transfer its performance or adoption claims without reproduction.

**Quantitative Model / Trade-off Comparison:**
Report schema-feature coverage, compile/cache time, decode latency, structural validity, semantic validity, and task success separately. A structurally valid rate cannot be substituted for semantic accuracy.

**Worked Example:**
The object `{\"amount\":-10,\"account\":\"B\"}` can satisfy a permissive JSON grammar while violating a business invariant and caller authority. Walk it through every validation stage.

**Knowledge Check:**
1. Why is `format` behavior dependent on dialect/configuration?
2. Can a grammar verify that a cited fact is true?

**Guided Practice:**
Test nested unions, references, optional fields, impossible domains, and unsupported keywords across post-hoc validation and one constrained backend.

**Feedback Contract:**
- *Expected Evidence*: Pinned dialect/subset, feature tests, structural and semantic metrics, and unsupported cases.
- *Common Failure*: Reporting valid JSON as task correctness.
- *Diagnostic Hint*: At which validation stage did the candidate first fail?
- *Concept to Revisit*: Validation Ladder.

**Learning Outcome:**
Prove structural validity for supported constraints while keeping semantic validation and system effects separate.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 13.4 — Streaming Is a Protocol State Machine

**Engineering Question:**
How can arbitrary legal event boundaries yield one stable logical response without converting truncation or cancellation into success?

**Concepts & Definitions:**
- **Transport event**: one provider chunk with protocol identity and ordering metadata.
- **Logical object**: content or tool call assembled according to terminal semantics.
- **Terminal state**: complete, truncated, cancelled, refused, or failed.

**Mechanism Explanation:**

Provider chunks are transport events, not complete JSON objects or messages. Preserve sequence/event IDs, choice index, tool-call ID, event type, raw bytes/text, timestamps, and terminal/error/cancel signals. Assemble fragmented Unicode, content, reasoning channels where exposed, and tool arguments by stable identity.

Do not parse every partial JSON fragment as though it were complete. Track states such as `started`, `partial`, `complete`, `truncated`, `cancelled`, `refused`, and `failed`; validate a logical object when its protocol says it is complete. Client disconnect does not prove provider cancellation, and a finish reason must remain distinct from application acceptance.

Test arbitrary boundary splits, duplicated/out-of-order events where transport allows them, parallel choices/calls, missing terminators, early disconnect, content after a terminal marker, and backpressure from a slow consumer.

**Quantitative Model / Derivation:**
For a legal event sequence $E$, assembly is a stateful fold $S_n=F(S_0,E)$. Chunk-boundary invariance requires legal partitions of the same byte/event semantics to produce the same completed logical object and terminal state.

**Worked Example:**
Split a multibyte character and tool arguments across four events, then inject a disconnect before the terminal event. The assembler may retain a partial object but must not validate it as complete.

**Knowledge Check:**
1. Why is parsing each JSON fragment independently incorrect?
2. Does client disconnect prove provider-side cancellation?

**Guided Practice:**
Replay one response over randomized legal chunkings, duplicate/out-of-order cases permitted by the transport, missing terminators, and slow-consumer backpressure.

**Feedback Contract:**
- *Expected Evidence*: Raw ordered events, stable IDs, explicit terminal state, and invariant completed output across legal splits.
- *Common Failure*: Treating end-of-connection as successful completion.
- *Diagnostic Hint*: Which protocol event authorizes final validation?
- *Concept to Revisit*: Streaming State Machine.

**Learning Outcome:**
Reconstruct the same completed logical response across legal chunkings and fail closed on ambiguous termination.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 13.5 — Error Taxonomy, Repair, Retry, and Fallback

**Engineering Question:**
Which failure classes justify retry, repair, fallback, abstention, or immediate stop under one root deadline?

**Concepts & Definitions:**
- **Stage/class**: where and why a failure occurred.
- **Root budget**: attempts, deadline, tokens, and cost shared across all recovery layers.
- **Semantic retry**: another stochastic sample, not restoration of a transport exchange.

**Mechanism Explanation:**

Normalize both stage and class:

| Stage/class | Default decision direction |
|---|---|
| Local schema/config validation | reject or repair before dispatch |
| Authentication/permission/policy | do not retry unchanged |
| Connection/transport with no response | bounded retry if deadline permits |
| Rate limit/overload | honor server signal; budgeted backoff/fallback |
| Deadline/cancellation | stop work or return explicit unfinished state |
| Truncation/context/output limit | change request/budget; blind resample is insufficient |
| Parse/schema failure | validate backend support; bounded repair/resample |
| Refusal/content policy | preserve as typed outcome, not transport error |
| Semantic/evidence failure | new sample, validator-guided repair, abstain, or escalate |

All attempts share root request ID, absolute deadline, maximum attempts, token/cost budget, and cancellation. A semantic retry is another stochastic sample, not restoration of a failed network exchange. Never allow nested SDK, gateway, harness, and application retries to multiply invisibly.

**Quantitative Model / Derivation:**
If layer $j$ independently permits $r_j$ attempts, hidden nesting can allow up to $\prod_j r_j$ dispatches. A root attempt budget converts this multiplication into one explicit bound.

**Worked Example:**
An SDK, gateway, and application each allow three attempts. Without coordination the worst case is 27 dispatches; with a root maximum of three, every layer consumes the same ledger and deadline.

**Knowledge Check:**
1. Why should permission denial not be retried unchanged?
2. When is a fallback incompatible even if it returns parseable output?

**Guided Practice:**
Map each error-table row to a decision, evidence requirement, deadline behavior, and terminal outcome.

**Feedback Contract:**
- *Expected Evidence*: Stage/class, effect certainty, shared deadline/attempt ledger, and fallback capability proof.
- *Common Failure*: Nested retries or a fresh deadline at every layer.
- *Diagnostic Hint*: How many physical dispatches can one root request create?
- *Concept to Revisit*: Attempt Amplification.

**Learning Outcome:**
Explain why each retry or fallback is safe, useful, and still inside the original contract.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 13.6 — Conformance, Replay, and Attempt Accounting

**Engineering Question:**
What evidence localizes drift to request construction, backend behavior, streaming, normalization, validation, or recovery?

**Concepts & Definitions:**
- **Replay manifest**: immutable lineage for logical request, effective attempts, raw events, validation, and terminal state.
- **Conformance fixture**: a known contract probe, not a promise of bitwise model replay.
- **Attempt amplification**: work across all attempts relative to root requests or accepted output.

**Mechanism Explanation:**

A replay manifest records resolved endpoint/model, harness/adapter versions, canonical and effective requests, prompt/tool/schema IDs, sampling/stopping settings, seed if supported, timestamps, raw events, normalized output, validation results, attempts, usage, and terminal status. This supports diagnosis but cannot guarantee bitwise replay when kernels are stochastic, providers revise hidden components, or state is unavailable.

**Quantitative Model / Derivation:**
For sequential attempts:

$$
L_{e2e}=L_{pre}+\sum_{i=1}^{N}
(L_{serialize,i}+L_{queue/connect,i}+L_{stream,i}+L_{validate/repair,i}+L_{backoff,i}).
$$

Use the critical path for overlapping work. Sum tokens and cost across every attempt. Track:

$$
A=\frac{N_{attempts}}{N_{root\ requests}},\qquad
A_{tok}=\frac{T_{all\ attempts}}{T_{final\ accepted}}.
$$

State denominator policy when no output is accepted. These ratios measure amplification, not quality.

Run conformance fixtures for canonical requests, schema vocabulary, stream splitting, errors, cancellation, usage, and raw-versus-normalized preservation. The claim that such probes catch adapter drift earlier is an **H**: compare time-to-detection and user-impacting failures against outcome-only monitoring.

**Worked Example:**
For two root requests with one and three attempts, $A=4/2=2$. If all attempts consume 2,400 tokens and accepted outputs consume 800, $A_{tok}=3$; state the zero-accepted-output policy separately.

**Knowledge Check:**
1. Why can a complete manifest still fail to reproduce identical tokens?
2. Which latency formula applies to overlapping fallback probes?

**Independent Practice:**
Change one serializer, adapter, schema subset, endpoint, and event shape at a time; use raw/effective diffs and fixtures to rank the failing boundary.

**Feedback Contract:**
- *Expected Evidence*: Complete attempts, raw and normalized data, critical-path timing, all-attempt usage/cost, and a bounded reproducibility claim.
- *Common Failure*: Keeping only the final accepted response.
- *Diagnostic Hint*: What is the earliest representation that differs?
- *Concept to Revisit*: Replay Manifest and Attempt Ledger.

**Learning Outcome:**
Attribute regressions to request construction, target capability, provider behavior, stream assembly, normalization, validation, or recovery.

*(Effort: 35m instruction, 25m practice)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12) — dialect/vocabulary and validation semantics.
- [Grammar-Constrained Decoding for Structured NLP Tasks without Finetuning](https://arxiv.org/abs/2305.13971) — Geng et al., EMNLP 2023.
- [XGrammar](https://arxiv.org/abs/2411.15100) — Dong et al., MLSys 2025.

**CURRENT DEFAULT:** canonical/effective request separation; versioned prompts/schemas/adapters; explicit capability checks; preservation of raw protocol data; staged validation; stream state machines; typed errors; bounded shared retry budget; complete attempt accounting.

**WORKLOAD-DEPENDENT:** prompt format, schema complexity, constrained-decoding backend, repair strategy, semantic validator, retry count/backoff, fallback set, and retention/redaction policy.

**FRONTIER:** [JSONSchemaBench](https://arxiv.org/abs/2501.10868) as a broad structured-output benchmark; [XGrammar-2](https://blog.mlc.ai/2026/05/04/xgrammar-2-fast-customizable-structured-generation) structural tags for mixed free-form/structured protocols; 2025–2026 prompt-robustness methods. None establishes universal portability or zero overhead.

**LEGACY / INSUFFICIENT:** concatenate strings into a prompt; parse JSON with regex; trust “JSON mode” as schema or semantic correctness; assume OpenAI-shaped endpoints are semantically identical; parse every stream chunk independently; retry all exceptions; record only the final accepted response; treat a seed as deterministic replay.

**PRODUCTION SOURCE TRACE**

- Repository: `mlc-ai/xgrammar`
- Revision: `4221346f3d26b8306b46de19cb40c8a525c4871c`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `python/xgrammar/compiler.py::GrammarCompiler.compile_json_schema`, `python/xgrammar/matcher.py::{GrammarMatcher.fill_next_token_bitmask,GrammarMatcher.accept_token}`, and `python/xgrammar/contrib/hf.py::LogitsProcessor.__call__`.
- Execution path: schema + tokenizer information → compiled grammar → matcher state → valid-next-token bitmask → logits masking → sampled token → matcher advance.
- Scope: current pinned implementation example, not proof of every schema feature, hosted provider, semantic output, or claimed benchmark speed.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Canonical Request and Adapter Conformance

- **Objective**: Build canonical request/response/event/error contracts and demonstrate semantic conformance across two deliberately different adapters.
- **Pre-Registered Hypothesis**: Capability probes will identify at least one incompatibility hidden by transport-level success before production outcome monitoring does.
- **Independent Variables**: Adapter, target, capability, serializer, and request fixture.
- **Dependent Variables**: Contract preservation, explicit degradation/rejection, drift detection, latency, and attempt cost.

- Define typed canonical request/response/event/error models and two deliberately different adapters.
- Build a capability registry and canonical fixtures for roles, content blocks, tools, output schemas, stops, usage, refusal, and errors.
- Break with unknown fields, silent defaults, prompt escaping, reordered examples, unsupported parameters, changed context limits, and false compatible fallbacks.
- Artifact: canonical/effective request diff, capability evidence, and portability matrix.
- **Break & Falsify**: Inject unknown fields, silent defaults, escaping changes, unsupported parameters, and an incompatible fallback; any silent required-field loss falsifies conformance.
- **Alignment**: Lessons 13.1–13.2.
- **Effort Estimate**: 3h total.

### LAB B — Structured-Output Validation Ladder

- **Objective**: Compare post-hoc and constrained structured-output paths across schema features and semantic checks.
- **Pre-Registered Hypothesis**: Structural constraints will improve supported structural validity but will not eliminate semantic or task failures.
- **Independent Variables**: Schema feature, backend, constraint path, model/seed, and payload length.
- **Dependent Variables**: Compile/cache time, decode latency, feature coverage, parse/schema/semantic/task outcomes, tokens, and cost.

- Implement unconstrained JSON prompting, post-hoc parse/repair, schema validation, and constrained decoding.
- Cross schemas with enums, nested unions, recursion/references, optional/required fields, numeric/string constraints, `format`, additional/unevaluated properties, impossible/empty domains, and long tool sets.
- Measure compile time/cache, decode latency, feature coverage, parse/schema success, semantic/task quality, refusals, truncation, tokens, and cost.
- Break “valid JSON means correct”; trace the pinned XGrammar path and report unsupported semantics.
- **Break & Falsify**: Include impossible domains, unsupported keywords, and structurally valid semantic violations; a result where semantic errors remain falsifies equivalence of grammar and correctness.
- **Alignment**: Lesson 13.3.
- **Effort Estimate**: 3h total.

### LAB C — Streaming and Recovery Chaos

- **Objective**: Prove stream assembly and recovery semantics under legal chunking and injected protocol/error faults.
- **Pre-Registered Hypothesis**: Legal chunk partitions will yield invariant completed objects, while ambiguous termination remains non-success under the declared protocol.
- **Independent Variables**: Chunking/order, event fault, error class, timeout, retry/fallback policy, and consumer speed.
- **Dependent Variables**: Assembly agreement, terminal-state correctness, attempts, latency, tokens/cost, and unsafe success.

- Replay identical logical responses under every legal chunk split, including fragmented Unicode and parallel tool arguments.
- Inject missing terminal events, duplicate/out-of-order events, disconnect, cancellation race, 429/5xx, timeout, truncation, refusal, invalid schema, and semantic failure.
- Verify typed terminal states, stable assembly, absolute deadlines, shared attempt budgets, no nested retry multiplication, and compatible fallback only.
- Artifact: event-state machine, fault matrix, attempt tree, and latency/cost amplification report.
- **Break & Falsify**: Inject missing terminal, duplication/order faults, disconnect, cancellation race, overload, truncation, refusal, invalid schema, and nested retries; any ambiguous success or budget escape falsifies the design.
- **Alignment**: Lessons 13.4–13.5.
- **Effort Estimate**: 3h total.

### LAB D — Replay and Drift Detection

- **Objective**: Operate versioned replay/conformance evidence and localize one-at-a-time harness drift.
- **Pre-Registered Hypothesis**: Raw/effective manifests plus targeted fixtures will localize seeded adapter drift earlier than end-outcome-only monitoring for at least one preregistered change.
- **Independent Variables**: Prompt/serializer, adapter, schema dialect, endpoint/model, and event shape.
- **Dependent Variables**: Detection delay, attribution accuracy, user-impacting escapes, false alerts, stage latency, and cost.

- Persist privacy-safe canonical/effective manifests, raw/normalized fixtures, validation results, usage, timing, and terminal status.
- Change prompt serializer, adapter version, schema dialect/subset, endpoint, model, and streaming event shape one at a time.
- Compare conformance probes with end-outcome monitoring for detection delay and user impact; include no-op and intentionally degraded changes.
- Artifact: ranked regression attribution, canary policy, rollback evidence, and limits on reproducibility.
- **Break & Falsify**: Include no-op and intentionally degraded changes; failure to improve detection for the seeded set limits the hypothesis.
- **Alignment**: Lesson 13.6 and Incident 13.1.
- **Effort Estimate**: 3h total.

## 07 Break / Incident Scenarios

### Incident 13.1 — A “Compatible” Provider Migration Corrupts Tool Calls

- **Incident Symptoms**: A new endpoint passes smoke tests and lowers median latency, but streamed tool arguments intermittently fail parsing; retries amplify cost; fallback changes semantics; refusals become empty successes; usage disappears; and dashboards retain only final successes.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: Serializer drift, unsupported schema features, strictness/`format` differences, chunk assembly, tool-call identity, truncation, error normalization, nested retries, incompatible fallback, model drift, or survivorship bias.
  2. *Rank Initial Plausibility*: Use raw-event and effective-request differences without assuming the provider alone is causal.
  3. *Identify Missing Evidence*: Recover canonical/effective requests, versions, probes, dialect/subset, ordered raw events, terminal states, validation stages, all attempts/backoff, usage, spans, and fallback decisions.
  4. *Design Discriminating Tests*: Replay identical raw events through old/new assemblers, bypass normalization, disable retries/fallback, and run an independent semantic oracle; state falsifying outcomes.
  5. *Execute Causal Diagnosis*: Rank the earliest differing boundary and interacting retry/monitoring effects.
  6. *Prescribe Mitigation and Prevention*: Roll back incompatible routing, preserve raw failures, repair the earliest boundary, and gate targets with conformance probes.
  7. *Remeasure*: Schema coverage, semantic success, attempt/token amplification, latency, refusals/failures, and offered-request goodput.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Portable Multi-Backend Model Gateway

Design a harness for two non-identical backends supporting text, tools, structured output, and streaming under one application contract.

**Required Deliverables**:
1. Canonical/effective request and raw/normalized response schemas.
2. Capability registry, probes, and degradation/rejection policy.
3. Versioned prompt/serializer fixtures and schema dialect/subset policy.
4. Validation ladder and pinned constrained-decoding source trace.
5. Streaming state machine and adversarial chunking evidence.
6. Error taxonomy, shared retry/fallback budget, attempt ledger, and replay manifest.
7. Chaos/load evidence plus canary, kill, and rollback plan.
8. Evidence-backed diagnosis of Incident 13.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit a pinned trace from schema compilation through matcher state, valid-next-token mask, logits processing, and token acceptance. State which schema semantics and native/runtime behavior were not executed or verified.

### Rubric Dimensions

- **Boundary and Portability**: *Insufficient* records one SDK object. *Competent* separates canonical/effective/raw/normalized forms and probes capabilities. *Strong* demonstrates drift localization and explicit degradation.
- **Structure and Streaming**: *Insufficient* equates JSON with correctness. *Competent* pins dialect/subset and implements terminal-aware assembly. *Strong* proves legal-chunk invariance and layered validation under faults.
- **Recovery and Accounting**: *Insufficient* retries errors independently. *Competent* shares deadlines/budgets and records every attempt. *Strong* quantifies amplification and rejects incompatible fallback.
- **Diagnosis and Source Trace**: *Insufficient* blames a provider from symptoms. *Competent* traces a pinned path and compares raw/effective evidence. *Strong* uses discriminating replays, oracle stages, and remeasurement.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Canonical/effective model I/O contract | 13.1 | LAB A | Incident / Mastery | Request diff and provenance |
| Capability negotiation and portability | 13.2 | LAB A, LAB D | Incident / Mastery | Probe and conformance matrix |
| Structured output and validation | 13.3 | LAB B | Incident / Mastery | Coverage/quality/latency surface |
| Streaming and recovery | 13.4–13.5 | LAB C | Incident / Mastery | Event invariance and fault matrix |
| Replay and attempt accounting | 13.6 | LAB D | Incident / Mastery | Replay manifest and amplification report |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner successfully completing Module 13 must be able to:
1. Preserve canonical intent, effective payloads, raw events, normalized output, and accepted outcomes.
2. Prove required capabilities and make unsupported/degraded behavior explicit.
3. Version prompts, schemas, serializers, adapters, and resolved targets.
4. Separate structural, semantic, authority, and effect validation.
5. Assemble streams by protocol state and fail closed on ambiguous termination.
6. Classify errors before bounded recovery and account for every attempt.
7. Trace pinned source without universalizing one implementation.

### Module Wrap-Up (Final Mental Model Reconstruction)

- **The Core Invariant**: The harness is a protocol compiler and evidence-preserving validator around an uncertain model endpoint.
- **The Data Path**: `application contract → canonical request → capability check → effective payload → raw events → assembly/normalization → validation → typed outcome`.
- Reliability comes from explicit capabilities, reversible transformations, staged validation, stateful streaming, bounded recovery, and complete attempt lineage.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

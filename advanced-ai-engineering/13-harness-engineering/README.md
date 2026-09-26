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

- **Engineering problem:** preserve required semantics across changing model endpoints while producing typed, observable, bounded outcomes.
- **Evidence rule:** distinguish source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**). Adapter convenience is not evidence of semantic equivalence.

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
  falsification: REQUIRED
```

The learner must be able to define canonical and effective requests; reject silent capability loss; version prompts and serializers; separate parse, schema, semantic, authority, and effect checks; implement structured and streaming outputs; classify errors before retry/fallback; account for every attempt; pin and trace a structured-decoding implementation; and diagnose raw-versus-normalized protocol drift.

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

A canonical request manifest includes task/version, prompt template and rendered messages, typed content blocks, context and memory selections, tool and output schemas, model alias and resolved target, sampling/stopping parameters, timeout, metadata, and adapter policy. The adapter compiles it into an effective request; record both where policy permits.

Prompts and serializers are executable dependencies. A 2025 controlled study reports sensitivity to subtle phrasing and formatting changes across its evaluated tasks and models. Do not universalize its effect sizes; use it to justify versioning delimiters, ordering, examples, escaping, chat templates, and schema renderers and testing them on your workload.

The normalized response must not destroy the raw response. Stable application fields need provenance back to raw events and an explicit `missing`, `unsupported`, or `not reported` state rather than fabricated defaults.

**Outcome:** replay which logical request was intended, which bytes/events were exchanged, and which transformations produced the application result.

### Lesson 13.2 — Capability Negotiation and Portability

Define required capabilities `C_req(r)` and verified target capabilities `C_verified(p)`. Dispatch only when:

$$
C_{req}(r)\subseteq C_{verified}(p),
$$

after recording any deliberate emulation. This is an exact policy rule; discovering capabilities is empirical and must be refreshed.

Probe at least content roles/types, context and output limits, tool-call semantics, schema dialect/subset, streaming event types, log probabilities, seeds, stop behavior, usage, cancellation, refusals, and errors. The same field name or “compatible API” label does not prove identical behavior.

A fallback is compatible only if it satisfies required capabilities and passes task-specific prompt/schema/serializer fixtures. If emergency policy relaxes a requirement, return an explicit degraded outcome. Routing for price/quality optimization belongs to Module 22; here the question is whether a call remains contract-compatible.

**Outcome:** reject silent field loss and demonstrate portability with conformance evidence rather than adapter claims.

### Lesson 13.3 — Structured Output and Constrained Decoding

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

**Outcome:** prove structural validity for supported constraints while keeping semantic validation and system effects separate.

### Lesson 13.4 — Streaming Is a Protocol State Machine

Provider chunks are transport events, not complete JSON objects or messages. Preserve sequence/event IDs, choice index, tool-call ID, event type, raw bytes/text, timestamps, and terminal/error/cancel signals. Assemble fragmented Unicode, content, reasoning channels where exposed, and tool arguments by stable identity.

Do not parse every partial JSON fragment as though it were complete. Track states such as `started`, `partial`, `complete`, `truncated`, `cancelled`, `refused`, and `failed`; validate a logical object when its protocol says it is complete. Client disconnect does not prove provider cancellation, and a finish reason must remain distinct from application acceptance.

Test arbitrary boundary splits, duplicated/out-of-order events where transport allows them, parallel choices/calls, missing terminators, early disconnect, content after a terminal marker, and backpressure from a slow consumer.

**Outcome:** reconstruct the same completed logical response across legal chunkings and fail closed on ambiguous termination.

### Lesson 13.5 — Error Taxonomy, Repair, Retry, and Fallback

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

**Outcome:** explain why each retry or fallback is safe, useful, and still inside the original contract.

### Lesson 13.6 — Conformance, Replay, and Attempt Accounting

A replay manifest records resolved endpoint/model, harness/adapter versions, canonical and effective requests, prompt/tool/schema IDs, sampling/stopping settings, seed if supported, timestamps, raw events, normalized output, validation results, attempts, usage, and terminal status. This supports diagnosis but cannot guarantee bitwise replay when kernels are stochastic, providers revise hidden components, or state is unavailable.

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

**Outcome:** attribute regressions to request construction, target capability, provider behavior, stream assembly, normalization, validation, or recovery.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Canonical Request and Adapter Conformance

- Define typed canonical request/response/event/error models and two deliberately different adapters.
- Build a capability registry and canonical fixtures for roles, content blocks, tools, output schemas, stops, usage, refusal, and errors.
- Break with unknown fields, silent defaults, prompt escaping, reordered examples, unsupported parameters, changed context limits, and false compatible fallbacks.
- Artifact: canonical/effective request diff, capability evidence, and portability matrix.

### LAB B — Structured-Output Validation Ladder

- Implement unconstrained JSON prompting, post-hoc parse/repair, schema validation, and constrained decoding.
- Cross schemas with enums, nested unions, recursion/references, optional/required fields, numeric/string constraints, `format`, additional/unevaluated properties, impossible/empty domains, and long tool sets.
- Measure compile time/cache, decode latency, feature coverage, parse/schema success, semantic/task quality, refusals, truncation, tokens, and cost.
- Break “valid JSON means correct”; trace the pinned XGrammar path and report unsupported semantics.

### LAB C — Streaming and Recovery Chaos

- Replay identical logical responses under every legal chunk split, including fragmented Unicode and parallel tool arguments.
- Inject missing terminal events, duplicate/out-of-order events, disconnect, cancellation race, 429/5xx, timeout, truncation, refusal, invalid schema, and semantic failure.
- Verify typed terminal states, stable assembly, absolute deadlines, shared attempt budgets, no nested retry multiplication, and compatible fallback only.
- Artifact: event-state machine, fault matrix, attempt tree, and latency/cost amplification report.

### LAB D — Replay and Drift Detection

- Persist privacy-safe canonical/effective manifests, raw/normalized fixtures, validation results, usage, timing, and terminal status.
- Change prompt serializer, adapter version, schema dialect/subset, endpoint, model, and streaming event shape one at a time.
- Compare conformance probes with end-outcome monitoring for detection delay and user impact; include no-op and intentionally degraded changes.
- Artifact: ranked regression attribution, canary policy, rollback evidence, and limits on reproducibility.

## 07 Break / Incident Scenarios

### Incident 13.1 — A “Compatible” Provider Migration Corrupts Tool Calls

A new endpoint passes smoke tests and lowers median latency. Under production load, streamed tool arguments intermittently fail JSON parsing; retries triple cost; a fallback emits parseable but semantically different fields; refusals appear as empty successes; usage is missing; and only final successful attempts appear in dashboards.

Competing explanations include changed prompt/chat serialization, unsupported schema keywords, different strictness or `format` behavior, chunk-boundary assembly, tool-call ID/index mismatch, output truncation, provider error misclassification, nested retry multiplication, incompatible fallback, model behavior drift, or dashboard survivorship bias.

Recover canonical and effective requests, target/adapter/model versions, capability probe results, schema dialect/subset, raw ordered events with IDs/timestamps, finish/refusal/error states, validation-stage results, every attempt/backoff, usage estimates, latency spans, and fallback decisions. Replay raw events through old/new assemblers, run official and workload schemas, bypass normalization, disable retries/fallbacks, and use an oracle semantic validator. Patch the earliest failing boundary and remeasure schema coverage, semantic success, attempt/token amplification, latency, and offered-request goodput.

## 08 Mastery Assessment

Build a harness for two non-identical model backends supporting text, tools, structured output, and streaming. Deliver canonical and effective request schemas; capability registry and probes; versioned prompt/serializer fixtures; JSON Schema dialect/subset policy; validation ladder; constrained-decoding source trace; streaming state machine; staged error taxonomy; shared retry/fallback budget; replay manifest; per-attempt metrics; chaos tests; canary/rollback plan; and a diagnosis of Incident 13.1.

## 09 Required Evidence & Rubric

- **Boundary:** canonical, effective, raw, normalized, and accepted representations remain distinct and traceable.
- **Portability:** required capabilities are proven by probes; unsupported and degraded behavior is explicit.
- **Prompts:** templates, ordering, delimiters, examples, schemas, and serializers are versioned/tested dependencies.
- **Structure:** dialect/subset and grammar/tokenizer versions are pinned; syntax, schema, semantics, authority, and effects are separate.
- **Streaming:** legal chunking yields invariant completed output; ambiguous termination cannot become success.
- **Recovery:** stage/class drive bounded retry, repair, fallback, refusal, abstention, or stop under one deadline/cost budget.
- **Accounting:** every attempt contributes to latency, usage, cost, load, and terminal-outcome metrics.
- **Diagnosis:** conformance and raw-event replays discriminate adapter, provider, model, stream, validator, and recovery hypotheses.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Canonical/effective model I/O contract | 13.1 | LAB A | Incident / Mastery | Request diff and provenance |
| Capability negotiation and portability | 13.2 | LAB A, LAB D | Incident / Mastery | Probe and conformance matrix |
| Structured output and validation | 13.3 | LAB B | Incident / Mastery | Coverage/quality/latency surface |
| Streaming and recovery | 13.4–13.5 | LAB C | Incident / Mastery | Event invariance and fault matrix |
| Replay and attempt accounting | 13.6 | LAB D | Incident / Mastery | Replay manifest and amplification report |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to show the effective request, reject unsupported capability instead of silently degrading it, version prompts and serializers, distinguish every validation gate, demonstrate structured-output coverage and limitations, assemble arbitrary legal stream chunking, classify before retrying, prove fallback compatibility, retain failed attempts in latency/cost/load, and trace pinned source without universalizing one implementation.

**Final mental model:** the harness is a protocol compiler and evidence-preserving validator around an uncertain model endpoint. Reliability comes from explicit capabilities, reversible transformations, staged validation, stateful streaming, bounded recovery, and complete attempt lineage—not from making every backend look the same.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

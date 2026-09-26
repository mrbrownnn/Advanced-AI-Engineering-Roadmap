# Module 12 — Agent Loop Engineering

## 00 Why This Module Exists

An agent is not a prompt that keeps talking. It is a controller that repeatedly converts an observed state into a proposed action, subjects that action to deterministic validation and authority policy, executes it through an external interface, incorporates the resulting observation, and stops for an explicit reason.

```text
goal + current state + remaining budget
                 |
                 v
             model proposal
                 |
        validate + authorize
          /              \
   reject/repair       execute tool
          |                |
          +---- observation envelope
                           |
                 verify progress/effect
                     /           \
             continue/replan   terminal state
```

This module owns the observe–decide–act loop, tool and observation contracts, budgets and stopping, retry/replanning, progress/stall detection, reflection as a fallible mechanism, authority boundaries, trajectory telemetry, and agent evaluation. Module 11 owns context and memory; Module 13 owns the broader model harness and structured-output reliability; Module 14 owns durable checkpoint/resume and idempotent side-effect execution; Module 18 owns full threat modeling.

**Research cutoff:** 2026-09-26.

- **Engineering problem:** maximize verified task utility while bounding invalid effects, policy violations, latency, tokens, calls, and cost.
- **Evidence rule:** label source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**). Model claims about success, failure, or tool completion are not execution evidence.

## 01 Baseline Assumptions

- Module 00: measurands, experimental units, uncertainty, and falsification.
- Module 04: queueing, end-to-end latency, throughput, goodput, overload, and backpressure.
- Module 07: behavioral uncertainty, calibration, and abstention.
- Modules 08–10: lineage, retrieval, evidence assembly, and citation correctness.
- Module 11: context accounting, typed memory, versioning, and memory provenance.

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

The learner must be able to define a loop as an explicit state machine; design typed tool and observation contracts; separate proposal from authorization; enforce independent resource and semantic stops; classify failures before retrying; detect no-progress and oscillation without blocking legitimate iteration; evaluate reflection against grounded feedback; inject tool anomalies; and diagnose complete trajectories rather than final answers alone.

## 03 Knowledge Map

```text
                         controller-owned state
                      /          |             \
                 goal/plan   budgets/policy   effect ledger
                      \          |             /
                           model proposal
                                 |
                  schema validation + authorization
                         /               \
                    invalid             valid
                 repair or stop      tool execution
                                           |
                              typed observation envelope
                                           |
                     progress? error class? postcondition?
                       /             |               \
                   continue        replan          terminate
```

Keep these distinctions explicit:

1. **Proposal:** what action did the model request?
2. **Authorization:** is this caller permitted to perform that action now?
3. **Execution:** what did the tool attempt, and with which contract/version?
4. **Effect:** did the external state change, not change, or become unknown?
5. **Observation:** what evidence was returned, parsed, truncated, or rejected?
6. **Progress:** did a verified subgoal or state predicate improve?
7. **Termination:** did the episode succeed, fail, abstain, escalate, or exhaust a budget?

## 04 Lessons

### Lesson 12.1 — Model the Loop as a Bounded Controller

Represent one episode as

$$
\tau=(s_0,a_0,o_1,s_1,\ldots,a_{T-1},o_T,s_T),\qquad
s_{t+1}=F(s_t,a_t,o_{t+1}).
$$

This is exact bookkeeping for a declared state schema. It is not automatically a Markov model: if `s_t` omits relevant history, permissions, hidden environment state, or pending effects, it is not sufficient to predict the next transition.

The controller—not the model—owns legal states, action validation, authorization, tool dispatch, budget accounting, terminal outcomes, and the trajectory ledger. The model may emit `call_tool`, `respond`, `abstain`, `ask`, or `escalate`; each is a proposal until the controller validates it.

ReAct is a reference pattern for interleaving reasoning and environment actions on evaluated tasks. It does not prove that an unbounded reasoning/action transcript is safe or generally reliable.

**Outcome:** implement a deterministic loop whose state, transitions, and terminal reason can be replayed and inspected.

### Lesson 12.2 — Tool and Observation Contracts

A production tool contract includes:

- stable name and version, typed arguments, validation, and error schema;
- caller identity, capability scope, preconditions, and approval policy;
- read-only, reversible, idempotent, compensatable, or irreversible effect class;
- deadline, cancellation, retry and idempotency semantics;
- structured result, effect status, postcondition evidence, and provenance.

Natural-language descriptions help model selection but are not an execution contract. Toolformer is evidence that a model can learn decisions about whether, when, and how to invoke scoped APIs; it does not supply runtime permission, transaction, timeout, or retry policy.

Every result becomes an observation envelope containing call ID, tool/version, canonical arguments, timestamps, raw and parsed output, validation result, error class, effect state, truncation, and lineage. Empty, truncated, malformed, stale, or adversarial content must not be observationally equivalent to success.

**Outcome:** reject malformed and unauthorized calls before execution and preserve enough evidence to distinguish output text from real effects.

### Lesson 12.3 — Budgets, Stops, Retry, and Replanning

Let the hard budget vector be

$$
B=(B_{steps},B_{model},B_{tool},B_{tokens},B_{time},B_{cost},B_{repeat},B_{errors}).
$$

Continue only while every consumed resource remains inside policy and no semantic terminal predicate has fired. A step cap bounds one dimension; it neither proves success nor prevents a single expensive or harmful step. Semantic outcomes include verified success, explicit failure, safe abstention, escalation, cancellation, and unknown effect.

Classify a failed call before choosing a response:

| Failure class | Typical next decision |
|---|---|
| Schema/validation | repair arguments or stop |
| Authorization/policy | do not retry unchanged; request approved path |
| Transient transport | bounded backoff/retry when effect is known absent |
| Rate limit | respect server signal and remaining deadline/budget |
| Deterministic application error | change plan or input; blind retry is futile |
| Semantic corruption | validate against invariants or independent evidence |
| Timeout/partial or unknown effect | verify postcondition before any retry |
| Permanently unavailable tool | choose a valid alternative, replan, or escalate |

ToolMaze's 2026 benchmark crosses explicit/implicit with transient/permanent perturbations and reports that anomaly recovery remains distinct from happy-path execution in its setup. Treat it as a frontier failure-injection design, not a universal production estimate.

**Outcome:** map error class and effect certainty to retry, repair, verify, replan, alternative, escalation, or stop.

### Lesson 12.4 — Progress, Cycles, and Reflection

Step count is not progress. Instrument verified subgoals, state hashes/deltas, action signatures, repeated error classes, tool-result novelty, plan similarity, and remaining budget. Repeated actions, alternating states, or nearly identical plans are useful stall signals, but legitimate polling and iterative refinement can look similar.

Treat online stall detection as a hypothesis. Compare a preregistered detector with a step-cap baseline and report early-stop savings, false stops, recovered success, effect errors, and cost. A detector that merely stops hard tasks sooner may reduce spend while destroying utility.

Reflexion is a reference mechanism that stores verbal feedback for later trials. Reflection is not independent evidence: a fluent explanation can preserve a wrong diagnosis. Admit a reflection into state or memory only with its source, outcome, confidence, validity window, and evidence; compare self-reflection with no-reflection, external-feedback, and oracle-feedback baselines.

**Outcome:** detect genuine no-progress and use feedback without turning self-critique into truth.

### Lesson 12.5 — Authority and Effect Verification

The model is not an authorization oracle. Enforce least-privilege credentials, tenant and object scope, rate and value limits, preconditions, previews, required approvals, and prohibited transitions outside the prompt. Untrusted observations cannot grant new authority.

For high-impact actions, separate:

```text
proposed intent -> policy decision -> approved command -> execution
        -> effect receipt -> independent postcondition -> user-visible claim
```

If a timeout occurs after dispatch, the effect may be unknown. Do not equate an absent response with no effect or retry automatically. Module 12 owns the decision to verify, stop, or escalate; Module 14 develops durable idempotency, checkpoint, and compensation mechanics; Module 18 develops adversarial security controls.

**Outcome:** demonstrate that a model cannot widen authority and cannot claim a side effect without verifiable evidence.

### Lesson 12.6 — Trajectory Evaluation and Diagnosis

AgentBench is reference evidence that interactive environments expose heterogeneous failure modes. A single aggregate score cannot establish general agent capability.

For a strictly sequential episode, wall time decomposes into controller, model, validation, tool, observation-processing, queue, and network components. For parallel branches, use critical-path timing; summing overlapping spans overstates elapsed time.

Report by workload and failure slice:

- verified task success, partial completion, abstention, escalation, and rejection;
- side-effect correctness, duplicate/unknown effects, policy violations, and human intervention;
- steps, model/tool calls, retries, tokens, tool/model latency, elapsed time, and cost;
- recovery conditioned on perturbation and on opportunity to recover;
- terminal reason and unfinished episodes.

Define episode goodput under a declared SLO as verified, policy-compliant successes per wall-clock time. Define cost of success with all included episode spend and an explicit zero-success policy. Never drop aborted, rejected, timed-out, or failed episodes merely because they lack a final answer.

**Outcome:** diagnose `SYMPTOM → COMPETING HYPOTHESES → MISSING EVIDENCE → DISCRIMINATING MEASUREMENT → RANKED EXPLANATION → INTERVENTION → REMEASUREMENT` from full trajectories.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [ReAct](https://arxiv.org/abs/2210.03629) — Yao et al., ICLR 2023: interleaved reasoning and actions on scoped tasks.
- [Toolformer](https://arxiv.org/abs/2302.04761) — Schick et al., 2023: learned decisions about API use.
- [Reflexion](https://arxiv.org/abs/2303.11366) — Shinn et al., 2023: verbal feedback retained across trials.
- [AgentBench](https://arxiv.org/abs/2308.03688) — Liu et al., ICLR 2024; arXiv revised 2025: multi-environment interactive evaluation.

**CURRENT DEFAULT:** controller-owned transitions; schema validation before dispatch; least-privilege authorization outside the model; independent call/time/token/cost limits; structured errors; trajectory telemetry; explicit terminal reasons. These are engineering defaults, not a claim that every framework implements them completely.

**WORKLOAD-DEPENDENT:** planner shape, reflection, retry count/backoff, progress metric, verifier, approval placement, degree of autonomy, parallelism, and utility weights.

**FRONTIER:** [ToolMaze](https://arxiv.org/abs/2606.05806) anomaly-recovery evaluation and adaptive recovery under changing tool availability/semantics. Benchmark transfer to real side effects remains open.

**LEGACY / INSUFFICIENT:** parse free-form action text and execute it directly; retry every exception; stop only when the model says “done”; trust reflection as verification; hide failed trajectories; report only final success; use one framework's loop as the definition of agents.

**PRODUCTION SOURCE TRACE**

- Repository: `langchain-ai/langgraph`
- Revision: `7daa3ab49d678a5da75edb08baa87db4a2be52c3`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `libs/prebuilt/langgraph/prebuilt/chat_agent_executor.py::create_react_agent`, `libs/langgraph/langgraph/pregel/main.py::Pregel.stream`, and `libs/langgraph/langgraph/errors.py::GraphRecursionError`.
- Entry path: compiled graph invocation/stream → model node → `AIMessage.tool_calls` → tool node → `ToolMessage` observations → model until no tool calls or another terminal path; Pregel stream checks a configurable recursion limit and raises `GraphRecursionError` when exhausted without a stop.
- Scope: an implementation example. `create_react_agent` is explicitly deprecated in the pinned source in favor of `langchain.agents.create_agent`; the recursion limit is a guardrail, not a semantic-success verifier.

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Deterministic Loop and Budget Guard

- Implement explicit states, legal transitions, semantic terminal predicates, and budgets for steps, calls, tokens, time, cost, repetition, and errors.
- Record a replayable trajectory and expose the terminal reason.
- Break with endless tool calls, alternating plans, repeated identical actions, a single expensive step, false “done,” and legitimate polling.
- Compare a step cap with progress/cycle detection; report false stops and recovered successes.

### LAB B — Tool Contract and Observation Chaos

- Define versioned tools spanning read-only, idempotent write, reversible write, and irreversible effects.
- Inject malformed arguments, permission denial, rate limits, timeout-before-dispatch, timeout-after-dispatch, partial response, corrupted success payload, truncation, stale results, and unknown effect.
- Verify that authority never comes from model text and that the observation envelope preserves effect uncertainty.
- Artifact: contract schemas, transition table, chaos matrix, and effect audit.

### LAB C — Recovery, Replanning, and Reflection

- Implement bounded retry, argument repair, alternative-tool selection, replanning, postcondition verification, safe escalation, and reflection.
- Cross explicit/implicit with transient/permanent failures and simple/complex tool topology.
- Compare no recovery, blind retry, classified recovery, self-reflection, external feedback, and oracle feedback with paired episodes.
- Falsify “more attempts help” and “reflection corrects errors.”

### LAB D — Trajectory Evaluation Under Load

- Evaluate mixed task lengths, tool latencies, side-effect classes, concurrency, and injected failures.
- Measure task/effect correctness, violations, intervention, recovery, steps/calls/tokens, latency distributions, cost, goodput, aborts, and unfinished episodes.
- Separate model, controller, queue, validation, tool, and network spans; use critical paths for overlap.
- Artifact: slice dashboard, oracle-stage replays, ranked incident diagnosis, canary and rollback policy.

## 07 Break / Incident Scenarios

### Incident 12.1 — The Agent Repeats a High-Impact Action

After a release, an agent calls a payment/message/deployment tool, times out, retries, alternates between “verify” and “execute,” consumes its budget, and finally reports success. Some external effects are duplicated; other episodes stopped before a recoverable tool became available.

Competing explanations include a client timeout before dispatch, server timeout after commit, missing idempotency/effect receipt, malformed observation, authorization drift, deterministic application error mislabeled transient, stale read-after-write, progress-detector false positive/negative, model/planner regression, queue delay, or an unavailable alternative path.

Collect call IDs and idempotency keys where supported, dispatch/receipt timestamps, canonical arguments, authorization decisions, tool/version, raw and parsed outputs, effect states, postcondition reads, state deltas, action signatures, budgets, retries/backoff, model/prompt versions, spans, terminal reason, and external audit records. Reconstruct the trajectory; determine effect certainty before replay; patch the earliest failing boundary; remeasure duplicate effects, verified success, false stops, latency, and cost.

## 08 Mastery Assessment

Design an agent controller for a multi-tool operational assistant that can read state, propose a plan, perform governed writes, recover from changing tool behavior, and stop safely. Deliver the state and transition schemas; tool/observation contracts; authority and approval policy; budget vector and semantic stops; failure-to-recovery table; stall detector; reflection experiment; anomaly matrix; trajectory metrics; current source trace; load test; release/kill/rollback plan; and a diagnosis of Incident 12.1.

## 09 Required Evidence & Rubric

- **Control model:** state, legal transitions, owner of each decision, and terminal reasons are explicit.
- **Contracts:** schemas, versions, side-effect class, authority, timeouts, errors, retry safety, and completion evidence are machine-checkable.
- **Bounds:** steps, calls, tokens, time, cost, repetition, and errors are independently enforced.
- **Recovery:** failure and effect certainty determine retry, repair, verification, replan, alternative, escalation, or stop.
- **Progress:** stall/cycle signals are evaluated for both savings and false termination.
- **Authority:** the model cannot expand credentials, bypass approval, or convert untrusted output into permission.
- **Evaluation:** success, effects, violations, intervention, failures, unfinished episodes, latency, tokens, and cost remain visible by slice.
- **Diagnosis:** competing hypotheses are discriminated with trajectories and external effect evidence before intervention.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Bounded state-transition controller | 12.1, 12.3 | LAB A | Incident / Mastery | Replayable trace and budget tests |
| Tool and observation contracts | 12.2 | LAB B | Incident / Mastery | Schema, authority, and chaos results |
| Recovery and no-progress detection | 12.3–12.4 | LAB A, LAB C | Incident / Mastery | Failure matrix and false-stop report |
| Authority and effect verification | 12.5 | LAB B | Incident / Mastery | Policy decisions and effect audit |
| Trajectory evaluation and diagnosis | 12.6 | LAB D | Mastery | Slice dashboard and oracle replays |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to keep proposal, authorization, execution, effect, observation, progress, and termination distinct; bound every episode across multiple resources; refuse blind retries after unknown effects; distinguish syntactic from semantic tool success; falsify progress and reflection mechanisms; trace a pinned runtime without universalizing it; and evaluate complete trajectories including harms, failures, abstentions, unfinished work, latency, and cost.

**Final mental model:** an agent loop is a fallible, resource-bounded control system around a stochastic proposer. Reliability comes from typed transitions, external authority, observable effects, classified recovery, explicit stops, and trajectory-level falsification—not from letting the model continue until its prose sounds complete.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

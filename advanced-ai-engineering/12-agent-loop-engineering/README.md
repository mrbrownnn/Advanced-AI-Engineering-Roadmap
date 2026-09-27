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

**Module Orientation**
- **Engineering Problem**: Maximize verified task utility while bounding invalid effects, policy violations, latency, tokens, calls, and cost.
- **What You Will Do**: Implement and instrument a bounded controller, define tool and observation contracts, inject ambiguous effects and recovery failures, compare progress and reflection mechanisms, trace a pinned LangGraph path, and defend a governed operational-agent design.
- **Environment**: Python 3.10+ for controller and fault-injection harnesses. Sandboxed mock tools and an append-only trajectory store are required; external side effects are optional and must remain disposable or simulated.
- **Evidence Rule**: Label source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**). Model claims about success, failure, or tool completion are not execution evidence.

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

The learner must be able to define a loop as an explicit state machine; design typed tool and observation contracts; separate proposal from authorization; enforce independent resource and semantic stops; classify failures before retrying; detect no-progress and oscillation without blocking legitimate iteration; evaluate reflection against grounded feedback; inject tool anomalies; and diagnose complete trajectories rather than final answers alone.

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

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

**Engineering Question:**
Which state and decisions must remain controller-owned so that an agent episode is bounded, inspectable, and replayable?

**Concepts & Definitions:**
- **Episode**: one bounded attempt to achieve a declared goal.
- **Controller state**: typed state owned by deterministic orchestration rather than model prose.
- **Terminal reason**: verified success, explicit failure, abstention, escalation, cancellation, or budget exhaustion.

**Quantitative Model / Derivation:**
Represent one episode as

$$
\tau=(s_0,a_0,o_1,s_1,\ldots,a_{T-1},o_T,s_T),\qquad
s_{t+1}=F(s_t,a_t,o_{t+1}).
$$

This is exact bookkeeping for a declared state schema. It is not automatically a Markov model: if `s_t` omits relevant history, permissions, hidden environment state, or pending effects, it is not sufficient to predict the next transition.

**Mechanism Explanation:**
The controller—not the model—owns legal states, action validation, authorization, tool dispatch, budget accounting, terminal outcomes, and the trajectory ledger. The model may emit `call_tool`, `respond`, `abstain`, `ask`, or `escalate`; each is a proposal until the controller validates it.

ReAct is a reference pattern for interleaving reasoning and environment actions on evaluated tasks. It does not prove that an unbounded reasoning/action transcript is safe or generally reliable.

**Worked Example:**
Trace `READY → PROPOSED → AUTHORIZED → DISPATCHED → OBSERVED → VERIFIED → SUCCEEDED` for a read-only lookup. Then replace the result with a timeout after dispatch: the legal next state is `EFFECT_UNKNOWN`, not automatic success or an assumption that nothing happened.

**Knowledge Check:**
1. Why is a model-produced “done” message only a proposal?
2. Which omitted variables would make the displayed state insufficient for replay or diagnosis?

**Guided Practice:**
Write a transition table with allowed predecessor states, guards, emitted evidence, and terminal reasons. Include malformed proposal, denied authorization, cancellation, and unknown effect.

**Feedback Contract:**
- *Expected Evidence*: Every transition has one deterministic owner and a recorded guard; no terminal success is reachable from an unverified effect.
- *Common Failure*: Treating the transcript as the complete state or allowing the model to choose its own authorization result.
- *Diagnostic Hint*: Can two different external realities produce the same recorded state?
- *Concept to Revisit*: Controller-Owned State.

**Learning Outcome:**
Implement a deterministic loop whose state, transitions, and terminal reason can be replayed and inspected.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 12.2 — Tool and Observation Contracts

**Engineering Question:**
How can a controller distinguish a syntactically valid call, an authorized execution, and a verified external effect?

**Concepts & Definitions:**
A production tool contract includes:

- stable name and version, typed arguments, validation, and error schema;
- caller identity, capability scope, preconditions, and approval policy;
- read-only, reversible, idempotent, compensatable, or irreversible effect class;
- deadline, cancellation, retry and idempotency semantics;
- structured result, effect status, postcondition evidence, and provenance.

**Mechanism Explanation:**
Natural-language descriptions help model selection but are not an execution contract. Toolformer is evidence that a model can learn decisions about whether, when, and how to invoke scoped APIs; it does not supply runtime permission, transaction, timeout, or retry policy.

Every result becomes an observation envelope containing call ID, tool/version, canonical arguments, timestamps, raw and parsed output, validation result, error class, effect state, truncation, and lineage. Empty, truncated, malformed, stale, or adversarial content must not be observationally equivalent to success.

**Quantitative Model / Derivation:**
For validation predicates $V_{schema}$, $V_{policy}$, and $V_{pre}$, dispatch is permitted only when
$$D=V_{schema}\land V_{policy}\land V_{pre}.$$
This is an exact controller rule for the declared contract; it says nothing about whether the eventual external effect is correct.

**Worked Example:**
A proposal requests `transfer(amount=100, account=B)`. The schema passes, but the caller lacks account scope, so policy validation rejects before dispatch. A second authorized attempt times out after dispatch; the envelope records `effect=unknown` and requires a postcondition query before retry.

**Knowledge Check:**
1. Why does valid JSON not imply authorized execution?
2. Which envelope field distinguishes “no response” from “no effect”?

**Independent Practice:**
Define one read-only, one idempotent-write, and one irreversible tool contract. Specify validation, authority, timeout, cancellation, effect evidence, and retry semantics.

**Feedback Contract:**
- *Expected Evidence*: Contract validation precedes dispatch; the observation preserves raw data, parsed data, effect certainty, and postcondition evidence.
- *Common Failure*: Trusting a success string or assuming a timeout means the write did not commit.
- *Diagnostic Hint*: What independent evidence names the external effect?
- *Concept to Revisit*: Proposal–Authorization–Effect Separation.

**Learning Outcome:**
Reject malformed and unauthorized calls before execution and preserve enough evidence to distinguish output text from real effects.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 12.3 — Budgets, Stops, Retry, and Replanning

**Engineering Question:**
How should an agent stop or recover when different resources, deadlines, and effect-certainty states conflict?

**Concepts & Definitions:**
- **Hard budget**: independently enforced maximum consumption of a named resource.
- **Semantic stop**: a terminal predicate based on verified task state rather than step count alone.
- **Unknown effect**: dispatch occurred but available evidence cannot establish whether the external commit happened.

**Quantitative Model / Derivation:**
Let the hard budget vector be

$$
B=(B_{steps},B_{model},B_{tool},B_{tokens},B_{time},B_{cost},B_{repeat},B_{errors}).
$$

Continue only while every consumed resource remains inside policy and no semantic terminal predicate has fired. A step cap bounds one dimension; it neither proves success nor prevents a single expensive or harmful step. Semantic outcomes include verified success, explicit failure, safe abstention, escalation, cancellation, and unknown effect.

**Mechanism Explanation:**
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

**Worked Example:**
Assume a root deadline of 10 s, a maximum of three tool attempts, and observed attempt durations of 2 s, 3 s, and 4 s with 1 s total backoff. The sequential path consumes the full 10 s; a fourth attempt is illegal even if its local SDK timeout would permit it. If attempt two has unknown effect, verification precedes attempt three.

**Knowledge Check:**
1. Why is retrying a deterministic authorization failure unchanged futile?
2. Why must nested SDK and controller retries share one root budget?

**Guided Practice:**
Map each failure-table row to `repair`, `retry`, `verify`, `replan`, `alternative`, `escalate`, or `stop`, including the evidence required to leave `effect=unknown`.

**Feedback Contract:**
- *Expected Evidence*: The decision uses error class, effect certainty, remaining deadline, and all budget dimensions.
- *Common Failure*: Resetting deadline or attempt count at each layer.
- *Diagnostic Hint*: Is the next action reducing uncertainty or merely repeating work?
- *Concept to Revisit*: Root Budget and Effect Certainty.

**Learning Outcome:**
Map error class and effect certainty to retry, repair, verify, replan, alternative, escalation, or stop.

*(Effort: 40m instruction, 25m practice)*

---

### Lesson 12.4 — Progress, Cycles, and Reflection

**Engineering Question:**
How can a controller stop genuine no-progress without terminating legitimate polling or iterative refinement?

**Concepts & Definitions:**
Step count is not progress. Instrument verified subgoals, state hashes/deltas, action signatures, repeated error classes, tool-result novelty, plan similarity, and remaining budget. Repeated actions, alternating states, or nearly identical plans are useful stall signals, but legitimate polling and iterative refinement can look similar.

**Mechanism Explanation:**
Treat online stall detection as a hypothesis. Compare a preregistered detector with a step-cap baseline and report early-stop savings, false stops, recovered success, effect errors, and cost. A detector that merely stops hard tasks sooner may reduce spend while destroying utility.

Reflexion is a reference mechanism that stores verbal feedback for later trials. Reflection is not independent evidence: a fluent explanation can preserve a wrong diagnosis. Admit a reflection into state or memory only with its source, outcome, confidence, validity window, and evidence; compare self-reflection with no-reflection, external-feedback, and oracle-feedback baselines.

**Quantitative Model / Trade-off Comparison:**
For a detector, report false-stop rate among episodes that the baseline later solves, saved calls among genuinely stalled episodes, and net verified utility after cost. No single count is sufficient because aggressive stopping can improve cost while reducing success.

**Worked Example:**
Compare two traces with action signature `poll(job-7)` repeated four times. In one, the observation version and job progress advance; in the other, the same stale observation repeats. A repetition count alone flags both, while state-delta evidence separates them.

**Knowledge Check:**
1. What evidence distinguishes an oscillation from a valid two-phase protocol?
2. Why is self-reflection not an independent oracle?

**Guided Practice:**
Pre-register a stall detector and compare step-cap, detector-only, self-reflection, external-feedback, and oracle-feedback conditions on paired episodes.

**Feedback Contract:**
- *Expected Evidence*: Report success, false stops, saved work, effect errors, and cost by task/failure slice.
- *Common Failure*: Calling every repeated action a loop or treating reflective prose as verified state.
- *Diagnostic Hint*: Did any externally grounded predicate improve?
- *Concept to Revisit*: Verified Progress.

**Learning Outcome:**
Detect genuine no-progress and use feedback without turning self-critique into truth.

*(Effort: 35m instruction, 30m practice)*

---

### Lesson 12.5 — Authority and Effect Verification

**Engineering Question:**
Where must authorization and postcondition checks sit so that model text cannot widen authority or fabricate completion?

**Concepts & Definitions:**
The model is not an authorization oracle. Enforce least-privilege credentials, tenant and object scope, rate and value limits, preconditions, previews, required approvals, and prohibited transitions outside the prompt. Untrusted observations cannot grant new authority.

**Mechanism Explanation:**
For high-impact actions, separate:

```text
proposed intent -> policy decision -> approved command -> execution
        -> effect receipt -> independent postcondition -> user-visible claim
```

If a timeout occurs after dispatch, the effect may be unknown. Do not equate an absent response with no effect or retry automatically. Module 12 owns the decision to verify, stop, or escalate; Module 14 develops durable idempotency, checkpoint, and compensation mechanics; Module 18 develops adversarial security controls.

**Quantitative Model / Derivation:**
Let $A(c,o,v)$ be the controller policy for caller $c$, object $o$, and proposed value $v$, and let $P(e)$ be an independent postcondition for effect $e$. A success claim is permitted only when dispatch was authorized and $P(e)$ is observed; model confidence is not an input to either predicate.

**Worked Example:**
A deployment proposal passes schema validation but exceeds the caller's environment scope. The controller denies it without sending credentials. For an authorized deployment whose response is lost, a versioned read-back must match the approved artifact before the episode can claim success.

**Knowledge Check:**
1. Can an observation containing “admin approved” grant authority?
2. Why is an effect receipt stronger than generated narration yet sometimes still insufficient?

**Independent Practice:**
Design a preview/approve/execute/read-back protocol with an immutable approval binding and test wrong tenant, changed value, expired approval, and stale postcondition.

**Feedback Contract:**
- *Expected Evidence*: Policy decisions and postconditions are external to the model and bound to exact intent/version.
- *Common Failure*: Reusing an approval after arguments change.
- *Diagnostic Hint*: Which controller record binds caller, object, value, and time?
- *Concept to Revisit*: External Authority Boundary.

**Learning Outcome:**
Demonstrate that a model cannot widen authority and cannot claim a side effect without verifiable evidence.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 12.6 — Trajectory Evaluation and Diagnosis

**Engineering Question:**
Which trajectory-level measurements distinguish a useful recovery mechanism from one that hides failures or amplifies cost?

**Concepts & Definitions:**
AgentBench is reference evidence that interactive environments expose heterogeneous failure modes. A single aggregate score cannot establish general agent capability.

**Quantitative Model / Derivation:**
For a strictly sequential episode, wall time decomposes into controller, model, validation, tool, observation-processing, queue, and network components. For parallel branches, use critical-path timing; summing overlapping spans overstates elapsed time.

Report by workload and failure slice:

- verified task success, partial completion, abstention, escalation, and rejection;
- side-effect correctness, duplicate/unknown effects, policy violations, and human intervention;
- steps, model/tool calls, retries, tokens, tool/model latency, elapsed time, and cost;
- recovery conditioned on perturbation and on opportunity to recover;
- terminal reason and unfinished episodes.

Define episode goodput under a declared SLO as verified, policy-compliant successes per wall-clock time. Define cost of success with all included episode spend and an explicit zero-success policy. Never drop aborted, rejected, timed-out, or failed episodes merely because they lack a final answer.

**Mechanism Explanation:**
Join controller, model, validation, tool, effect, queue, and client spans with stable episode/call/effect identities. Slice by workload, failure injection, authorization class, terminal reason, and recovery opportunity before interpreting an aggregate.

**Worked Example:**
In a 60 s observation window, assume 20 offered episodes, 12 verified policy-compliant successes within the SLO, three failures, two abstentions, one rejection, and two unfinished episodes. Declared episode goodput is $12/60=0.2$ successful episodes/s; it is not $12/15$ and unfinished work remains visible.

**Knowledge Check:**
1. Why can final-answer accuracy hide duplicate effects?
2. When does summing branch latencies overstate end-to-end time?

**Guided Practice:**
Build a trajectory report that preserves all terminal classes and compare a retry policy on paired injected-failure episodes.

**Feedback Contract:**
- *Expected Evidence*: Offered denominator, terminal reason, effect correctness, opportunity-to-recover, critical-path latency, all-attempt tokens/cost, and confidence procedure are explicit.
- *Common Failure*: Reporting only completed episodes or only the final successful attempt.
- *Diagnostic Hint*: Which missing class would make the policy look better?
- *Concept to Revisit*: Trajectory-Level Goodput.

**Learning Outcome:**
Diagnose `SYMPTOM → COMPETING HYPOTHESES → MISSING EVIDENCE → DISCRIMINATING MEASUREMENT → RANKED EXPLANATION → INTERVENTION → REMEASUREMENT` from full trajectories.

*(Effort: 40m instruction, 30m practice)*

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

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Deterministic Loop and Budget Guard

- **Objective**: Implement explicit states, legal transitions, semantic terminal predicates, and independent budgets; record a replayable trajectory and terminal reason.
- **Pre-Registered Hypothesis**: A state-delta/cycle detector will reduce calls on injected no-progress episodes relative to a step-cap baseline without exceeding a declared false-stop tolerance on solvable episodes.
- **Independent Variables**: Stop policy, task topology, progress signal, polling behavior, and budget vector.
- **Dependent Variables**: Verified success, false-stop rate, calls/tokens/time/cost, terminal reason, and replay agreement.
- **Break & Falsify**: Inject endless calls, alternating plans, identical actions, one expensive step, false `done`, and legitimate polling. A detector that saves work only by stopping recoverable episodes falsifies the claimed benefit.
- **Alignment**: Lessons 12.1, 12.3, and 12.4.
- **Effort Estimate**: 2.5h implementation, 0.5h analysis (3h total).

### LAB B — Tool Contract and Observation Chaos

- **Objective**: Define versioned read-only, idempotent-write, reversible-write, and irreversible tool contracts with typed observation envelopes.
- **Pre-Registered Hypothesis**: Contract-first dispatch will block malformed/unauthorized calls and preserve unknown-effect states without converting them into false success.
- **Independent Variables**: Effect class, authorization, response integrity, dispatch timing, timeout, and postcondition availability.
- **Dependent Variables**: Pre-dispatch rejection, effect-state classification, unsafe retries, duplicate effects, and evidence completeness.
- **Break & Falsify**: Inject malformed arguments, permission denial, rate limits, timeout before/after dispatch, partial/corrupt/truncated/stale results, and unknown effects. Any unauthorized dispatch or unverified success falsifies the contract.
- **Alignment**: Lessons 12.2 and 12.5.
- **Effort Estimate**: 2.5h implementation, 0.5h analysis (3h total).

### LAB C — Recovery, Replanning, and Reflection

- **Objective**: Implement classified retry, repair, verification, replanning, alternatives, escalation, and reflection.
- **Pre-Registered Hypothesis**: Classified recovery will improve verified success per unit cost over blind retry on recoverable perturbations; self-reflection will not be assumed equivalent to external feedback.
- **Independent Variables**: Failure explicitness/permanence, tool topology, recovery policy, and feedback source.
- **Dependent Variables**: Verified recovery, unsafe effects, attempts, latency, cost, false success, and escalation.
- **Break & Falsify**: Cross explicit/implicit with transient/permanent failures; compare no recovery, blind retry, classified recovery, self-reflection, external feedback, and oracle feedback. Include a workload where extra attempts reduce utility.
- **Alignment**: Lessons 12.3–12.5.
- **Effort Estimate**: 2.5h implementation, 0.5h analysis (3h total).

### LAB D — Trajectory Evaluation Under Load

- **Objective**: Evaluate mixed task lengths, tool latencies, effect classes, concurrency, and injected failures from complete trajectories.
- **Pre-Registered Hypothesis**: Full offered-episode accounting will expose a different recovery ranking than success-only final-answer scoring on at least one preregistered injected-failure slice.
- **Independent Variables**: Concurrency, task length, tool latency, effect class, failure injection, and recovery policy.
- **Dependent Variables**: Task/effect correctness, violations, intervention, recovery, calls/tokens, critical-path latency, cost, goodput, aborts, and unfinished episodes.
- **Break & Falsify**: Remove failed attempts or sum overlapping spans and measure the induced ranking error; a workload with unchanged ranking limits the claim rather than invalidating the experiment.
- **Alignment**: Lesson 12.6 and Incident 12.1.
- **Effort Estimate**: 2.5h implementation, 0.5h analysis (3h total).

## 07 Break / Incident Scenarios

### Incident 12.1 — The Agent Repeats a High-Impact Action

- **Incident Symptoms**: After a release, an agent calls a payment/message/deployment tool, times out, retries, alternates between `verify` and `execute`, exhausts its budget, and reports success. Some effects are duplicated while some recoverable episodes stop early.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: Include timeout before dispatch, commit before lost acknowledgement, missing idempotency/effect receipt, malformed observation, authorization drift, deterministic error mislabeled transient, stale read-back, progress-detector error, model/planner regression, queue delay, and unavailable alternative path.
  2. *Rank Initial Plausibility*: Use the duplicate-effect audit and timeout location without treating either as conclusive.
  3. *Identify Missing Evidence*: Recover call/effect IDs, canonical arguments, authorization decisions, dispatch/receipt times, raw and parsed outputs, postconditions, state deltas, budgets, retries, versions, spans, terminal reasons, and external audit records.
  4. *Design Discriminating Tests*: Reconcile each effect ID before replay, disable automatic retry, and replay captured observations through old/new controllers. State which result falsifies each leading hypothesis.
  5. *Execute Causal Diagnosis*: Reconstruct complete trajectories and rank the earliest supported failing boundary, including interacting mechanisms.
  6. *Prescribe Mitigation and Prevention*: Freeze retries for unknown effects, reconcile affected operations, then add atomic effect identity, postcondition verification, classified recovery, and safe rollout gates as applicable.
  7. *Remeasure*: Predeclare acceptable duplicate/unknown effects, verified success, false stops, latency, and cost; repeat the same fault injections after the change.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Governed Operations Assistant

Design an agent controller that reads operational state, proposes changes, performs approved writes, tolerates changing tool behavior, and stops safely under a shared latency/cost budget. The workload includes read-only investigation, reversible configuration edits, an irreversible notification, concurrent episodes, and injected unknown-effect timeouts.

**Required Deliverables**:
1. State/transition schemas and explicit owners for proposal, authorization, execution, effect, observation, progress, and termination.
2. Versioned tool/observation contracts with effect classes, deadlines, cancellation, and postconditions.
3. Authority/approval policy plus immutable binding from preview to approved command.
4. Budget vector, semantic stops, and failure-to-recovery decision table.
5. Stall/cycle detector and paired reflection experiment with false-stop analysis.
6. Anomaly matrix, complete trajectory metrics, load/fault results, and source trace.
7. Release, kill, rollback, and reconciliation plan.
8. Evidence-backed diagnosis and remediation of Incident 12.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit a pinned trace of a production agent-loop implementation. For the reference LangGraph revision, map compiled invocation through model and tool nodes, observation return, stop behavior, and recursion-limit failure. Separate static source observation from controller requirements and note the inspected API's deprecation status.

### Rubric Dimensions

- **Control Model**: *Insufficient* relies on transcript/model intent. *Competent* defines legal states, transitions, owners, and terminal reasons. *Strong* proves replay/guard invariants under injected faults.
- **Contracts and Authority**: *Insufficient* validates syntax only. *Competent* enforces typed contracts and external authorization. *Strong* demonstrates preview binding, unknown-effect handling, and independent postconditions.
- **Bounds and Recovery**: *Insufficient* uses one step cap or retries every error. *Competent* enforces independent budgets and classified recovery. *Strong* measures attempt amplification, false recovery, and deadline propagation.
- **Progress and Reflection**: *Insufficient* treats repetition or self-critique as truth. *Competent* evaluates grounded progress and reflection baselines. *Strong* quantifies false stops, saved work, and slice-dependent utility.
- **Evaluation and Diagnosis**: *Insufficient* scores final answers only. *Competent* preserves complete trajectories and competing hypotheses. *Strong* uses discriminating tests, external effect evidence, and remeasurement.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Bounded state-transition controller | 12.1, 12.3 | LAB A | Incident / Mastery | Replayable trace and budget tests |
| Tool and observation contracts | 12.2 | LAB B | Incident / Mastery | Schema, authority, and chaos results |
| Recovery and no-progress detection | 12.3–12.4 | LAB A, LAB C | Incident / Mastery | Failure matrix and false-stop report |
| Authority and effect verification | 12.5 | LAB B | Incident / Mastery | Policy decisions and effect audit |
| Trajectory evaluation and diagnosis | 12.6 | LAB D | Mastery | Slice dashboard and oracle replays |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner successfully completing Module 12 must be able to:
1. Keep proposal, authorization, execution, effect, observation, progress, and termination distinct.
2. Bound every episode across calls, tokens, time, cost, repetition, errors, and semantic stops.
3. Refuse blind retry after unknown effects and prove completion with independent evidence.
4. Falsify a progress detector and reflection mechanism without hiding false stops.
5. Trace a pinned runtime without generalizing one framework into the agent definition.
6. Evaluate complete offered trajectories including harms, failures, abstentions, unfinished work, latency, and cost.

### Module Wrap-Up (Final Mental Model Reconstruction)

- **The Core Invariant**: An agent loop is a fallible, resource-bounded control system around a stochastic proposer.
- **The Control Path**: `state → proposal → validation/authorization → execution → observation/effect verification → progress decision → terminal state or next bounded step`.
- Reliability comes from typed transitions, external authority, observable effects, classified recovery, explicit stops, and trajectory-level falsification—not from letting the model continue until its prose sounds complete.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

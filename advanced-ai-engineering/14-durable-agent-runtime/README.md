# Module 14 — Durable Agent Runtime

## 00 Why This Module Exists

An agent loop that works inside one process is not durable. Workers crash, deployments replace code, queues redeliver work, acknowledgements disappear, humans answer days later, and an external side effect may commit while the runtime still records “unknown.” A durable runtime must recover orchestration state without replaying effects blindly.

```text
workflow input
     |
durable history/checkpoint <---- timers / signals / approvals
     |
deterministic orchestration
     |
activity command + logical effect ID
     |
external system <---- idempotency / fencing / reconciliation
     |
effect receipt or unknown state
     |
history transition / retry / compensate / escalate
```

This module owns workflow history/checkpoints, deterministic replay, activities/effect boundaries, delivery semantics, idempotency, transactional outbox/inbox, sagas and compensation, leases/fencing, fan-out/join, durable waits and HITL, history growth, versioning, and crash recovery. Module 12 owns agent decisions and tool policy; Module 13 owns model-I/O adapters; Module 18 owns threat modeling; Module 23 owns platform-wide reliability operations.

**Research cutoff:** 2026-09-26.

**Module Orientation**
- **Engineering Problem**: Recover long-running work to a correct, explainable state without losing, duplicating, or falsely “rolling back” externally visible effects.
- **What You Will Do**: Build durable history and effect boundaries, crash every commit window, implement idempotency/compensation/fencing, exercise durable joins and approvals, replay old histories, trace Temporal, and defend a recovery architecture.
- **Environment**: Python 3.10+, a durable workflow runtime or deterministic local simulator, transactional storage, disposable external-effect mocks, and fault injection around persistence and acknowledgement boundaries.
- **Evidence Rule**: Distinguish source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**). Runtime marketing terms never replace an explicit persistence and effect contract.

## 01 Baseline Assumptions

- Module 00: invariants, experiments, uncertainty, and falsification.
- Module 04: queues, retries as offered load, saturation, and latency distributions.
- Module 08: identity, lineage, versioning, and immutable event evidence.
- Module 12: state transitions, tool contracts, authorization, retries, and terminal states.
- Module 13: canonical requests, attempt lineage, errors, deadlines, and replay manifests.

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

The learner must be able to state what is durably committed; rebuild state from history; keep nondeterminism and I/O behind recorded boundaries; reason about at-most/at-least/effectively-once behavior; implement atomic idempotency and reconciliation; design honest compensation; fence stale workers; persist joins/timers/approvals; replay-test upgrades; control history growth; and crash the system at every effect boundary.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
                workflow/run identity + code version
                              |
                    persisted ordered history
                              |
                  deterministic state rebuild
                              |
                      command/activity intent
                              |
        +---------------- effect boundary ----------------+
        | operation key | attempt | auth | deadline       |
        +-------------------------------------------------+
                              |
                 external effect + receipt/postcondition
                              |
         committed | absent | failed | partial | unknown
                              |
             continue | retry | compensate | escalate
```

Keep four identities separate: workflow/run, logical transition, activity attempt, and external effect. A retry creates a new attempt but should not invent a new logical effect.

## 04 Lessons

### Lesson 14.1 — Durable State and Deterministic Replay

**Engineering Question:**
What must be durably acknowledged so replay reconstructs the same orchestration decisions without pretending to reconstruct external reality?

**Concepts & Definitions:**
- **Durable prefix**: ordered history/checkpoint acknowledged by the persistence contract.
- **Deterministic replay**: rebuilding state and commands from recorded inputs.
- **Replay divergence**: candidate code emits commands inconsistent with retained history.

**Mechanism Explanation:**

For ordered persisted events `e_1...e_n` and a deterministic reducer `F`:

$$
s_n=\operatorname{fold}(F,s_0,[e_1,\ldots,e_n]).
$$

This reconstructs only state represented by the retained history and schemas. It does not reconstruct an external payment, message, deployment, or model response unless that observation/effect evidence was recorded.

Replay runtimes compare commands produced by current workflow code with recorded history. Temporal and Azure Durable Functions both document deterministic orchestrator constraints. Wall clocks, unrecorded random values, mutable environment/configuration, direct network/database/model calls, unordered iteration, and command-changing code edits can diverge.

Declare the persistence acknowledgement: when may a client believe a workflow start, transition, signal, approval, or cancellation is durable? Crash immediately before and after that point.

**Quantitative Model / Derivation:**
For retained events, $s_n=fold(F,s_0,[e_1,\ldots,e_n])$. This is exact only for state represented by the history and schemas; external effects require separate evidence.

**Worked Example:**
Crash immediately before and after the persistence acknowledgement for workflow start and one transition. State which client response is allowed and which durable prefix recovery can observe.

**Knowledge Check:**
1. Why can unrecorded wall time or randomness break replay?
2. Does replay prove an external payment exists?

**Guided Practice:**
Construct a four-event history, replay it under unchanged code, then inject a command-reordering revision and record the first divergence.

**Feedback Contract:**
- *Expected Evidence*: Acknowledgement boundary, retained inputs, reducer/command trace, and external-state exclusions.
- *Common Failure*: Calling a chat transcript a complete checkpoint.
- *Diagnostic Hint*: Which nondeterministic value was not recorded?
- *Concept to Revisit*: Durable Prefix.

**Learning Outcome:**
Rebuild the same workflow state and command sequence from a durable prefix while identifying all state that lives outside it.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 14.2 — Activity Boundaries, Delivery, and Idempotency

**Engineering Question:**
How should recovery act when an external commit may have succeeded but durable acknowledgement was lost?

**Concepts & Definitions:**
- **Activity/effect boundary**: recorded orchestration command around nondeterministic I/O.
- **Logical effect ID**: stable identity shared by delivery attempts.
- **Atomic idempotency**: receiver stores identity, canonical intent, and outcome with the effect.

**Mechanism Explanation:**

Put nondeterministic external I/O behind activities/effects. Record canonical intent, logical-operation key, attempt, authorization, dispatch, timeout/cancellation, response, effect state, receipt/postcondition, and reconciliation.

The critical ambiguity is:

```text
external commit succeeds
        |
worker/runtime crashes before durable acknowledgement
        |
recovery sees pending/timeout, while external state may already be changed
```

Replay cannot solve this. If the external system cannot participate in the same atomic transaction, safe recovery needs a stable operation identity and atomic deduplication or a trustworthy postcondition query. For protected state `x` and operation key `k`, the required idempotency property is:

$$
apply(apply(x,k),k)=apply(x,k).
$$

Test it under concurrent duplicates and key reuse with different intent. A header/key is not enough unless the receiver stores key + canonical intent + outcome atomically and defines scope and retention.

A transactional outbox stores domain change and publish intent in one local transaction. The relay can still publish twice after a crash; consumers need deduplication/inbox semantics. Track outbox lag, poison rows, ordering, and retention.

**Quantitative Model / Derivation:**
The required idempotency property is $apply(apply(x,k),k)=apply(x,k)$ for the same key and canonical intent. A receiver must reject key reuse with different intent.

**Worked Example:**
Enumerate crash windows: before dispatch, after dispatch/before commit, after external commit/before durable acknowledgement, and after acknowledgement. Only the first and last are unambiguous without receiver evidence.

**Knowledge Check:**
1. Why is an idempotency header alone insufficient?
2. Why can an outbox relay still publish twice?

**Guided Practice:**
Race two concurrent attempts with the same key and then reuse the key with different intent; verify one stored outcome and one conflict.

**Feedback Contract:**
- *Expected Evidence*: Stable logical ID, attempt IDs, atomic receiver record, receipt/postcondition, and reconciliation path.
- *Common Failure*: Check-then-act deduplication or retry after unknown commit.
- *Diagnostic Hint*: Which system owns the atomic decision?
- *Concept to Revisit*: Effect Convergence.

**Learning Outcome:**
Distinguish at-most-once, at-least-once, and effect convergence without claiming universal exactly-once execution.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 14.3 — Compensation and Concurrent Ownership

**Engineering Question:**
When can compensation restore a business invariant, and how does fencing prevent stale owners from creating later effects?

**Concepts & Definitions:**
- **Compensation**: authorized forward action responding to an earlier effect.
- **Lease**: time-bounded ownership hint.
- **Fencing token**: monotonically increasing epoch enforced by the protected resource.

**Mechanism Explanation:**

The 1987 Sagas paper is the reference model: decompose long-lived work into subtransactions with compensating transactions. In production, compensation is another forward action. It has authorization, retries, failures, cost, and evidence; it may restore a business invariant but cannot unread a message, retract a disclosure, erase an observed deployment, or reverse every real-world consequence.

Register compensation before or atomically with the effect intent, preserve dependency order, and expose states such as `compensation_pending`, `compensation_failed`, `partially_compensated`, and `manual_recovery`.

Leases alone do not revoke a paused stale worker. If ownership epoch `q` increases on takeover, the protected resource must reject a write older than its last accepted epoch. Fencing prevents later stale writes only where enforced; it cannot undo an already accepted effect.

**Quantitative Model / Trade-off Comparison:**
A resource with last accepted epoch $q_{last}$ accepts a write only when $q\ge q_{last}$ under its declared rule. This prevents later stale writes where enforced; it cannot undo prior effects.

**Worked Example:**
A saga reserves inventory, charges payment, then fails shipment. Draw dependency-aware compensation and mark notification as irreversible. Pause worker epoch 7, take over with epoch 8, then show the protected resource rejecting epoch 7.

**Knowledge Check:**
1. Why is compensation not database rollback?
2. What failure remains if a downstream resource ignores fencing tokens?

**Guided Practice:**
Implement `compensation_pending`, `failed`, `partially_compensated`, and `manual_recovery` paths plus a fenced/unfenced stale-worker test.

**Feedback Contract:**
- *Expected Evidence*: Compensation dependency order, authorization/evidence, irreversible cases, and resource-enforced epochs.
- *Common Failure*: Registering compensation after an untracked effect.
- *Diagnostic Hint*: Can a paused worker still reach the resource?
- *Concept to Revisit*: Forward Recovery and Fencing.

**Learning Outcome:**
Implement saga recovery and worker ownership without using “rollback” or “lock” more strongly than the mechanism supports.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 14.4 — Fan-Out, Joins, Timers, and Human Decisions

**Engineering Question:**
Which membership, deadline, cancellation, and identity state must persist for parallel and human-gated work to recover correctly?

**Concepts & Definitions:**
- **Durable membership**: expected child set recorded before or with dispatch.
- **Join predicate**: all-of, quorum, any-of, or partial-success rule.
- **Durable wait**: timer/event/approval persisted without occupying worker memory.

**Mechanism Explanation:**

For fan-out, persist the expected child set before/with dispatch, stable child IDs, attempts, per-child outcome, join predicate, deadline, cancellation, and duplicate/late handling. An all-of join's latency is at least the slowest critical child plus scheduling/join overhead; total work sums all attempts. Quorum and any-of change success/cancel policy, not the need for durable membership.

Long waits must not occupy worker threads or depend on process memory. Persist timer/event/approval identity, correlation, authorization, payload/schema version, deadline, cancellation, and supersession. Authenticate the human and bind the decision to the exact action preview/version; reject duplicate, expired, or late approval according to policy.

Measure workflow-history and payload growth. High fan-out, frequent signals, verbose model/tool payloads, and repeated retries can make replay and storage expensive. Use object references, child partitions, compaction/snapshots, or continue-as-new semantics only after documenting changed lineage, atomicity, cancellation, and query behavior.

**Quantitative Model / Derivation:**
For parallel child durations $T_i$, all-of latency is at least $\max_i T_i$ plus scheduling/join overhead, while total attempted work sums child attempts. History-size planning must account for membership, retries, signals, and payload references.

**Worked Example:**
Fan out to five children with one timeout and one duplicate completion. Compare all-of and quorum outcomes, then process an approval arriving after cancellation and show the declared rejection rule.

**Knowledge Check:**
1. Why must membership persist before join evaluation?
2. Does quorum remove the need to handle late children?

**Guided Practice:**
Estimate history entries for a declared fan-out/retry workload, then compare embedded payloads with object references and state the changed failure modes.

**Feedback Contract:**
- *Expected Evidence*: Stable child/approval IDs, membership, join/deadline/cancel policy, late-event handling, and measured history growth.
- *Common Failure*: Reconstructing membership from whatever children happen to return.
- *Diagnostic Hint*: Can recovery distinguish missing from never-scheduled children?
- *Concept to Revisit*: Durable Coordination.

**Learning Outcome:**
Recover parallel and human-gated work with explicit partial, timeout, and late-event semantics.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 14.5 — Recovery Telemetry and Crash Falsification

**Engineering Question:**
Which correlated telemetry and fault boundaries localize a recovery defect without guessing from a final workflow state?

**Concepts & Definitions:**
- **Crash matrix**: controlled before/after failures around each persistence/effect boundary.
- **Recovery oracle**: declared expected final state and external effects.
- **Effect ledger**: correlation of logical operation, attempts, receipts, and reconciliation.

**Mechanism Explanation:**

Observe separately:

- workflow/run/type/code version, history sequence/size, replay count/failure, and terminal state;
- workflow-task and activity-task queue delay, attempts, heartbeat/progress, timeout, cancellation, and retry;
- logical operation, external effect state, idempotency hit/conflict, receipt/postcondition, compensation, and reconciliation;
- timers/signals/approvals, duplicate/late/rejected events, fan-out membership, and join status;
- final business outcome, latency, cost, duplicate/lost effects, and manual intervention.

Build a crash matrix around every boundary: before/after history append, task delivery, external dispatch, external commit, receipt write, activity completion, outbox publish/mark, compensation, timer firing, signal/approval handling, lease expiry/takeover, and join completion.

The claim that boundary-targeted injection finds more failures than generic restart tests is an **H**. Compare unique actionable failures, escaped incidents, test cost, and oracle divergence. Use a no-failure oracle for final state/effects, while acknowledging that irreversible real-world effects may require a sandbox.

**Quantitative Model / Trade-off Comparison:**
Report recovery-time distribution, duplicate/lost/unknown effects, stuck workflows, replay failures, compensation outcomes, and manual interventions by injected boundary. Counts without opportunity denominators cannot compare fault coverage.

**Worked Example:**
Inject a crash after external commit but before activity completion. A workflow-only dashboard shows pending work; the external ledger shows the effect. Correlating the logical effect ID identifies reconciliation rather than blind retry.

**Knowledge Check:**
1. Which signal separates queue delay from replay failure?
2. Why can a generic process restart miss a narrow acknowledgement window?

**Guided Practice:**
Build and execute a boundary matrix from history append through compensation and join completion, using a no-failure oracle where safe.

**Feedback Contract:**
- *Expected Evidence*: Fault timing, opportunity count, correlated workflow/activity/effect IDs, oracle delta, and cleanup.
- *Common Failure*: Declaring recovery correct because the workflow eventually closed.
- *Diagnostic Hint*: What did the external system commit?
- *Concept to Revisit*: Boundary-Targeted Falsification.

**Learning Outcome:**
Localize a recovery defect to persistence, replay, scheduling, activity, external effect, compensation, or coordination.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 14.6 — Versioning and Replay-Safe Deployment

**Engineering Question:**
How can code evolve while retained histories continue to replay and in-flight work preserves its original semantics?

**Concepts & Definitions:**
- **Command-affecting change**: code/config revision that alters history consumption or emitted commands.
- **Replay corpus**: representative retained histories for every active version/state.
- **Version routing**: explicit mapping from histories/new starts to compatible code.

**Mechanism Explanation:**

Long-running executions cross software releases. Command-affecting changes—reordering activities/timers, changing identities/types, altering branches, or consuming history differently—can break replay. Use runtime-specific patch/version routing and replay representative retained histories under candidate code before canary.

At Temporal Python SDK revision `eb642b14947bd8bcdf8816cffb6a63869803f5c6`, public workflow `start_activity`/`execute_activity` helpers delegate to the current runtime's `workflow_start_activity`; `Replayer.replay_workflow` and `replay_workflows` consume histories and surface replay failures, including nondeterminism. This is a source-reading example, not a universal workflow architecture.

Test histories from every active version and branch, including timers waiting, activities pending/retrying, compensation in progress, approvals pending, and history near limits. Define how old workers/code remain available, how new starts route, and when histories/data can be retired.

**Quantitative Model / Derivation:**
Capacity planning must measure retained-history count/size, replay latency, task queue delay, and recovery concurrency. These are empirical inputs; one average history cannot bound a heavy-tailed active population.

**Worked Example:**
A candidate revision moves a timer before an activity. Replay a history that recorded the old activity first; the mismatch blocks release until patch/version routing preserves the old command sequence.

**Knowledge Check:**
1. Why must pending timers and compensations appear in the replay corpus?
2. When may old workers be retired?

**Independent Practice:**
Replay every active branch/version including near-limit histories, then specify canary, routing, rollback, and data-retirement conditions.

**Feedback Contract:**
- *Expected Evidence*: History inventory, replay results, compatibility matrix, measured replay cost, and old-code retention plan.
- *Common Failure*: Testing only newly started workflows.
- *Diagnostic Hint*: Which recorded command does candidate code emit differently?
- *Concept to Revisit*: Replay-Safe Evolution.

**Learning Outcome:**
Deploy workflow changes without stranding or silently changing in-flight executions.

*(Effort: 35m instruction, 25m practice)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [SAGAS](https://www.cs.princeton.edu/techreports/1987/070.pdf) — Garcia-Molina and Salem, 1987.
- [Temporal Workflow Definition](https://docs.temporal.io/workflow-definition) — replay, deterministic constraints, activities, and versioning.
- [Azure Durable orchestrator code constraints](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-code-constraints) — independent replay/determinism implementation family.

**CURRENT DEFAULT:** durable transition history/checkpoints; deterministic replay where the runtime uses it; explicit activity/effect boundaries; stable operation identities; atomic endpoint deduplication or reconciliation; bounded retries; durable timers/events; version/replay tests; state/activity/effect telemetry.

**WORKLOAD-DEPENDENT:** event sourcing versus checkpointing, local versus remote activities, history partitioning, saga compensation, heartbeat interval, lease duration, fan-out topology, approval placement, and storage/retention.

**FRONTIER:** durable integrations for AI agent frameworks and model/tool workflows, replay-safe AI telemetry, and richer durable event streams. Adoption by one runtime does not establish universal semantics.

**LEGACY / INSUFFICIENT:** save the chat transcript and call it a checkpoint; resume from a step number without effect evidence; assume a timeout means no commit; add an idempotency key without atomic receiver support; call compensation rollback; rely on an unfenced lease; block a worker during HITL; upgrade workflow code without replay tests.

**PRODUCTION SOURCE TRACE**

- Repository: `temporalio/sdk-python`
- Revision: `eb642b14947bd8bcdf8816cffb6a63869803f5c6`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `temporalio/workflow/_activities.py::{start_activity,execute_activity}` and `temporalio/worker/_replayer.py::{Replayer.replay_workflow,Replayer.replay_workflows,Replayer._workflow_replay_iterator}`.
- Execution paths: workflow activity helper → current runtime `workflow_start_activity` → handle/result; history iterator → replay worker → eviction/result hook → replay result → raise/aggregate failure.
- Scope: current Python SDK snapshot. Server/bridge behavior and recovery were not executed; other runtimes implement durability differently.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — History, Replay, and Crash Recovery

- **Objective**: Build deterministic durable state reconstruction and crash every persistence/acknowledgement boundary.
- **Pre-Registered Hypothesis**: Recorded nondeterministic inputs and replay tests will detect command divergence missed by restart-only tests on the seeded revisions.
- **Independent Variables**: Persistence boundary, nondeterministic input, code revision, and crash timing.
- **Dependent Variables**: Replay agreement/failure, recovered state, command sequence, recovery time, and stuck workflows.

- Implement a stateful workflow with recorded commands and separate activities; build a deterministic reducer or use a durable runtime.
- Inject clocks, randomness, mutable config, unordered iteration, direct I/O, and command-changing code revisions.
- Crash before/after transition acknowledgement, task delivery, activity schedule/result, and checkpoint/history writes.
- Artifact: state reconstruction proof, replay corpus, nondeterminism report, and recovery-time distribution.
- **Break & Falsify**: Inject unrecorded clock/random/config/direct I/O and command reorder; any silent divergent command falsifies replay safety.
- **Alignment**: Lesson 14.1.
- **Effort Estimate**: 3h total.

### LAB B — Effect Safety and Compensation

- **Objective**: Implement atomic receiver idempotency, reconciliation, outbox/inbox, and dependency-aware compensation.
- **Pre-Registered Hypothesis**: Atomic key+intent+outcome storage will prevent duplicate receiver effects under concurrent duplicate delivery where check-then-act does not.
- **Independent Variables**: Deduplication design, crash window, concurrency, retention, key conflict, and compensation failure.
- **Dependent Variables**: Duplicate/lost/unknown effects, conflicts, reconciliation time, compensation state, and manual work.

- Build an endpoint with atomic idempotency key + intent hash + stored outcome; contrast a check-then-act implementation.
- Inject concurrent duplicates, crash after commit/before acknowledgement, retention expiry, key conflict, timeout, partial effect, stale read, outbox relay duplicate, and poison message.
- Add dependency-aware compensation and irreversible/manual-recovery cases.
- Artifact: effect ledger, duplicate/loss matrix, reconciliation procedure, and compensation evidence.
- **Break & Falsify**: Crash after commit/before ack, expire keys, poison relay, reuse keys, and fail compensation; any duplicate effect under supported idempotency scope falsifies the design.
- **Alignment**: Lessons 14.2–14.3.
- **Effort Estimate**: 3h total.

### LAB C — Parallelism, Fencing, and Durable HITL

- **Objective**: Recover fan-out/join, stale-worker takeover, and authenticated long-lived approvals.
- **Pre-Registered Hypothesis**: Resource-enforced epochs will reject resumed stale writers; durable membership and approval binding will preserve declared outcomes after restart.
- **Independent Variables**: Join policy, child outcome/order, lease epoch, fencing enforcement, approval timing/version, and restart.
- **Dependent Variables**: Join correctness, stale-write acceptance, approval rejection, history growth, latency, and intervention.

- Persist fan-out membership and implement all-of, quorum, any-of, partial-success, timeout, and cancellation joins.
- Pause a worker beyond lease expiry, issue a new epoch, then resume the stale worker against fenced and unfenced resources.
- Wait durably for an authenticated approval; inject duplicate, late, expired, wrong-version, and unauthorized decisions.
- Artifact: join state machine, fencing proof, approval audit, and history-growth curve.
- **Break & Falsify**: Inject duplicate/late children, timeout/cancel, stale workers, and wrong/late/expired approvals; acceptance of a stale or misbound action falsifies safety.
- **Alignment**: Lessons 14.3–14.4.
- **Effort Estimate**: 3h total.

### LAB D — Upgrade and Recovery Release Gate

- **Objective**: Replay representative retained histories and compare boundary-targeted crashes with generic restarts.
- **Pre-Registered Hypothesis**: Boundary-targeted injection will find at least one actionable seeded recovery defect not exposed by generic restart under the same budget.
- **Independent Variables**: History version/state/size, candidate code, injection strategy, queue load, and routing policy.
- **Dependent Variables**: Replay failures, unique defects, recovery time, queue delay, effect convergence, compensation, cost, and intervention.

- Replay representative histories under candidate code, including every active version/state and near-limit histories.
- Measure history/payload growth, replay latency, task/activity attempts, queueing, recovery time, external effect convergence, compensation, and manual intervention.
- Compare boundary-targeted crash injection with generic process restarts using a preregistered oracle.
- Artifact: compatibility matrix, crash coverage, canary/rollback/routing plan, and source trace.
- **Break & Falsify**: Include all active states and near-limit histories; a candidate that strands any supported history fails the gate.
- **Alignment**: Lessons 14.5–14.6 and Incident 14.1.
- **Effort Estimate**: 3h total.

## 07 Break / Incident Scenarios

### Incident 14.1 — Resumed Agent Duplicates an Irreversible Action

- **Incident Symptoms**: After a timeout and replay, an external action duplicates, compensation fails, a late approval is applied, a stale worker writes after takeover, and older histories fail after rollout.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: Lost acknowledgement, non-atomic/expired idempotency, stale read-back, nested retry, outbox duplicate, compensation order, approval binding, unfenced lease, command drift, history pressure, or queue saturation.
  2. *Rank Initial Plausibility*: Use external duplicate and replay evidence without assuming one root cause.
  3. *Identify Missing Evidence*: Recover workflow/history, task attempts, logical operation/intent, receiver records, dispatch/receipt/postconditions, external audit, outbox/inbox, compensation, approval, epochs, replay failure, history size, queue spans, and business outcome.
  4. *Design Discriminating Tests*: Freeze unknown-effect retry, reconcile by logical ID, replay old histories, and inject each crash window with predicted/falsifying outcomes.
  5. *Execute Causal Diagnosis*: Rank the earliest supported boundary and interacting recovery failures.
  6. *Prescribe Mitigation and Prevention*: Reconcile first; then repair atomic identity, fencing, approval/version policy, replay routing, and history controls as evidence requires.
  7. *Remeasure*: Duplicate/lost/unknown effects, stuck rate, replay failures, recovery latency, compensation, and manual work.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Durable Approval and Fulfillment Workflow

Design a workflow that calls models/tools, performs governed writes, fans out fulfillment, waits days for approval, compensates partial success, and crosses worker and code deployments.

**Required Deliverables**:
1. History/state schemas, acknowledgement contract, and replay constraints.
2. Activity/effect ledger plus delivery, idempotency, and reconciliation proof.
3. Outbox/inbox path, saga graph, irreversible/manual-recovery policy, and fencing protocol.
4. Join, cancellation, timer, and authenticated HITL state machines.
5. History/replay capacity bounds and recovery objectives.
6. Representative replay corpus, crash matrix, and pinned source trace.
7. Canary/version-routing/rollback plan and diagnosis of Incident 14.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit a pinned trace from workflow activity helper through runtime scheduling and from retained history through replay failure reporting. State which server/bridge and real recovery behaviors remain unexecuted.

### Rubric Dimensions

- **Durability and Replay**: *Insufficient* saves a transcript/step. *Competent* defines acknowledgements and deterministic replay. *Strong* crash-tests durable prefixes and every active code path.
- **Effects and Recovery**: *Insufficient* assumes timeout means no effect. *Competent* uses stable effect identity and reconciliation. *Strong* proves concurrent idempotency, conflict, retention, compensation, and unknown-commit behavior.
- **Coordination and Capacity**: *Insufficient* keeps joins/approvals in memory. *Competent* persists membership, timers, approvals, and fencing. *Strong* measures history/replay growth and failure behavior at limits.
- **Diagnosis and Evolution**: *Insufficient* reports final status only. *Competent* correlates workflow/activity/effect evidence. *Strong* uses discriminating crash tests, replay corpora, routing, and remeasurement.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Durable history and deterministic replay | 14.1 | LAB A | Incident / Mastery | Replay corpus and crash report |
| Effect safety and idempotency | 14.2 | LAB B | Incident / Mastery | Effect ledger and duplicate/loss tests |
| Compensation and fencing | 14.3 | LAB B, LAB C | Incident / Mastery | Saga outcomes and stale-writer proof |
| Fan-out and durable HITL | 14.4 | LAB C | Incident / Mastery | Join/approval state machines |
| Recovery diagnosis and upgrades | 14.5–14.6 | LAB D | Incident / Mastery | Crash coverage and compatibility matrix |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner successfully completing Module 14 must be able to:
1. Reconstruct state from a declared durable prefix and control nondeterminism.
2. Refuse exactly-once claims without atomic receiver evidence.
3. Survive duplicate/concurrent delivery and reconcile unknown effects.
4. Model compensation honestly and fence stale owners.
5. Persist joins, timers, cancellations, and approval identity/version.
6. Bound history/replay growth and replay-test every active version.
7. Diagnose crash windows from correlated workflow/activity/effect evidence.

### Module Wrap-Up (Final Mental Model Reconstruction)

- **The Core Invariant**: A durable runtime remembers decisions, while external reality has independent commit points.
- **The Recovery Path**: history → deterministic state/command → activity attempt → external effect evidence → commit, reconcile, compensate, or escalate.
- Correctness comes from history plus effect identity, atomic idempotency or reconciliation, fencing, explicit compensation, durable coordination, and adversarial crash testing—not from replay alone.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

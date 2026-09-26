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

- **Engineering problem:** recover long-running work to a correct, explainable state without losing, duplicating, or falsely “rolling back” externally visible effects.
- **Evidence rule:** distinguish source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**). Runtime marketing terms never replace an explicit persistence and effect contract.

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
  falsification: REQUIRED
```

The learner must be able to state what is durably committed; rebuild state from history; keep nondeterminism and I/O behind recorded boundaries; reason about at-most/at-least/effectively-once behavior; implement atomic idempotency and reconciliation; design honest compensation; fence stale workers; persist joins/timers/approvals; replay-test upgrades; control history growth; and crash the system at every effect boundary.

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

For ordered persisted events `e_1...e_n` and a deterministic reducer `F`:

$$
s_n=\operatorname{fold}(F,s_0,[e_1,\ldots,e_n]).
$$

This reconstructs only state represented by the retained history and schemas. It does not reconstruct an external payment, message, deployment, or model response unless that observation/effect evidence was recorded.

Replay runtimes compare commands produced by current workflow code with recorded history. Temporal and Azure Durable Functions both document deterministic orchestrator constraints. Wall clocks, unrecorded random values, mutable environment/configuration, direct network/database/model calls, unordered iteration, and command-changing code edits can diverge.

Declare the persistence acknowledgement: when may a client believe a workflow start, transition, signal, approval, or cancellation is durable? Crash immediately before and after that point.

**Outcome:** rebuild the same workflow state and command sequence from a durable prefix while identifying all state that lives outside it.

### Lesson 14.2 — Activity Boundaries, Delivery, and Idempotency

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

**Outcome:** distinguish at-most-once, at-least-once, and effect convergence without claiming universal exactly-once execution.

### Lesson 14.3 — Compensation and Concurrent Ownership

The 1987 Sagas paper is the reference model: decompose long-lived work into subtransactions with compensating transactions. In production, compensation is another forward action. It has authorization, retries, failures, cost, and evidence; it may restore a business invariant but cannot unread a message, retract a disclosure, erase an observed deployment, or reverse every real-world consequence.

Register compensation before or atomically with the effect intent, preserve dependency order, and expose states such as `compensation_pending`, `compensation_failed`, `partially_compensated`, and `manual_recovery`.

Leases alone do not revoke a paused stale worker. If ownership epoch `q` increases on takeover, the protected resource must reject a write older than its last accepted epoch. Fencing prevents later stale writes only where enforced; it cannot undo an already accepted effect.

**Outcome:** implement saga recovery and worker ownership without using “rollback” or “lock” more strongly than the mechanism supports.

### Lesson 14.4 — Fan-Out, Joins, Timers, and Human Decisions

For fan-out, persist the expected child set before/with dispatch, stable child IDs, attempts, per-child outcome, join predicate, deadline, cancellation, and duplicate/late handling. An all-of join's latency is at least the slowest critical child plus scheduling/join overhead; total work sums all attempts. Quorum and any-of change success/cancel policy, not the need for durable membership.

Long waits must not occupy worker threads or depend on process memory. Persist timer/event/approval identity, correlation, authorization, payload/schema version, deadline, cancellation, and supersession. Authenticate the human and bind the decision to the exact action preview/version; reject duplicate, expired, or late approval according to policy.

Measure workflow-history and payload growth. High fan-out, frequent signals, verbose model/tool payloads, and repeated retries can make replay and storage expensive. Use object references, child partitions, compaction/snapshots, or continue-as-new semantics only after documenting changed lineage, atomicity, cancellation, and query behavior.

**Outcome:** recover parallel and human-gated work with explicit partial, timeout, and late-event semantics.

### Lesson 14.5 — Recovery Telemetry and Crash Falsification

Observe separately:

- workflow/run/type/code version, history sequence/size, replay count/failure, and terminal state;
- workflow-task and activity-task queue delay, attempts, heartbeat/progress, timeout, cancellation, and retry;
- logical operation, external effect state, idempotency hit/conflict, receipt/postcondition, compensation, and reconciliation;
- timers/signals/approvals, duplicate/late/rejected events, fan-out membership, and join status;
- final business outcome, latency, cost, duplicate/lost effects, and manual intervention.

Build a crash matrix around every boundary: before/after history append, task delivery, external dispatch, external commit, receipt write, activity completion, outbox publish/mark, compensation, timer firing, signal/approval handling, lease expiry/takeover, and join completion.

The claim that boundary-targeted injection finds more failures than generic restart tests is an **H**. Compare unique actionable failures, escaped incidents, test cost, and oracle divergence. Use a no-failure oracle for final state/effects, while acknowledging that irreversible real-world effects may require a sandbox.

**Outcome:** localize a recovery defect to persistence, replay, scheduling, activity, external effect, compensation, or coordination.

### Lesson 14.6 — Versioning and Replay-Safe Deployment

Long-running executions cross software releases. Command-affecting changes—reordering activities/timers, changing identities/types, altering branches, or consuming history differently—can break replay. Use runtime-specific patch/version routing and replay representative retained histories under candidate code before canary.

At Temporal Python SDK revision `eb642b14947bd8bcdf8816cffb6a63869803f5c6`, public workflow `start_activity`/`execute_activity` helpers delegate to the current runtime's `workflow_start_activity`; `Replayer.replay_workflow` and `replay_workflows` consume histories and surface replay failures, including nondeterminism. This is a source-reading example, not a universal workflow architecture.

Test histories from every active version and branch, including timers waiting, activities pending/retrying, compensation in progress, approvals pending, and history near limits. Define how old workers/code remain available, how new starts route, and when histories/data can be retired.

**Outcome:** deploy workflow changes without stranding or silently changing in-flight executions.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — History, Replay, and Crash Recovery

- Implement a stateful workflow with recorded commands and separate activities; build a deterministic reducer or use a durable runtime.
- Inject clocks, randomness, mutable config, unordered iteration, direct I/O, and command-changing code revisions.
- Crash before/after transition acknowledgement, task delivery, activity schedule/result, and checkpoint/history writes.
- Artifact: state reconstruction proof, replay corpus, nondeterminism report, and recovery-time distribution.

### LAB B — Effect Safety and Compensation

- Build an endpoint with atomic idempotency key + intent hash + stored outcome; contrast a check-then-act implementation.
- Inject concurrent duplicates, crash after commit/before acknowledgement, retention expiry, key conflict, timeout, partial effect, stale read, outbox relay duplicate, and poison message.
- Add dependency-aware compensation and irreversible/manual-recovery cases.
- Artifact: effect ledger, duplicate/loss matrix, reconciliation procedure, and compensation evidence.

### LAB C — Parallelism, Fencing, and Durable HITL

- Persist fan-out membership and implement all-of, quorum, any-of, partial-success, timeout, and cancellation joins.
- Pause a worker beyond lease expiry, issue a new epoch, then resume the stale worker against fenced and unfenced resources.
- Wait durably for an authenticated approval; inject duplicate, late, expired, wrong-version, and unauthorized decisions.
- Artifact: join state machine, fencing proof, approval audit, and history-growth curve.

### LAB D — Upgrade and Recovery Release Gate

- Replay representative histories under candidate code, including every active version/state and near-limit histories.
- Measure history/payload growth, replay latency, task/activity attempts, queueing, recovery time, external effect convergence, compensation, and manual intervention.
- Compare boundary-targeted crash injection with generic process restarts using a preregistered oracle.
- Artifact: compatibility matrix, crash coverage, canary/rollback/routing plan, and source trace.

## 07 Break / Incident Scenarios

### Incident 14.1 — Resumed Agent Duplicates an Irreversible Action

A worker times out after dispatching an external action. Recovery replays the workflow, retries the activity, and reports success. The external system shows duplicates; one compensating action fails; an approval arrived after cancellation; and a stale worker writes after lease takeover. A code rollout also leaves older workflows stuck on replay.

Competing explanations include acknowledgement lost after external commit, non-atomic idempotency, operation-key reuse or expiry, stale postcondition read, nested activity retry, outbox duplicate, incomplete compensation ordering, missing approval version/deadline, unfenced lease, changed command order, history/payload pressure, or queue saturation.

Collect workflow/run/version and ordered history, task/activity attempts, logical operation/intent hash, idempotency records, dispatch/receipt/postcondition, external audit state, outbox/inbox state, compensation graph/outcomes, approval identity/version/time, lease epochs/fencing decisions, replay failure, history size, queue spans, and terminal business outcome. Freeze automatic retries when effect is unknown; reconcile before action; patch the earliest boundary; replay old histories; re-run the crash matrix; remeasure duplicates, loss, stuck rate, recovery latency, and manual work.

## 08 Mastery Assessment

Design a durable agent workflow that calls models/tools, performs governed external writes, fans out work, waits days for approval, compensates partial success, and survives workers and deployments. Deliver history/state schemas; replay constraints; activity/effect ledger; delivery and idempotency proof; outbox/inbox path; saga graph; fencing protocol; join and HITL state machines; capacity bounds; replay corpus; crash matrix; current source trace; and a diagnosis of Incident 14.1.

## 09 Required Evidence & Rubric

- **Durability:** acknowledgement and recoverable history/checkpoint boundaries are explicit and crash-tested.
- **Replay:** nondeterministic inputs and command-affecting code changes are controlled and replay-tested.
- **Effects:** logical operation, attempts, unknown commit, receipts, postconditions, and reconciliation are distinct.
- **Idempotency:** receiver-side atomicity, intent conflicts, scope, retention, concurrency, and downstream duplication are tested.
- **Compensation:** forward-effect semantics, irreversible cases, partial/failed states, and manual recovery remain visible.
- **Coordination:** fan-out membership, join/cancel/timeout, fencing, timers, and approval authorization survive restart.
- **Capacity/versioning:** history/payload growth and every active workflow version have operational limits and a migration route.
- **Diagnosis:** workflow, task, activity, external effect, compensation, and business outcome evidence are correlated.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Durable history and deterministic replay | 14.1 | LAB A | Incident / Mastery | Replay corpus and crash report |
| Effect safety and idempotency | 14.2 | LAB B | Incident / Mastery | Effect ledger and duplicate/loss tests |
| Compensation and fencing | 14.3 | LAB B, LAB C | Incident / Mastery | Saga outcomes and stale-writer proof |
| Fan-out and durable HITL | 14.4 | LAB C | Incident / Mastery | Join/approval state machines |
| Recovery diagnosis and upgrades | 14.5–14.6 | LAB D | Incident / Mastery | Crash coverage and compatibility matrix |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to reconstruct state from a declared durable prefix; keep nondeterminism and external I/O out of replay; refuse exactly-once claims without atomic endpoint evidence; survive duplicate/concurrent delivery; model compensation honestly; fence stale owners; persist joins/timers/approvals; bound history growth; replay-test in-flight versions; and diagnose crash windows from correlated workflow/activity/effect evidence.

**Final mental model:** a durable runtime remembers decisions and coordinates retries, but external reality has its own commit points. Correctness comes from history plus effect identity, atomic idempotency or reconciliation, fencing, explicit compensation, durable coordination, and adversarial crash testing—not from replay alone.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

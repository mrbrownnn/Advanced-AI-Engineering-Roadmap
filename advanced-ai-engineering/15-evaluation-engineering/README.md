# Module 15 — Evaluation Engineering

## 00 Why This Module Exists

An evaluation score is not evidence until its decision, population, unit, system boundary, data, grader, uncertainty, and failure policy are explicit. Evaluation engineering builds that chain and keeps it reproducible as models, prompts, tools, traffic, and judges change.

```text
decision + risk tolerance
          |
population -> sampled/versioned items -> system under test -> outcomes
          |                                  |                 |
       slices                            attempts/effects    failures too
          \__________________________________|_________________/
                                             |
                          graders -> estimates -> uncertainty
                                             |
                            gate -> shadow/canary/A-B -> decision
                                             ^
                         incidents, drift, fresh data, user outcomes
```

This module owns the evaluation program: contracts, datasets, grader validation, human and LLM judging, statistical comparison, regression gates, benchmark freshness, end-to-end outcome accounting, and offline-to-online validity. Module 00 owns general evidence foundations; Module 07 owns behavioral calibration; Module 12 evaluates agent-loop behavior but owns loop mechanics; Module 16 deepens falsification and adversarial test design; Module 23 owns production observability and SLO operations.

**Research cutoff:** 2026-09-26.

- **O — source observation:** what a cited paper, specification, or pinned implementation actually reports.
- **D — derivation:** what follows under stated assumptions.
- **H — engineering hypothesis:** a causal or operational claim that must survive workload telemetry.

## 01 Baseline Assumptions

- Module 00: construct validity, experimental units, sampling, uncertainty, and falsification.
- Modules 04 and 07: latency/goodput and behavioral uncertainty/calibration.
- Modules 08–11: lineage, retrieval evidence, context, and memory state.
- Modules 12–14: trajectories, attempt identity, external effects, and durable execution.

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

The learner must be able to define an evaluation estimand and population; build versioned datasets and slices; select and validate graders; run paired, dependence-aware comparisons; define release gates before seeing results; include failed and unfinished outcomes; connect offline evidence to online outcomes; trace a pinned harness; and diagnose a disagreement among benchmark, judge, system, and user metrics.

## 03 Knowledge Map

Keep four things separate:

1. **Construct:** the property the decision cares about—task success, safety, user utility, cost, or latency.
2. **Observation:** raw output, tool effect, human label, executable test, or production event.
3. **Estimator:** aggregation, weighting, uncertainty, and slice procedure.
4. **Decision rule:** improvement/noninferiority thresholds, hard constraints, escalation, and rollout policy.

A grader observes a proxy. A benchmark samples a population. A confidence interval describes a sampling procedure. None is the product decision by itself.

## 04 Lessons

### Lesson 15.1 — Evaluation Contract and Experimental Unit

Start with the decision: what change could the result authorize, block, or investigate? Pin the target population and sampling frame, independent unit, system boundary, policy and model versions, outcomes, thresholds, uncertainty procedure, missingness, and exclusions. Request, turn, conversation, trajectory, user, and session are not interchangeable units.

When A and B run on the same independent blocks, compare paired differences:

$$d_i=m_B(i)-m_A(i),\qquad \bar d=\frac{1}{n}\sum_i d_i.$$

Resample or model the independent block. Ten generations from one prompt increase information about conditional variability, not the number of independent prompts. A row bootstrap over correlated turns or user sessions is overconfident.

HELM is a reference for scenario and metric coverage, not a universal production suite. Its durable lesson is to expose which scenarios and desiderata are measured and which remain absent.

**Outcome:** state exactly what quantity a result estimates and which decision it can support.

### Lesson 15.2 — Dataset Lineage, Slices, Freshness, and Contamination

Every item needs stable identity, origin/license, capture time, sampling probability or intended weight, split, version, deduplication lineage, slice metadata, expected answer or rubric provenance, and exclusion history. Preserve a protected final holdout; routine regression sets become development data once teams inspect and optimize against them.

Choose slices from risks and workload structure before results: task, language, locale, prompt/output length, user cohort, retrieval/tool path, policy class, traffic source, difficulty, and failure mode. Report both prevalence-weighted impact and critical-slice constraints. Tiny slices need uncertainty, not confident rankings.

LiveBench is a current design example: recent sources, frequent refresh, objective grading, and diverse tasks limit contamination. “Contamination-limited” is the defensible claim. Fresh data does not prove absence of private leakage, tuning feedback, benchmark-specific adaptation, or production relevance.

**Outcome:** reproduce dataset membership and reason about leakage, representativeness, and hidden regressions.

### Lesson 15.3 — Graders Are Measurement Instruments

Use the strongest valid oracle for each construct:

| Grader | Strong use | Failure to test |
|---|---|---|
| Exact/normalized match | Canonical closed answers | Valid equivalents; normalization bugs |
| Executable test | Observable program/tool invariant | Incomplete tests; sandbox nondeterminism |
| Semantic/rule checker | Explicit domain invariants | Coverage and implementation errors |
| LLM judge | Open-ended rubric at scale | Position, verbosity, self/preference, prompt and model drift |
| Human/domain expert | Normative, ambiguous, high-risk judgment | Rater interpretation, fatigue, identity cues, disagreement |

The MT-Bench/Chatbot Arena paper reports useful judge–human agreement in its settings and documents position, verbosity, self-enhancement, and reasoning limitations. A later systematic study shows position behavior varies by judge and task. Therefore validate a judge against blinded, independently adjudicated labels; fix judge model, prompt, decoding, parser, and rubric; randomize and swap answer order; allow ties/abstention; and report confusion, consistency, disagreement, and slice behavior.

For $n$ pairwise judgments, the descriptive tie-adjusted preference rate is

$$\hat p=\frac{W_B+0.5T}{n}.$$

It is conditional on sampled items, presentation, judge/rater population, and protocol—not context-free model quality. Swapping order detects sensitivity but does not manufacture ground truth.

Human protocols require rubric examples and counterexamples, qualification, randomization/blinding where feasible, repeated labels, tie/abstain, disagreement analysis, adjudication, privacy, and worker well-being. Do not erase meaningful plural judgments by forcing consensus.

**Outcome:** quantify what every grader gets wrong before trusting it in a gate.

### Lesson 15.4 — Uncertainty, Multiple Comparisons, and Release Gates

Report paired effect sizes and intervals, not only independent score bars. Cluster or block by the actual sampling unit; repeat stochastic generations when conditional variability matters; retain seeds and attempt identity where supported. Bootstrap intervals are approximations, not magic: few clusters, distribution shift, adaptive sampling, or nonregular statistics can break them.

Predeclare metric direction, practical effect/tolerance, confidence procedure, critical slices, and multiplicity policy. For a harm metric with $\Delta=m_B-m_A$, a noninferiority gate can require

$$UCB(\Delta)\le \tau.$$

For a benefit metric, an improvement rule can require $LCB(\Delta)\ge\epsilon$. The interval procedure determines coverage; $\tau$ and $\epsilon$ are product/risk judgments. Repeatedly searching metrics, slices, prompts, and seeds until one passes invalidates nominal error rates.

A scalar $S=\sum_k w_k z_k$ is meaningful only with fixed directions, scales, transforms, and weights. Preserve the metric vector, Pareto frontier, and hard safety/reliability constraints so a gain cannot compensate for an unacceptable harm.

**Outcome:** make release decisions auditable under practical and statistical uncertainty.

### Lesson 15.5 — End-to-End and Offline-to-Online Validity

Score offered work, not only completed successes. Keep timeouts, refusals, invalid output, retrieval/tool failure, retries, abandonment, and unfinished trajectories in explicit terminal classes. Conditioning quality on completion permits a weak system to improve by dropping hard cases.

Join task quality with safety, latency, cost, and completion. A thresholded offered-request goodput is

$$G=\frac{1}{T}\sum_i \mathbf{1}[q_i\ge q^*,\ l_i\le l^*,\ c_i\le c^*,\ terminal_i=accepted],$$

with units of useful accepted requests per time. Thresholds are workload decisions; show sensitivity and the quality–latency–cost frontier.

For agents, preserve turn and trajectory success, attempts, tool effects, recovery, intervention, irreversible harm, and final state. A correct final answer reached after unauthorized or duplicate effects is not a successful trajectory.

Offline scores are proxies. Shadowing, canaries, randomized A/B tests, or other causally credible designs must test whether they predict user and business outcomes under real traffic, latency, interaction, and feedback. Use guardrails and account for interference, novelty, selection, and delayed effects.

**Outcome:** prevent a benchmark improvement from silently reducing production utility.

### Lesson 15.6 — Evaluation Operations and Harness Trace

Use a staged funnel: deterministic unit/contract checks, sampled regression suites, adversarial and slice suites, grader/human audits, shadow/canary, and controlled online evidence. Measure each stage's cost, latency, rerun variance, unique defects, overlap, false blocks, and escaped incidents. The claim that staging lowers cost and escapes is an **H**, not a guarantee.

An immutable run manifest includes dataset/slice hashes, item/root request IDs, model/system/policy/harness/grader revisions, prompts and generation settings, all attempts, raw and normalized outputs, grades, timing, token/cost data, exclusions, terminal states, and environment.

**Production source trace:** at EleutherAI `lm-evaluation-harness` revision `d6de81643928d653435c431bae19945d41d32520`, `simple_evaluate` loads tasks and delegates to `evaluate`; the flow builds requests, dispatches request types, calls `Task.process_results`, and aggregates configured metrics. Relevant task APIs include `Task.build_all_requests`, `construct_requests`, `process_results`, `aggregation`, and `higher_is_better`. This is static inspection of one pinned implementation, not the definition of evaluation engineering.

JudgeArena (2026) is frontier evidence for making judge, benchmark, prompt/protocol, inference backend, and metadata swappable and reproducible. It does not remove the need to validate the judge or reproduce performance claims.

**Outcome:** operate evaluation as versioned production infrastructure with known cost and failure coverage.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [Holistic Evaluation of Language Models](https://arxiv.org/abs/2211.09110) — Liang et al., 2022; scenarios, metric coverage, and transparent artifacts.
- [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena](https://arxiv.org/abs/2306.05685) — Zheng et al., 2023; judge utility and documented biases.
- [Judging the Judges](https://arxiv.org/abs/2406.07791) — Shi et al., 2024; position-bias measurement.

**CURRENT DEFAULT:** explicit evaluation contract; immutable dataset and run lineage; paired comparisons; dependence-aware uncertainty; validated grader portfolio; success/failure denominators; slice constraints; cost/latency/quality reporting; staged regression and online validation.

**WORKLOAD-DEPENDENT:** grader mix, rubric, sample/weighting, repeats, cluster unit, thresholds, risk slices, judge model, refresh cadence, online design, and acceptable evaluation cost.

**FRONTIER:** [LiveBench](https://proceedings.iclr.cc/paper_files/paper/2025/hash/e4a46394ba5378b3f9a186a5b4c650d1-Abstract-Conference.html) for continuously refreshed contamination-limiting evaluation; [JudgeArena](https://arxiv.org/abs/2608.02620) for reproducible judge configuration. Neither proves zero contamination or universally valid automated judging.

**LEGACY / INSUFFICIENT:** one static benchmark as product quality; one mean without uncertainty; success-only scoring; independent tests on paired items; rows-as-independent for repeated turns; judge self-scoring without calibration; forced-choice without ties; changing prompts/graders invisibly; tuning repeatedly on the test set; one weighted score that offsets critical harm.

**PRODUCTION SOURCE TRACE**

- Repository: `EleutherAI/lm-evaluation-harness`
- Revision: `d6de81643928d653435c431bae19945d41d32520`
- Verified: 2026-09-26; static inspection only.
- Files/symbols: `lm_eval/evaluator.py::{simple_evaluate,evaluate}` and `lm_eval/api/task.py::Task.{build_all_requests,construct_requests,process_results,aggregation,higher_is_better}`.
- Path: task loading/configuration → instance construction → request dispatch → per-document results → aggregation and run metadata.

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Evaluation Contract and Dataset

- Define one production decision, population, independent unit, system boundary, estimands, thresholds, and missingness policy.
- Build an immutable dataset manifest with provenance, sampling/weights, deduplication, protected split, freshness, and risk slices.
- Break it with duplicates, near-duplicates, stale questions, slice imbalance, label leakage, repeated user/session rows, and post-result exclusions.
- Artifact: coverage map, leakage audit, weighted/unweighted estimates, and limitations register.

### LAB B — Grader Meta-Evaluation

- Implement exact/executable, semantic, two LLM-judge, and blinded human/adjudication paths on the same stratified sample.
- Randomize and swap pairwise order; vary answer length, identity cues, rubric, judge prompt/model, and ambiguous cases.
- Measure confusion/agreement, ties/abstention, repeat stability, order consistency, slice errors, latency, and full attempt cost.
- Artifact: grader card and a justified policy for automatic grade, dual grade, abstain, or expert escalation.

### LAB C — Paired Regression Gate

- Compare two system revisions on shared independent blocks with paired effects and block bootstrap intervals.
- Add stochastic repeats without pretending they are new items; exercise noninferiority, improvement, hard slice constraints, and multiplicity control.
- Break the gate with unpaired analysis, tiny slices, metric shopping, rerun-until-pass, aggregate compensation, and success-only deletion.
- Artifact: preregistered release rule, sensitivity analysis, and decision record.

### LAB D — End-to-End Evaluation Pipeline

- Build a staged suite and persist a full manifest through the pinned harness path.
- Evaluate request, turn, and trajectory outcomes; inject timeout, refusal, invalid output, tool/retrieval failure, retry, duplicate effect, and abandoned session.
- Shadow or simulate an online validation and compare offline deltas with completion, latency, cost, safety, and user outcome.
- Measure stage cost, unique defect yield, false blocks, rerun instability, escaped regressions, and detection time.
- Artifact: evaluation DAG, source trace, offline-online validity report, rollout/rollback gate, and TODO_VERIFY list.

## 07 Break / Incident Scenarios

### Incident 15.1 — The Judge Says Better; Users Say Worse

A new release wins the offline pairwise suite and passes an aggregate gate. After rollout, abandonment and support escalations rise while dashboards still show higher “quality.”

Competing explanations include position/verbosity bias, changed judge prompt/model, broken order randomization, evaluation traffic mismatch, stale or contaminated items, unreported timeouts/refusals, retries counted only on final success, latency regression, retrieval/tool failures, harmful effects hidden by final-answer grading, multiple-comparison selection, or a genuinely invalid online metric.

Recover immutable run manifests, item/sample weights, system and grader versions, paired raw outputs, order assignment, human calibration labels, all attempts and terminal states, latency/cost, tool effects, traffic slices, assignment logs, and predeclared gates. Re-grade a blinded stratified sample with executable/domain oracles and humans; swap order; include all offered requests; reproduce old/new systems on identical items; decompose offline and online deltas by slice; and verify experiment integrity. Rank explanations, intervene at the earliest falsified boundary, and remeasure offline effect, judge error, completion, latency, cost, safety, and user outcome.

## 08 Mastery Assessment

Design and operate a release evaluation for an LLM system change. Deliver the decision contract; population/unit/boundary; immutable dataset and slices; leakage/freshness controls; executable, judge, and human grader cards; paired dependence-aware statistics; multiplicity-aware release gates; offered-request quality/latency/cost accounting; trajectory/effect evaluation; pinned harness trace; staged cost/coverage telemetry; online-validation plan; and a diagnosis of Incident 15.1.

## 09 Required Evidence & Rubric

- **Validity:** construct, observation, estimator, and decision rule remain distinct.
- **Population:** sampling frame, unit, weights, slices, exclusions, and shift are explicit.
- **Lineage:** data, model, prompt, policy, harness, grader, and environment versions are reproducible.
- **Graders:** each is calibrated against an independent reference and tested for bias, drift, ties, and disagreement.
- **Statistics:** comparison is paired where possible; dependence and multiplicity match the design; practical effect accompanies significance.
- **Accounting:** failures, retries, unfinished work, latency, cost, and effects remain in denominators.
- **Transfer:** offline metrics are validated against guarded online outcomes, not asserted equivalent.
- **Operations:** suite cost, flakiness, freshness, unique detection, false blocks, and escape rate are measured.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Evaluation contract and population | 15.1–15.2 | LAB A | Incident / Mastery | Contract, manifest, coverage |
| Grader and human protocol validity | 15.3 | LAB B | Incident / Mastery | Grader card and calibration |
| Statistical comparison and gates | 15.4 | LAB C | Mastery | Paired effects, intervals, gate |
| End-to-end and online validity | 15.5 | LAB D | Incident / Mastery | Offered-work frontier and online evidence |
| Evaluation operations and traceability | 15.6 | LAB D | Mastery | Run manifest, source trace, funnel metrics |

## 11 Exit Criteria & Module Wrap-Up

Pass requires the learner to name the decision and independent unit, reproduce dataset/system/grader versions, expose missingness and failure denominators, validate an LLM judge rather than trust it, quantify paired effects with appropriate uncertainty, prevent aggregate compensation of critical regressions, connect offline evidence to online outcomes, and trace a pinned evaluation harness without generalizing it.

**Final mental model:** evaluation is a versioned measurement-and-decision system. Its credibility comes from population fit, valid observations, dependence-aware inference, explicit value judgments, end-to-end denominators, and falsifiable links to production outcomes—not from benchmark familiarity or a precise-looking score.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

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

**Module Orientation**
- **Engineering Problem**: Produce decision-relevant evidence whose population, measurement instrument, uncertainty, and release rule remain reproducible as the system changes.
- **What You Will Do**: Specify an evaluation contract, build a versioned sliced dataset, calibrate graders, run paired dependence-aware comparisons, account for all terminal outcomes, trace the evaluation harness, and defend an offline-to-online release gate.
- **Environment**: Python 3.10+, immutable dataset/run manifests, deterministic and model-based graders, optional blinded human labels, and a statistical notebook or test harness.
- **Evidence Rule**: **O** is a source observation, **D** is a derivation under stated assumptions, and **H** is an engineering hypothesis requiring telemetry.

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
  experimental: REQUIRED
  statistical: REQUIRED
  production_reasoning: REQUIRED
  failure_analysis: REQUIRED
  falsification: REQUIRED
  security: SELECTIVE
  economics: REQUIRED
  architecture_tradeoff: REQUIRED
  research_connection: REQUIRED

estimated_effort:
  instruction: 4h
  guided_practice: 3h
  labs: 12h
  assessment: 3h
  source_trace: 2h
  total: 24h
```

The learner must be able to define an evaluation estimand and population; build versioned datasets and slices; select and validate graders; run paired, dependence-aware comparisons; define release gates before seeing results; include failed and unfinished outcomes; connect offline evidence to online outcomes; trace a pinned harness; and diagnose a disagreement among benchmark, judge, system, and user metrics.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

Keep four things separate:

1. **Construct:** the property the decision cares about—task success, safety, user utility, cost, or latency.
2. **Observation:** raw output, tool effect, human label, executable test, or production event.
3. **Estimator:** aggregation, weighting, uncertainty, and slice procedure.
4. **Decision rule:** improvement/noninferiority thresholds, hard constraints, escalation, and rollout policy.

A grader observes a proxy. A benchmark samples a population. A confidence interval describes a sampling procedure. None is the product decision by itself.

## 04 Lessons

### Lesson 15.1 — Evaluation Contract and Experimental Unit

**Engineering Question:**
What population quantity is being estimated, at which independent unit, and for which decision?

**Concepts & Definitions:**
- **Construct**: property the decision cares about.
- **Estimand**: precisely defined population quantity.
- **Independent unit**: block at which sampling variability is modeled.

**Quantitative Model / Derivation:**

Start with the decision: what change could the result authorize, block, or investigate? Pin the target population and sampling frame, independent unit, system boundary, policy and model versions, outcomes, thresholds, uncertainty procedure, missingness, and exclusions. Request, turn, conversation, trajectory, user, and session are not interchangeable units.

When A and B run on the same independent blocks, compare paired differences:

$$d_i=m_B(i)-m_A(i),\qquad \bar d=\frac{1}{n}\sum_i d_i.$$

Resample or model the independent block. Ten generations from one prompt increase information about conditional variability, not the number of independent prompts. A row bootstrap over correlated turns or user sessions is overconfident.

HELM is a reference for scenario and metric coverage, not a universal production suite. Its durable lesson is to expose which scenarios and desiderata are measured and which remain absent.

**Mechanism Explanation:**
Bind the decision, target population, sampling frame, unit, system boundary, versions, outcomes, thresholds, uncertainty, missingness, and exclusions before observing results.

**Worked Example:**
Two systems run on 100 shared prompts with five generations each. Pair by prompt and resample prompts, not 500 rows; repeats estimate conditional generation variability rather than 500 independent tasks.

**Knowledge Check:**
1. Why are turns from one conversation not automatically independent?
2. Which decision changes if the system boundary includes retries and retrieval?

**Guided Practice:**
Write an evaluation contract for one release decision and identify every unit that could be confused with the independent block.

**Feedback Contract:**
- *Expected Evidence*: Population, frame, independent unit, boundary, estimand, versions, thresholds, and missingness are explicit.
- *Common Failure*: Treating repeated samples as new independent prompts.
- *Diagnostic Hint*: What process generated the independent blocks?
- *Concept to Revisit*: Evaluation Estimand.

**Learning Outcome:**
State exactly what quantity a result estimates and which decision it can support.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 15.2 — Dataset Lineage, Slices, Freshness, and Contamination

**Engineering Question:**
How can a dataset support reproducible estimates without becoming an invisible development target or hiding critical slices?

**Concepts & Definitions:**
- **Lineage**: identity, origin, transformations, labels/rubrics, and exclusions.
- **Slice**: preregistered subgroup tied to workload structure or risk.
- **Protected holdout**: data withheld from routine inspection and optimization.

**Quantitative Model / Derivation:**

Every item needs stable identity, origin/license, capture time, sampling probability or intended weight, split, version, deduplication lineage, slice metadata, expected answer or rubric provenance, and exclusion history. Preserve a protected final holdout; routine regression sets become development data once teams inspect and optimize against them.

Choose slices from risks and workload structure before results: task, language, locale, prompt/output length, user cohort, retrieval/tool path, policy class, traffic source, difficulty, and failure mode. Report both prevalence-weighted impact and critical-slice constraints. Tiny slices need uncertainty, not confident rankings.

LiveBench is a current design example: recent sources, frequent refresh, objective grading, and diverse tasks limit contamination. “Contamination-limited” is the defensible claim. Fresh data does not prove absence of private leakage, tuning feedback, benchmark-specific adaptation, or production relevance.

**Mechanism Explanation:**
Preserve stable item identity, sampling/weighting, split, deduplication, provenance, and exclusion history. Report prevalence-weighted impact beside critical-slice constraints.

**Quantitative Model / Derivation:**
For declared slice weights $w_s$ summing to one, a prevalence-weighted estimate is $\hat m=\sum_s w_s\hat m_s$. It does not authorize a severe critical-slice regression to be averaged away.

**Worked Example:**
Assume slice weights 0.9 and 0.1 with scores 0.90 and 0.40. The weighted score is 0.85, but a predeclared critical-slice floor of 0.70 still fails.

**Knowledge Check:**
1. Why does frequent refresh limit rather than eliminate contamination?
2. When does a regression set become development data?

**Guided Practice:**
Build a manifest, deduplication lineage, freshness policy, weighted estimate, and critical-slice gate for a stratified sample.

**Feedback Contract:**
- *Expected Evidence*: Reproducible membership, weights, leakage audit, uncertainty by slice, and protected split policy.
- *Common Failure*: Post-result exclusions or an aggregate that compensates critical harm.
- *Diagnostic Hint*: Which items were inspected during development?
- *Concept to Revisit*: Dataset Lineage and Slice Constraints.

**Learning Outcome:**
Reproduce dataset membership and reason about leakage, representativeness, and hidden regressions.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 15.3 — Graders Are Measurement Instruments

**Engineering Question:**
How should deterministic, model-based, and human graders be calibrated before they influence a release gate?

**Concepts & Definitions:**
- **Oracle**: observation procedure valid for a declared construct.
- **Meta-evaluation**: measuring grader error against independent adjudicated evidence.
- **Order sensitivity**: judgment changes caused by presentation order rather than answer quality.

**Quantitative Model / Derivation:**

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

**Mechanism Explanation:**
Choose the strongest valid oracle, blind/randomize where feasible, permit ties/abstention, and report confusion, disagreement, consistency, slice error, drift, and full cost.

**Worked Example:**
Across 20 order-balanced pairs, suppose B wins 9, A wins 7, and 4 tie. The descriptive tie-adjusted preference for B is $(9+0.5\times4)/20=0.55$; it is not context-free model quality.

**Knowledge Check:**
1. Why does swapping order detect sensitivity but not create ground truth?
2. What independent evidence is needed before using a judge in a gate?

**Guided Practice:**
Calibrate exact/executable, semantic, two judge, and blinded human paths on one stratified sample; report disagreement rather than forcing consensus.

**Feedback Contract:**
- *Expected Evidence*: Fixed grader versions/protocol, order assignment, ties, reference labels, slice error, uncertainty, latency, and cost.
- *Common Failure*: Self-judging or forced choice without calibration.
- *Diagnostic Hint*: Does disagreement track answer position, length, or slice?
- *Concept to Revisit*: Grader Validity.

**Learning Outcome:**
Quantify what every grader gets wrong before trusting it in a gate.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 15.4 — Uncertainty, Multiple Comparisons, and Release Gates

**Engineering Question:**
How can paired evidence, dependence, practical thresholds, and repeated looks be combined into an auditable release decision?

**Concepts & Definitions:**
- **Paired effect**: within-block difference between systems.
- **Practical threshold**: minimum benefit or maximum tolerated harm.
- **Multiplicity policy**: control for selecting among many metrics, slices, or repeated analyses.

**Quantitative Model / Derivation:**

Report paired effect sizes and intervals, not only independent score bars. Cluster or block by the actual sampling unit; repeat stochastic generations when conditional variability matters; retain seeds and attempt identity where supported. Bootstrap intervals are approximations, not magic: few clusters, distribution shift, adaptive sampling, or nonregular statistics can break them.

Predeclare metric direction, practical effect/tolerance, confidence procedure, critical slices, and multiplicity policy. For a harm metric with $\Delta=m_B-m_A$, a noninferiority gate can require

$$UCB(\Delta)\le \tau.$$

For a benefit metric, an improvement rule can require $LCB(\Delta)\ge\epsilon$. The interval procedure determines coverage; $\tau$ and $\epsilon$ are product/risk judgments. Repeatedly searching metrics, slices, prompts, and seeds until one passes invalidates nominal error rates.

A scalar $S=\sum_k w_k z_k$ is meaningful only with fixed directions, scales, transforms, and weights. Preserve the metric vector, Pareto frontier, and hard safety/reliability constraints so a gain cannot compensate for an unacceptable harm.

**Mechanism Explanation:**
Predeclare metric direction, blocks/clusters, interval procedure, thresholds, critical slices, and repeated-look policy. Plan sample size through detectable-effect or interval-width sensitivity using pilot variance, then re-evaluate assumptions.

**Worked Example:**
For paired differences $[0.02,0.01,-0.01,0.04]$, the mean is 0.015. A release still depends on the declared block-aware interval and practical threshold; the positive mean alone cannot pass the gate.

**Knowledge Check:**
1. Why can row bootstrap understate uncertainty for user-clustered data?
2. What happens to nominal error rates under rerun-until-pass behavior?

**Guided Practice:**
Run paired block bootstrap and sensitivity planning, then compare improvement, noninferiority, hard-slice, and multiplicity-aware gates.

**Feedback Contract:**
- *Expected Evidence*: Paired effects, independent blocks, uncertainty procedure, effect/tolerance, sample-planning assumptions, and look policy.
- *Common Failure*: Independent score bars on paired items or significance without practical effect.
- *Diagnostic Hint*: Which choices were made after seeing results?
- *Concept to Revisit*: Dependence-Aware Release Gate.

**Learning Outcome:**
Make release decisions auditable under practical and statistical uncertainty.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 15.5 — End-to-End and Offline-to-Online Validity

**Engineering Question:**
When does an offline quality gain fail to improve offered-user utility under real latency, failures, effects, and traffic?

**Concepts & Definitions:**
- **Offered denominator**: every eligible request/session, including failure and unfinished states.
- **Goodput**: useful accepted outcomes meeting declared constraints per time.
- **Online validity**: causal evidence that proxy changes transfer to user/business outcomes.

**Quantitative Model / Derivation:**

Score offered work, not only completed successes. Keep timeouts, refusals, invalid output, retrieval/tool failure, retries, abandonment, and unfinished trajectories in explicit terminal classes. Conditioning quality on completion permits a weak system to improve by dropping hard cases.

Join task quality with safety, latency, cost, and completion. A thresholded offered-request goodput is

$$G=\frac{1}{T}\sum_i \mathbf{1}[q_i\ge q^*,\ l_i\le l^*,\ c_i\le c^*,\ terminal_i=accepted],$$

with units of useful accepted requests per time. Thresholds are workload decisions; show sensitivity and the quality–latency–cost frontier.

For agents, preserve turn and trajectory success, attempts, tool effects, recovery, intervention, irreversible harm, and final state. A correct final answer reached after unauthorized or duplicate effects is not a successful trajectory.

Offline scores are proxies. Shadowing, canaries, randomized A/B tests, or other causally credible designs must test whether they predict user and business outcomes under real traffic, latency, interaction, and feedback. Use guardrails and account for interference, novelty, selection, and delayed effects.

**Mechanism Explanation:**
Join quality, safety, completion, latency, cost, and external effects; validate offline deltas using guarded shadow/canary/randomized designs with explicit assignment and exposure units. Check sample-ratio mismatch, interference, novelty, selection, and delayed outcomes.

**Worked Example:**
In 100 s, assume 50 offered requests: 30 accepted outcomes satisfy quality/latency/cost thresholds, 8 time out, 4 refuse, 3 fail tools, and 5 remain unfinished. Goodput is $30/100=0.3$ useful requests/s; success-only quality excludes evidence needed for the product decision.

**Knowledge Check:**
1. How can completion-conditioned quality improve while user utility falls?
2. Why can user-level assignment be required for an interactive product?

**Guided Practice:**
Build an offered-outcome ledger and an online validation plan specifying assignment, exposure, guardrails, sample-ratio checks, interference, and delayed effects.

**Feedback Contract:**
- *Expected Evidence*: All terminal classes, trajectory effects, thresholds, sensitivity, assignment integrity, and offline/online deltas by slice.
- *Common Failure*: Counting only final successes or treating an offline proxy as causal proof.
- *Diagnostic Hint*: Which offered users disappeared from the denominator?
- *Concept to Revisit*: Offline-to-Online Validity.

**Learning Outcome:**
Prevent a benchmark improvement from silently reducing production utility.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 15.6 — Evaluation Operations and Harness Trace

**Engineering Question:**
How should evaluation run as versioned infrastructure whose cost, coverage, drift, and escaped defects are observable?

**Concepts & Definitions:**
- **Evaluation funnel**: staged checks from deterministic contracts through controlled online evidence.
- **Run manifest**: immutable identity and lineage for data, system, grader, attempts, outcomes, and environment.
- **Unique defect yield**: adjudicated failures found by a stage beyond overlap with other stages.

**Quantitative Model / Derivation:**

Use a staged funnel: deterministic unit/contract checks, sampled regression suites, adversarial and slice suites, grader/human audits, shadow/canary, and controlled online evidence. Measure each stage's cost, latency, rerun variance, unique defects, overlap, false blocks, and escaped incidents. The claim that staging lowers cost and escapes is an **H**, not a guarantee.

An immutable run manifest includes dataset/slice hashes, item/root request IDs, model/system/policy/harness/grader revisions, prompts and generation settings, all attempts, raw and normalized outputs, grades, timing, token/cost data, exclusions, terminal states, and environment.

**Production source trace:** at EleutherAI `lm-evaluation-harness` revision `d6de81643928d653435c431bae19945d41d32520`, `simple_evaluate` loads tasks and delegates to `evaluate`; the flow builds requests, dispatches request types, calls `Task.process_results`, and aggregates configured metrics. Relevant task APIs include `Task.build_all_requests`, `construct_requests`, `process_results`, `aggregation`, and `higher_is_better`. This is static inspection of one pinned implementation, not the definition of evaluation engineering.

JudgeArena (2026) is frontier evidence for making judge, benchmark, prompt/protocol, inference backend, and metadata swappable and reproducible. It does not remove the need to validate the judge or reproduce performance claims.

**Mechanism Explanation:**
Version every dependency, preserve all attempts and exclusions, measure stage cost/latency/flakiness/unique defects/false blocks/escapes, and trace the pinned harness without universalizing it.

**Quantitative Model / Trade-off Comparison:**
A stage is not justified by case count alone. Compare unique consequential defect yield and escaped incidents against compute, judge, human, latency, and triage cost under a predeclared decision rule.

**Worked Example:**
A cheap contract stage catches failures also found downstream, while one slice suite finds two unique severe regressions. Report overlap and severity rather than claiming the largest test count is best.

**Knowledge Check:**
1. Which manifest fields localize a grader change from a model change?
2. Why is a pinned source path not the definition of evaluation engineering?

**Independent Practice:**
Run one immutable manifest through task loading, request construction, dispatch, per-document processing, aggregation, and metadata capture; inject one change at a time.

**Feedback Contract:**
- *Expected Evidence*: Versioned run, complete attempts, stage costs, unique/overlap defects, false blocks, escapes, and source trace.
- *Common Failure*: Changing data or graders invisibly between runs.
- *Diagnostic Hint*: What is the earliest differing manifest field?
- *Concept to Revisit*: Evaluation Operations.

**Learning Outcome:**
Operate evaluation as versioned production infrastructure with known cost and failure coverage.

*(Effort: 35m instruction, 25m practice)*

---

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

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Evaluation Contract and Dataset

- **Objective**: Build a reproducible contract and sampled dataset for one production decision.
- **Pre-Registered Hypothesis**: Preregistered units, weights, slices, and missingness will reveal at least one ranking or uncertainty change hidden by naive row-level scoring.
- **Independent Variables**: Sampling frame, split/freshness, weighting, slice definition, and deduplication.
- **Dependent Variables**: Coverage, leakage, weighted/unweighted effects, slice uncertainty, and exclusions.

- Define one production decision, population, independent unit, system boundary, estimands, thresholds, and missingness policy.
- Build an immutable dataset manifest with provenance, sampling/weights, deduplication, protected split, freshness, and risk slices.
- Break it with duplicates, near-duplicates, stale questions, slice imbalance, label leakage, repeated user/session rows, and post-result exclusions.
- Artifact: coverage map, leakage audit, weighted/unweighted estimates, and limitations register.
- **Break & Falsify**: Inject duplicates, stale items, imbalance, leakage, clustered rows, and post-result exclusion; survival of a biased estimate falsifies the controls.
- **Alignment**: Lessons 15.1–15.2.
- **Effort Estimate**: 3h total.

### LAB B — Grader Meta-Evaluation

- **Objective**: Calibrate executable, semantic, judge, and human paths on the same stratified sample.
- **Pre-Registered Hypothesis**: Order balancing and independent labels will expose measurable judge error or sensitivity on at least one preregistered stress slice.
- **Independent Variables**: Grader/protocol, answer order/length/cue, rubric, judge model/prompt, and slice.
- **Dependent Variables**: Confusion/agreement, ties/abstention, order consistency, slice error, stability, latency, and cost.

- Implement exact/executable, semantic, two LLM-judge, and blinded human/adjudication paths on the same stratified sample.
- Randomize and swap pairwise order; vary answer length, identity cues, rubric, judge prompt/model, and ambiguous cases.
- Measure confusion/agreement, ties/abstention, repeat stability, order consistency, slice errors, latency, and full attempt cost.
- Artifact: grader card and a justified policy for automatic grade, dual grade, abstain, or expert escalation.
- **Break & Falsify**: Include ambiguous cases and deliberately invalid cues; a judge whose errors cannot be bounded must not control the gate.
- **Alignment**: Lesson 15.3.
- **Effort Estimate**: 3h total.

### LAB C — Paired Regression Gate

- **Objective**: Implement paired dependence-aware improvement/noninferiority gates with protected slice constraints.
- **Pre-Registered Hypothesis**: Block-aware paired analysis will produce different uncertainty than naive unpaired/row-level analysis on seeded clustered data.
- **Independent Variables**: System revision, block/repeat structure, gate type, threshold, multiplicity policy, and sample size.
- **Dependent Variables**: Paired effect, interval width/coverage diagnostics, decision, slice violations, and rerun stability.

- Compare two system revisions on shared independent blocks with paired effects and block bootstrap intervals.
- Add stochastic repeats without pretending they are new items; exercise noninferiority, improvement, hard slice constraints, and multiplicity control.
- Break the gate with unpaired analysis, tiny slices, metric shopping, rerun-until-pass, aggregate compensation, and success-only deletion.
- Artifact: preregistered release rule, sensitivity analysis, and decision record.
- **Break & Falsify**: Use tiny slices, metric shopping, rerun-until-pass, aggregate compensation, and success-only deletion; any undeclared selection that passes exposes an invalid gate.
- **Alignment**: Lesson 15.4.
- **Effort Estimate**: 3h total.

### LAB D — End-to-End Evaluation Pipeline

- **Objective**: Run a versioned staged evaluation from deterministic checks through simulated/shadow online validation.
- **Pre-Registered Hypothesis**: Complete offered-outcome accounting will expose at least one regression hidden by accepted-success-only scoring in the injected set.
- **Independent Variables**: Failure class, trajectory effect, traffic slice, stage, assignment/exposure design, and system version.
- **Dependent Variables**: Quality, completion, safety, latency, cost, goodput, stage yield, false blocks, escapes, and online outcome.

- Build a staged suite and persist a full manifest through the pinned harness path.
- Evaluate request, turn, and trajectory outcomes; inject timeout, refusal, invalid output, tool/retrieval failure, retry, duplicate effect, and abandoned session.
- Shadow or simulate an online validation and compare offline deltas with completion, latency, cost, safety, and user outcome.
- Measure stage cost, unique defect yield, false blocks, rerun instability, escaped regressions, and detection time.
- Artifact: evaluation DAG, source trace, offline-online validity report, rollout/rollback gate, and TODO_VERIFY list.
- **Break & Falsify**: Inject timeout/refusal/invalid output/tool failure/retry/duplicate effect/abandonment and experiment-integrity faults; any invisible offered outcome falsifies completeness.
- **Alignment**: Lessons 15.5–15.6 and Incident 15.1.
- **Effort Estimate**: 3h total.

## 07 Break / Incident Scenarios

### Incident 15.1 — The Judge Says Better; Users Say Worse

- **Incident Symptoms**: A release wins offline pairwise evaluation and an aggregate gate, yet rollout increases abandonment and support escalation while the quality dashboard remains positive.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: Judge bias/drift, broken randomization, traffic mismatch, contamination, missing terminal outcomes, success-only retries, latency/tool/retrieval regressions, hidden harmful effects, metric selection, or invalid online measurement.
  2. *Rank Initial Plausibility*: Use timing, slice, and assignment evidence without assuming offline or online metrics are ground truth.
  3. *Identify Missing Evidence*: Recover manifests, weights, versions, paired outputs, order assignment, calibration labels, all attempts/states, latency/cost/effects, traffic slices, assignment/exposure logs, and preregistered gates.
  4. *Design Discriminating Tests*: Blind re-grade with independent oracles/humans, swap order, include offered work, replay both systems on identical items, and verify assignment/sample ratios; state falsifiers.
  5. *Execute Causal Diagnosis*: Decompose offline and online deltas by slice and rank supported interacting explanations.
  6. *Prescribe Mitigation and Prevention*: Pause/roll back if guardrails require it, then repair the earliest invalid measurement or system boundary.
  7. *Remeasure*: Offline effect, judge error, completion, latency, cost, safety, user outcomes, and experiment integrity.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Evidence-Gated LLM Release

Design and operate the release evaluation for a system change serving heterogeneous interactive and automated workflows.

**Required Deliverables**:
1. Decision contract, population, independent unit, boundary, estimands, and missingness.
2. Immutable dataset/slice manifest with leakage, freshness, and weighting controls.
3. Executable, judge, and human grader cards with meta-evaluation.
4. Paired dependence-aware statistics, sensitivity planning, and multiplicity-aware gates.
5. Offered-request quality/latency/cost/effect accounting.
6. Pinned harness trace and staged cost/coverage/escape telemetry.
7. Online validation, guardrail, rollout, and rollback plan.
8. Evidence-backed diagnosis of Incident 15.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit a pinned trace from task loading and request construction through dispatch, per-document processing, aggregation, and run metadata. Separate observed implementation behavior from general evaluation requirements.

### Rubric Dimensions

- **Validity and Population**: *Insufficient* reports a score. *Competent* defines construct, population, unit, estimator, and decision. *Strong* proves lineage, weighting, dependence, slices, and shift limitations.
- **Graders and Statistics**: *Insufficient* trusts a judge or mean. *Competent* calibrates graders and reports paired uncertainty. *Strong* handles multiplicity, sensitivity planning, disagreement, and practical thresholds.
- **Accounting and Transfer**: *Insufficient* scores successes only. *Competent* retains failures/cost/latency/effects and plans guarded online validation. *Strong* verifies assignment integrity and explains offline-online divergence.
- **Operations and Diagnosis**: *Insufficient* changes fixtures invisibly. *Competent* versions the pipeline and source trace. *Strong* measures unique yield, false blocks, escapes, and discriminating remeasurement.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Evaluation contract and population | 15.1–15.2 | LAB A | Incident / Mastery | Contract, manifest, coverage |
| Grader and human protocol validity | 15.3 | LAB B | Incident / Mastery | Grader card and calibration |
| Statistical comparison and gates | 15.4 | LAB C | Mastery | Paired effects, intervals, gate |
| End-to-end and online validity | 15.5 | LAB D | Incident / Mastery | Offered-work frontier and online evidence |
| Evaluation operations and traceability | 15.6 | LAB D | Mastery | Run manifest, source trace, funnel metrics |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner successfully completing Module 15 must be able to:
1. Name the decision, population, independent unit, boundary, and estimand.
2. Reproduce dataset, system, harness, and grader versions.
3. Expose missingness, failures, retries, unfinished work, latency, cost, and effects.
4. Validate automated/human graders and quantify paired effects with appropriate uncertainty.
5. Prevent aggregate compensation of critical regressions.
6. Test rather than assume offline-to-online transfer.
7. Trace a pinned harness without generalizing it.

### Module Wrap-Up (Final Mental Model Reconstruction)

- **The Core Invariant**: Evaluation is a versioned measurement-and-decision system.
- **The Evidence Path**: decision/population → sampled observations → graders → dependence-aware estimates → explicit gate → guarded online validation.
- Credibility comes from population fit, valid observations, explicit value judgments, complete denominators, and falsifiable links to production outcomes—not from a precise-looking score.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

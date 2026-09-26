# Module 07 — Model Behavior & Uncertainty

## 00 Why This Module Exists

An LLM can be fluent and wrong, uncertain and correct, confidently wrong on one slice, or well calibrated on average while failing a subgroup. “Hallucination,” “confidence,” and “reliability” are not measurements until the engineer defines the event, evidence boundary, population, sampling unit, and consequence.

This module builds the operational loop:

$$
\text{behavior contract} \to \text{labeled events and slices}
\to \text{uncertainty signal} \to \text{calibration}
\to \text{accept/abstain/escalate} \to \text{regression monitoring}.
$$

The module covers failure taxonomy, factuality/truthfulness measurement, calibration, distribution shift, selective prediction, uncertainty signals for free-form generation, conformal factuality, and behavior regression. Test-time search and verifiers belong to Module 06; general evaluation-program design belongs to Module 15; retrieval quality to Modules 09–10; security behavior to Module 18.

**Research cutoff:** 2026-09-26.

- **Engineering problem:** determine when a model output should be trusted, checked, abstained, or escalated under a declared deployment contract.
- **What you will do:** annotate failures at claim level; construct and calibrate confidence signals; measure proper scores and risk-coverage; break uncertainty methods under shift and correlated errors; trace generation scores in current source; and diagnose a release regression hidden by aggregates.
- **Evidence rule:** preserve source observations (**O**), assumption-backed derivations (**D**), and telemetry-dependent hypotheses (**H**). A detector score is not a factuality label, and a marginal guarantee is not a per-request promise.

## 01 Baseline Assumptions

- **Module 00:** measurands, units, sampling frames, uncertainty intervals, preregistration, leakage, multiple comparisons, provenance, and falsification.
- **Module 01:** logits, softmax, sampling, autoregressive likelihood, tokenization, and decoding controls.
- **Module 06:** candidate sampling, verifier fallibility, oracle-versus-selected accuracy, and test-time cost. This module uses these mechanisms only as uncertainty inputs.
- **Mathematics:** conditional expectation, Bernoulli loss, binning, logarithms, and empirical distributions.
- **Product contract:** an answer event, cost of wrong answer, cost of abstention/escalation, evidence authority, knowledge date, and target population must be declared.

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

The learner must be able to:

1. replace “hallucination” with an auditable, task-relative failure taxonomy;
2. define confidence as a forecast of a specific event on a specific population;
3. measure calibration without relying on ECE alone;
4. distinguish likelihood, self-evaluation, sample disagreement, semantic uncertainty, and learned scores;
5. select abstention thresholds using risk-coverage and application utility;
6. state exactly what a conformal guarantee covers and which assumptions sustain it;
7. detect slice and shift regressions across model, prompt, runtime, retrieval, tool, and evaluator versions;
8. diagnose apparent behavior changes through label audit and matched reruns.

## 03 Knowledge Map

```text
task + evidence + time + output obligations
                    |
                    v
             behavior contract
                    |
          output -> events/claims -> labels + severity + slices
                    |
     +--------------+-----------------------------+
     |              |              |              |
 likelihood   self-evaluation  sample/semantic  external score
     +--------------+--------------+--------------+
                    |
             held-out calibration
                    |
       proper scores + reliability + ranking
                    |
      threshold -> answer / abstain / check / escalate
                    |
       coverage + conditional risk + utility + capacity
                    |
        versioned regression and shift monitoring
```

Never collapse the following:

- correctness, factual support, truthfulness, usefulness, instruction compliance, and safety;
- token likelihood, answer correctness probability, and confidence expressed in natural language;
- calibration, discrimination/ranking, and raw task accuracy;
- low risk at low coverage and reliability for the whole population;
- marginal conformal coverage and per-prompt correctness.

---

### LAYER 1: FOUNDATIONS AND MECHANISMS

## 04 Lessons

### Lesson 7.1 — Behavior Contracts and Failure Taxonomy

**Engineering question:** What exactly failed, relative to which evidence and obligation?

A behavior contract declares:

- task, user population, language and time window;
- allowed context, retrieval/tool outputs, authoritative sources, and knowledge cutoff;
- required answer, refusal, citation, format, and uncertainty behavior;
- unit of analysis: response, claim, citation, step, session, or user outcome;
- label rules, ambiguity/adjudication, severity, and downstream consequence.

A useful taxonomy may include unsupported fabrication, contradiction with supplied evidence, stale fact, wrong attribution/citation, reasoning error, omission, instruction violation, invalid refusal, format failure, and evaluator failure. Categories can interact and are not universal: an unsupported claim in closed-book QA differs from a contradiction in grounded summarization.

TruthfulQA isolates susceptibility to selected human misconceptions. FActScore decomposes long-form output into atomic facts and measures supported-fact precision. Neither is “the hallucination metric”: TruthfulQA is a fixed behavior slice, while FActScore depends on claim decomposition, retrieval, evidence authority, entailment, and time. Factual precision also omits required-fact recall and usefulness.

Kalai and Vempala prove a lower bound for a defined class of arbitrary facts under their calibration assumptions. The theorem is important precisely because its scope is narrow: it does not make systematic facts, grounded answers, or all production errors inevitable.

**Learning outcome:** create failure labels that identify an intervention instead of hiding distinct causes under one rate.

---

### Lesson 7.2 — Confidence, Calibration, and Proper Measurement

**Engineering question:** Probability of what, for whom, and under which sampling process?

For binary event $Y$ and forecast $Q$, population calibration targets:

$$
\mathbb{E}[Y\mid Q=q]=q.
$$

This is an estimand, not a finite-sample test result. Change correctness rules, prompts, languages, time, or subgroups and the estimand changes. Aggregate calibration can coexist with severe subgroup miscalibration.

For confidence bins $B_b$, a common estimator is:

$$
ECE=\sum_b\frac{n_b}{n}|acc(B_b)-conf(B_b)|.
$$

ECE depends on binning and finite-sample noise. Report bin definitions/counts and uncertainty. Complement it with accuracy and ranking plus proper scores such as:

$$
Brier=\frac1n\sum_i(q_i-y_i)^2,
$$

$$
NLL=-\frac1n\sum_i[y_i\log q_i+(1-y_i)\log(1-q_i)].
$$

These scores combine calibration and sharpness; they do not localize the failure cause.

Scalar temperature scaling uses:

$$
p_T(y\mid x)=softmax(z(x)/T)_y,\quad T>0.
$$

It preserves argmax but changes probability sharpness. Fit $T$ only on calibration data, freeze it before testing, and repeat after shift. One scalar cannot generally correct task-, class-, language-, or subgroup-conditional errors.

For free-form generation, declare how a scalar is constructed: selected-token log probability, length normalization, answer-option probability, prompted P(True), learned probe, verifier, or another signal. Jiang et al. show calibration problems in studied generative QA models; Kadavath et al. show promising but task-dependent P(True)/P(IK behavior. Neither licenses universal self-confidence semantics.

**Learning outcome:** turn “confidence” into a reproducible forecast and prove its meaning on held-out data.

---

### Lesson 7.3 — Selective Prediction and Abstention

**Engineering question:** Which outputs should the system deliver, and what happens to the rest?

For acceptance event $A_t$ at threshold $t$ and loss $L$:

$$
coverage(t)=P(A_t),\qquad risk(t)=\mathbb{E}[L\mid A_t].
$$

Plot risk against coverage and report prespecified operating points. A system can achieve near-zero answered risk by rejecting almost everything. Therefore record abstention/retry/escalation cost, user abandonment, queue/capacity impact, subgroup coverage, and risk among rejected cases when labels are obtainable.

Calibration and ranking answer different questions. A monotone score may rank risk well but be numerically miscalibrated; a globally calibrated score can rank poorly inside a narrow operating range. Thresholds selected on the test set leak outcomes. Thresholds selected once can fail after model, prompt, traffic, or knowledge shift.

Possible actions are not only “answer” or “refuse”: retrieve evidence, invoke a deterministic tool, sample/reason again, ask a clarifying question, return a bounded uncertainty set, or escalate. Each action needs its own cost and failure contract.

**Learning outcome:** defend an operating point by coverage, conditional risk, utility, subgroup impact, and downstream capacity.

---

### Lesson 7.4 — Uncertainty for Free-Form Generation

**Engineering question:** How can we estimate uncertainty when many strings express the same answer?

Useful signal families include:

- token/sequence likelihood from an accessible model;
- prompted or trained self-evaluation such as P(True);
- disagreement across stochastic samples;
- semantic clustering and entropy over meanings;
- hidden-state probes or independent learned detectors;
- retrieval/evidence support and entailment scores.

SelfCheckGPT uses black-box sample inconsistency as a hallucination signal. Semantic entropy groups generations by meaning before computing uncertainty. These are valuable mechanisms, not oracles. A model can repeat one correlated falsehood with low disagreement. Surface variation can be factual paraphrase. Semantic clustering can merge contradiction or split equivalents. Sampling adds cost and changes with temperature.

Evaluation must use an independent correctness label—not the same detector being assessed—and report discrimination, calibration, risk-coverage, cost, and performance by failure type. Test counterexamples: stable misconceptions, ambiguous questions, multiple correct answers, paraphrases, entity aliases, stale facts, and adversarially persuasive false statements.

**Learning outcome:** choose the granularity and signal that match the event, then actively search for consistency illusions.

---

### Lesson 7.5 — Conformal Guarantees Without Guarantee Inflation

**Engineering question:** What does a finite-sample coverage statement actually promise?

Conformal methods calibrate a nonconformity score or output-set rule on exchangeable examples. Conformal factuality connects correctness to entailment sets and backs off toward less-specific output to meet a target marginal guarantee under its assumptions.

Audit every guarantee:

- prediction object and event covered;
- marginal versus conditional/per-group statement;
- calibration sample and exchangeability assumption;
- score and entailment/evidence semantics;
- finite-sample procedure and tie/randomization details;
- what happens after prompt, traffic, temporal, model, or evaluator shift;
- coverage cost: less-specific output, larger sets, or more abstention.

A nominal marginal guarantee does not imply every prompt is correct, all subgroups meet the rate, the answer is complete/useful, or a changed pipeline remains covered. Monitor realized coverage with confidence intervals, but do not “repair” a violation by repeatedly tuning on the test stream without accounting for adaptivity.

**Learning outcome:** implement conformal control while preserving the exact theorem-to-system contract.

---

### Lesson 7.6 — Behavior Regression and Shift Diagnosis

**Engineering question:** Did the model change, or did the population, evidence, evaluator, or pipeline change?

A regression suite combines:

- stable anchor items for paired release comparison;
- time-sensitive items with explicit as-of dates;
- deployment-language, domain, and subgroup slices;
- adversarial and known-severity failures;
- sampled production cases with provenance and privacy controls;
- repeated samples for stochastic behavior;
- calibration, proper score, and risk-coverage outputs—not accuracy alone.

Version model weights, system/developer/user prompts, decoding, runtime, retrieval corpus/index, tools, evaluator, label policy, and sampling frame. When a slice moves, consider model behavior, prompt interaction, runtime/tokenizer change, retrieval/tool failure, judge drift, label error, contamination, and traffic shift.

Use paired deltas where the same items are valid across releases; use independent-population methods for live samples. Predeclare practically important margins and high-severity zero/near-zero tolerance rules. Exploratory slices found after seeing results require held-out replication.

**Learning outcome:** diagnose behavior changes causally enough to select rollback, recalibration, routing, retrieval, prompt, or model interventions.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [On Calibration of Modern Neural Networks](https://arxiv.org/abs/1706.04599) — Guo et al. (2017): calibration metrics and temperature scaling.
- [Can You Trust Your Model's Uncertainty?](https://arxiv.org/abs/1906.02530) — Ovadia et al. (2019): uncertainty under dataset shift; original domain is image classification.
- [Selective Classification for Deep Neural Networks](https://arxiv.org/abs/1705.08500) — Geifman and El-Yaniv (2017): coverage and selective risk.
- [How Can We Know When Language Models Know?](https://arxiv.org/abs/2012.00955) — Jiang et al. (TACL 2021): generative QA calibration.
- [Language Models (Mostly) Know What They Know](https://arxiv.org/abs/2207.05221) — Kadavath et al. (2022): P(True) and P(IK), including transfer limitations.
- [TruthfulQA](https://arxiv.org/abs/2109.07958) — Lin, Hilton, and Evans (ACL 2022): imitative-falsehood slice.
- [FActScore](https://arxiv.org/abs/2305.14251) — Min et al. (EMNLP 2023): atomic-fact factual precision.

**CURRENT DEFAULT**

- Explicit event/population definitions, held-out calibration, risk-coverage reporting, slice-based regression, and versioned evaluator provenance are baseline practices.
- No one uncertainty signal, ECE binning, threshold, or “hallucination metric” is a universal default.

**WORKLOAD-DEPENDENT**

- Temperature scaling, verbalized confidence, learned probes, sample disagreement, semantic entropy, tool/retrieval checks, abstention, and escalation.
- The correct operating point depends on costs, severity, user workflow, coverage target, and available fallback capacity.

**FRONTIER**

- [Semantic Uncertainty: Linguistic Invariances for Uncertainty Estimation in Natural Language Generation](https://arxiv.org/abs/2302.09664) — Kuhn, Gal, and Farquhar (ICLR 2023).
- [Language Models with Conformal Factuality Guarantees](https://arxiv.org/abs/2402.10978) — Mohri and Hashimoto (2024).
- [Calibrated Language Models Must Hallucinate](https://arxiv.org/abs/2311.14648) — Kalai and Vempala (STOC 2024): theoretical result with explicit fact/calibration scope.
- 2025–2026 semantic-uncertainty probes, efficient estimators, and trajectory readers remain frontier and workload-dependent; benchmark results do not establish a production default.

**LEGACY / INSUFFICIENT WHEN USED ALONE**

- Treating raw token probability as answer correctness probability.
- Reporting accuracy without calibration, or ECE without accuracy/ranking/bin details.
- Reporting selective risk without coverage and abstention cost.
- Treating one fixed benchmark as a complete behavior contract.
- Using an LLM judge or detector score as its own ground truth.

**PRODUCTION SOURCE TRACE**

- Repository: `huggingface/transformers`
- Revision: `27166ea03f12c940f23176a904ab1d2ff1a3dcbb`
- Verified: 2026-09-26 by static inspection; not executed here.
- File: `src/transformers/generation/utils.py`.
- Symbols: `GenerateDecoderOnlyOutput`, `GenerationMixin.generate`, `GenerationMixin.compute_transition_scores`.
- Observed behavior: generation outputs can retain processed per-step scores and raw logits when requested; `compute_transition_scores` stacks/gathers selected-token scores and optionally applies `log_softmax`.
- Generalization: these are generation-path scores at a pinned upstream revision. They do not become calibrated answer-correctness probabilities without an explicit event, aggregation, labels, and validation.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow:

$$
\text{PREDICT}\to\text{BUILD}\to\text{MEASURE}\to\text{EXPLAIN}
\to\text{BREAK}\to\text{IMPROVE}\to\text{FALSIFY}.
$$

### LAB A — Claim-Level Failure Taxonomy

- **Objective:** annotate long-form outputs under an explicit evidence/time contract and compare response-, claim-, and severity-weighted metrics.
- **Variables:** prompt family, evidence availability, knowledge date, answer length, domain, and model/prompt revision.
- **Measurements:** atomic-claim precision and required-claim recall, contradiction, stale/unsupported attribution, omission, refusal, instruction compliance, severity, inter-annotator agreement, and adjudication.
- **Break/falsify:** include ambiguous claims, multiple valid phrasings, changing facts, unanswerable prompts, correct but uncited statements, and retrieved evidence conflicts. Show a case where one response-level label hides mixed claim behavior.
- **Artifact:** versioned annotation guide, raw labels, adjudication log, evaluator error analysis, and intervention mapping.
- **Effort:** 4h.

### LAB B — Calibration Under Controlled Shift

- **Objective:** construct answer-confidence signals and compare raw, temperature-scaled, self-reported, and optional learned confidence.
- **Split:** separate fit, calibration, in-distribution test, and shifted test by the deployment unit; no threshold or temperature tuning on test.
- **Measurements:** accuracy, Brier, NLL, declared-bin ECE with uncertainty, reliability plot, ranking metric, subgroup calibration, and risk-coverage.
- **Break/falsify:** shift prompt format, domain, language, time, and model version; change length normalization; find equal-ECE models with different decision utility or an aggregate-calibrated model with subgroup error.
- **Artifact:** confidence semantics, source trace, calibration code/tests, raw predictions, bins/counts, shift matrix, and recalibration trigger.
- **Effort:** 5h.

### LAB C — Abstention and Consistency Illusions

- **Objective:** compare likelihood, P(True), black-box disagreement, and semantic uncertainty as routing signals.
- **Variables:** sample count, temperature, equivalence method, threshold, task ambiguity, and fallback action.
- **Measurements:** detector discrimination/calibration, coverage-risk, sample cost, latency, false accept/reject severity, subgroup coverage, fallback success, and escalation load.
- **Break/falsify:** stable false misconceptions, diverse correct paraphrases, aliases, multiple correct answers, and adversarially confident false claims. Use independent labels, not consensus, as correctness.
- **Artifact:** raw generations, semantic clusters with audits, routing frontier, counterexample set, and selected operating point with rejected alternatives.
- **Effort:** 5h.

### LAB D — Release Regression and Conformal Scope

- **Objective:** compare two pinned releases on anchor/time/slice suites, then implement either a conformal back-off/set policy or a prespecified abstention gate.
- **Measurements:** paired behavior deltas, severity, calibration/proper scores, marginal and subgroup coverage, risk-coverage, temporal validity, evaluator agreement, and system cost.
- **Break/falsify:** alter exchangeability through temporal/domain shift, change evaluator revision, inject a retrieval outage, and inspect a slice whose regression is hidden by the mean. Verify whether the stated guarantee still applies before interpreting coverage.
- **Artifact:** release manifest, frozen thresholds/calibration set, regression report, assumption audit, incident decision, and rollback/recalibration criteria.
- **Effort:** 5h.

## 07 Break / Incident Scenarios

### Incident 07.1 — The Aggregate Dashboard Is Green

A new model/prompt release has unchanged overall exact-match accuracy and lower reported ECE. Production complaints show confidently wrong time-sensitive answers in one language, more abstentions for a high-value subgroup, and citations that look plausible but do not support the claims. The automatic judge was also upgraded.

The learner must:

1. **Compete hypotheses:** true model regression; changed confidence semantics; prompt/decoding shift; temporal knowledge drift; retrieval/index outage; judge drift; label-policy change; traffic mix; answer-normalization bug; or subgroup threshold mismatch.
2. **Recover evidence:** paired old/new raw outputs, prompt/model/runtime/retrieval/judge versions, claim/evidence records, as-of dates, confidence and bins, coverage-risk by slice, abstention destination, sampled human adjudication, and queue/cost effects.
3. **Discriminate:** rescore both releases with both evaluator versions and an audited sample; replay the same prompts/evidence; freeze retrieval; inspect score distributions; recompute calibration and coverage on matched slices; vary only the suspected component.
4. **Rank causes:** use matched deltas, temporal order, label audit, and uncertainty. Aggregate stability and lower ECE are not exculpatory.
5. **Intervene:** rollback/gate the affected route, repair evidence or evaluator, recalibrate thresholds, refresh time-sensitive sources, or change fallback capacity based on evidence.
6. **Remeasure:** repeat the identical slice, calibration, coverage, severity, citation-support, and operational contract with preregistered promotion criteria.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Transfer Problem — Build a Trustworthy Answering Gate

Design an uncertainty and regression system for a multilingual assistant answering both stable and time-sensitive factual questions. It can answer, retrieve/check, abstain, or escalate.

Deliver:

1. behavior contracts and a non-overlapping-enough failure taxonomy with evidence, time, unit, severity, and ambiguity rules;
2. a representative sampling/split plan and label/adjudication protocol;
3. at least three distinct confidence signals with exact semantics and cost;
4. calibration equations, proper scores, binning rules, uncertainty, ranking, and subgroup analyses;
5. risk-coverage and utility for every fallback, including rejected-user and capacity costs;
6. shift tests across prompt, language, time, domain, retrieval state, and model version;
7. a consistency/semantic-uncertainty counterexample analysis using independent labels;
8. an optional conformal guarantee statement copied into system terms with every assumption and non-guarantee;
9. a pinned runtime source trace explaining why generation scores are not correctness probabilities;
10. versioned release gates, canary, rollback, evaluator audit, and post-intervention remeasurement.

## 09 Required Evidence & Rubric

### Required Artifact: Behavior and Uncertainty Record

Each record must join prompt/task and all component revisions, allowed evidence and date, raw/sample output, atomic claims and labels, category/severity/adjudication, every confidence signal and transformation, acceptance/fallback action, evaluator provenance, and observed downstream outcome where lawful and available.

### Rubric Dimensions

- **Semantic precision:** events, evidence, time, population, and units are explicit.
- **Calibration discipline:** no score is called probability of correctness without held-out evidence; bins and proper scores are reproducible.
- **Decision quality:** risk, coverage, abstention/escalation utility, subgroup impact, and capacity are co-reported.
- **Adversarial reasoning:** stable falsehoods, paraphrases, ambiguous/multiple answers, shift, and judge failure are tested.
- **Guarantee discipline:** marginal, conditional, per-item, and post-shift claims remain distinct.
- **Diagnosis:** competing model/pipeline/evaluator/population causes are separated by matched experiments.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Behavior contract and taxonomy | Lesson 7.1 | LAB A | Mastery 1–2 | Annotation guide and adjudication |
| Confidence and calibration | Lesson 7.2 | LAB B | Mastery 3–4, 9 | Proper scores, reliability, source trace |
| Selective prediction | Lesson 7.3 | LAB C | Mastery 5 | Risk-coverage and fallback utility |
| Free-form uncertainty | Lesson 7.4 | LAB C | Mastery 3, 7 | Counterexamples and independent labels |
| Conformal guarantee scope | Lesson 7.5 | LAB D | Mastery 8 | Assumption-to-system audit |
| Behavior regression | Lesson 7.6 | LAB D | Incident / Mastery 6, 10 | Paired release and evaluator audit |

## 11 Exit Criteria & Module Wrap-Up

A learner passes when they can:

1. define a failure without relying on the word hallucination;
2. distinguish factuality, truthfulness, correctness, compliance, usefulness, and safety;
3. derive calibration, ECE, Brier/NLL, and risk-coverage with assumptions;
4. show why likelihood and verbal confidence need event-specific calibration;
5. select and defend abstention/escalation thresholds under shift and subgroup constraints;
6. break sample-consistency and semantic-uncertainty methods with realistic counterexamples;
7. state conformal guarantees without per-item or post-shift inflation;
8. trace generation-score source code at a pinned revision;
9. diagnose a hidden behavior regression and remeasure after intervention.

**Final mental model:** model uncertainty is not a property emitted by one number. It is a validated relationship between a score, an event, a population, and a decision. Reliability comes from maintaining that relationship under version and distribution change—and refusing to expand its scope beyond the evidence.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

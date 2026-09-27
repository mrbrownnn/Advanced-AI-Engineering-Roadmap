# Module 00 — Scientific AI Engineering

## 00 Why This Module Exists

An advanced AI system can produce convincing benchmark numbers while measuring the wrong population, hiding failures, confounding the treatment with machine state, or applying a valid formula to dependent observations. This module establishes the evidence discipline used by every later module.

The goal is not to turn every engineer into a statistician. The goal is to make engineering claims auditable: define the quantity, preserve the observations, state assumptions, design a comparison that can fail, and connect the result to a decision threshold.

**Module Orientation**

- **Engineering Problem**: Decide whether a measured change is real, practically important, and likely to transfer to the intended workload.
- **What You Will Do**: Build a timestamped benchmark harness, expose warmup and order bias, compare randomized paired treatments, quantify uncertainty without overstating it, break a load generator with coordinated omission, trace MLPerf LoadGen, and package a repeatable artifact.
- **Environment**: Python 3.10+ for harness and analysis work; a GPU is optional. Hardware-specific experiments must record the accelerator, driver, runtime, clocks/power policy, and isolation state.
- **Research Cutoff**: 2026-09-27. Canonical statistical methods remain relevant; current implementation claims remain pinned to the verified revisions below.

## 01 Baseline Assumptions

Prerequisites are basic algebra, basic probability, Python, and command-line use. This module introduces the curriculum-wide research contract:

- **O — Source observation**: what a source or experiment actually reports, scoped to its setup.
- **D — Derivation**: what follows from stated premises and assumptions.
- **H — Engineering hypothesis**: a causal explanation with a prediction, discriminating measurement, and falsifier.

This module teaches general measurement and experimental design. Domain-specific model evaluation belongs primarily to Module 15; GPU execution and profiler semantics to Module 02; serving queueing models to Module 04.

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
  security: NOT_APPLICABLE
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: REQUIRED

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 10h
  assessment: 3h
  source_trace: 2h
  total: 23h
```

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
Decision and target population
        ↓
Atomic claim + practical threshold
        ↓
Measurand, unit, boundary, success/failure semantics
        ↓
Workload + treatment + controls + experimental unit
        ↓
Randomized/blocked execution with state and provenance capture
        ↓
Raw event-level observations, failures, censoring, and metadata
        ↓
Estimator + uncertainty model + assumption checks
        ↓
Competing explanations + discriminating intervention
        ↓
Decision, rollback rule, and same-boundary remeasurement
```

Feedback paths matter. A slow trial can heat the device, change frequency, grow a queue, alter caching, or cause a response-coupled load generator to issue fewer requests. The measurement process can therefore change the state it intends to observe. Treat those paths as hypotheses and instrument them.

## 04 Lessons

### Lesson 0.1 — Claims, Estimands, and Evidence Types

**Engineering Question:** What exactly must be true for the result to justify a decision?

**Concepts & Definitions:**

A benchmark begins with a decision and an **estimand**: the population quantity the experiment aims to estimate. “Latency improved” is not an estimand. A usable statement might be:

> For successful requests from workload version W under offered load A, configuration B changes client-observed P95 end-to-end latency relative to A by an amount that matters operationally, while preserving the declared quality and failure criteria.

Define:

- target population and workload version;
- treatment and baseline;
- request and response boundaries;
- units and aggregation level;
- success, timeout, cancellation, retry, and exclusion semantics;
- practical threshold that would change the decision.

Keep O, D, and H separate. A paper's measured speedup is O. A FLOP or memory equation is D. “The speedup came from higher cache hit rate” is H until aligned telemetry and an intervention support it.

**Worked Example:** The quoted statement becomes testable only after W, A, configurations A/B, the quantile convention, the quality gate, and the practical threshold are instantiated.

**Knowledge Check:**
1. Which part of “B is faster because cache reuse increased” is O, D, or H?
2. Why does a statistically detectable effect not automatically justify deployment?

**Guided Practice:** Rewrite these claims atomically and label each O, D, or H:

1. “The new runtime is 20% faster.”
2. “P99 improved, so users will prefer it.”
3. “The kernel is memory-bound because GPU utilization is low.”

**Feedback Contract:** A strong answer adds population, workload, boundary, estimator, comparison, uncertainty, and limitations; it does not convert a correlation into a mechanism.

**Learning Outcome:** Write atomic, decision-linked claims while preserving the distinction among observation, derivation, and hypothesis.

*(Effort: 35m instruction, 20m practice)*

---

### Lesson 0.2 — Harness Mechanics and Measurement Boundaries

**Engineering Question:** Which timestamps and state transitions produce the reported value?

**Concepts & Definitions:** A measurement boundary names the clocks, lifecycle states, population, and inclusion rules that give an observed duration meaning.

**Mechanism Explanation:**

A harness should make its lifecycle observable:

1. pin code, data, configuration, dependencies, and hardware identity;
2. initialize the system and record initialization separately;
3. warm to a declared state or intentionally measure cold start;
4. schedule work according to a declared arrival model;
5. synchronize asynchronous execution at the observation boundary;
6. record intended issue, actual issue, start, completion, failure, and cancellation events;
7. preserve raw event data before aggregation.

**Quantitative Model / Derivation:**
For one observation, the measured interval is

$$Y_i=t_{end,i}-t_{start,i}.$$

This identity is exact only for the declared clocks and boundaries. It does not prove that $Y_i$ is GPU execution time, user-visible latency, or service time. Timer resolution, clock domain, asynchronous queues, batching, buffering, and instrumentation overhead can change the meaning.

**Warmup is part of the question.** Cold-start and steady-state performance are different estimands. Do not discard early samples merely because they are slower; declare which state matters and report transition behavior when it matters operationally.

**Coordinated omission:** If the generator waits for a response before issuing the next intended request, a long stall suppresses arrivals and therefore suppresses latency samples. Record an independent intended-arrival schedule when the production question assumes arrivals independent of completions.

**Worked Example:** If intended issue is 0 ms, actual issue is 20 ms, device work is 35–45 ms, and client receipt is 60 ms, intended-arrival latency, server residence, and device time are different measurements.

**Knowledge Check:** Why can a host timer around an asynchronous launch understate execution time, and which event reveals coordinated omission?

**Guided Practice:** Draw timestamp boundaries for client-observed latency, server residence, device execution, and post-processing. Identify which pairs of timestamps share a clock.

**Feedback Contract:** A strong answer names event boundaries, clock domains, inclusion rules, and intended-versus-actual issue times; it does not call submission delay device execution.

**Learning Outcome:** Implement and defend a lifecycle-aware timing contract that preserves cold state, failures, and intended arrivals.

*(Effort: 40m instruction, 25m practice)*

---

### Lesson 0.3 — Experimental Units, Randomization, Blocking, and Pairing

**Engineering Question:** What is actually independent, and what nuisance factors can reverse the comparison?

**Concepts & Definitions:**

The **experimental unit** is the smallest unit independently assigned to a treatment. Ten thousand requests inside one process are not ten thousand independent process-level replications. Generalization across seeds, model loads, machines, or days requires repetition at those levels.

Use:

- **control** to hold an important factor fixed;
- **blocking** to compare treatments within a nuisance-factor level;
- **randomization** to avoid systematically aligning treatment with uncontrolled drift;
- **pairing** to analyze within-block differences when observations are meaningfully matched.

**Quantitative Model / Derivation:**
For paired block differences $d_i=Y_{B,i}-Y_{A,i}$:

$$\bar d=\frac{1}{n}\sum_i d_i,\qquad SE(\bar d)=\frac{s_d}{\sqrt n}.$$

An approximate classical interval is

$$\bar d\pm t_{1-\alpha/2,n-1}\frac{s_d}{\sqrt n},$$

under the stated independence and distribution assumptions for the block-level differences.

**Worked Example:** Four independent blocks produce latency differences $[1.2,0.9,1.1,0.8]$ ms. Then $\bar d=1.0$ ms, $s_d\approx0.183$ ms, and the 95% t interval is approximately $1.0\pm3.182(0.183/2)=[0.71,1.29]$ ms. Four blocks are still weak evidence for transfer; the calculation does not manufacture independence.

**Knowledge Check:** Why are many requests in one process not necessarily independent process replications, and when is pairing invalid?

**Independent Practice:** Run A then B repeatedly without randomized order; then randomize order within fresh-process blocks and compare conclusions. CPU/GPU temperature, frequency, caches, allocator state, and background work can become treatment labels.

**Feedback Contract:** Identify the assignment unit, block factor, carryover check, paired differences, and transfer limits. Do not replace independent replications with correlated observations.

**Learning Outcome:** Design randomized, blocked, or paired comparisons at the level that supports the intended claim.

*(Effort: 45m instruction, 30m practice)*

---

### Lesson 0.4 — Distributions, Quantiles, Failures, and Outliers

**Engineering Question:** Which summary preserves the user-visible behavior relevant to the decision?

**Concepts & Definitions:**

Mean, median, quantiles, maximum, throughput, and failure rate answer different questions. A mean cannot guarantee P99. A P99 does not describe the worst case. A percentile is incomplete without:

- population and observation boundary;
- quantile convention;
- sample count and dependence structure;
- treatment of timeouts, errors, censoring, and dropped requests;
- uncertainty or stability across independent runs.

**Quantitative Model / Derivation:**
For an independent event with probability $q$, the chance of observing it at least once in $n$ trials is

$$P(\text{at least one})=1-(1-q)^n.$$

If $q=0.01$, achieving a 95% chance of seeing at least one such event requires

$$n\ge\frac{\ln(0.05)}{\ln(0.99)},$$

so $n=299$ after rounding upward. This is only an exposure check. It does not produce a precise confidence interval for P99.

**Worked Example:** The 299-trial result is an exposure calculation, not evidence that the maximum of 299 observations is a stable P99 estimator.

**Outliers are evidence until explained.** A slow observation may be clock corruption, a GC pause, thermal throttling, a retry, a real failure mode, or a valid heavy-tail sample. Keep raw data. Apply only predeclared rules, record reasons, and report sensitivity with and without exclusions or with robust estimators.

**Knowledge Check:** Why can a mean improve while P99 worsens, and what changes when timeouts disappear from the denominator?

**Guided Practice:** Given a run with 2,000 successes, 40 timeouts, and 10 client cancellations, define two defensible metrics for different decisions. Explain why silently dropping the 50 incomplete requests changes the population.

**Feedback Contract:** State population, quantile convention, denominator, censoring policy, and sensitivity analysis; never report only successful latency without naming the excluded population.

**Learning Outcome:** Select summaries and failure semantics that preserve the behavior relevant to the decision.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 0.5 — Confidence Intervals, Sample Size, and Practical Significance

**Engineering Question:** How uncertain is the estimate, and is the plausible effect large enough to matter?

**Concepts & Definitions:** An interval procedure, effect size, minimum practically important effect, and decision rule answer different questions.

**Quantitative Model / Derivation:**

For independent observations from a normal population with unknown variance, the classical interval for a mean is

$$\bar x\pm t_{1-\alpha/2,n-1}\frac{s}{\sqrt n}.$$

A 95% confidence procedure has approximately 95% long-run coverage under its assumptions. It does not mean there is a 95% posterior probability that the fixed parameter lies inside this realized interval.

**Worked Example:** With $n=25$, $\bar x=10$ ms, $s=2$ ms, and $t_{0.975,24}\approx2.064$, the half-width is $2.064(2/5)=0.826$ ms. Report approximately $[9.17,10.83]$ ms, conditional on the model. This is about an 8.3% half-width relative to the observed mean, not 5%.

For known $\sigma$ and target absolute half-width $E$:

$$n\ge\left(\frac{z_{1-\alpha/2}\sigma}{E}\right)^2.$$

With $\sigma=2$ ms, $E=0.5$ ms, and 95% confidence, the normal approximation gives $n\ge(1.96\times2/0.5)^2=61.47$, hence 62. If $\sigma$ came from a small pilot, 62 is a planning approximation. This formula does not size P99, ratios, clustered data, multiple comparisons, or statistical power for a minimum effect.

Report effect size and uncertainty against a **minimum practically important effect**. A small p-value does not measure effect size, practical value, or the probability that a hypothesis is true.

When observations are dependent or hierarchical, analyze at the correct level or use a model/resampling plan that preserves the dependency structure. Naively bootstrapping individual requests from one process does not create independent process replications.

**Knowledge Check:** What does a 95% frequentist procedure claim, and why can a narrow interval still be operationally irrelevant?

**Guided Practice:** Choose a minimum practically important effect and use pilot variance only as a planning input. Explain what changes when observations are clustered by process.

**Feedback Contract:** Report effect units, assumptions, practical threshold, experimental unit, and planning-versus-analysis status; do not use a mean interval to guarantee P99.

**Learning Outcome:** Quantify uncertainty without overstating probability, independence, tail coverage, or practical significance.

*(Effort: 50m instruction, 25m practice)*

---

### Lesson 0.6 — Provenance, Repeatability, and Source Tracing

**Engineering Question:** Could another engineer understand, rerun, and challenge the result?

**Concepts & Definitions:** Provenance connects a metric to code, data, configuration, environment, raw events, and transformation steps.

**Mechanism Explanation:**

The artifact should include:

- code revision and dirty-tree state;
- dependency lock or image identity;
- model, tokenizer, dataset, prompt, and workload versions;
- hardware, driver, firmware, clocks/power policy, topology, and resource isolation;
- full configuration and random seeds;
- raw event data, logs, failures, and checksums;
- analysis code and generated tables/plots;
- exact commands and known limitations.

This module adopts ACM's current terminology and cites it explicitly:

- **repeatability**: same team and setup;
- **reproducibility**: different team using the same setup/artifacts;
- **replicability**: different team with an independently developed setup.

Other communities have used the last two terms differently, so never rely on the word alone.

**Worked Example:** At MLCommons Inference commit `3fbc329939999c13d0a7b5e67fb2092287e06047`, trace:

1. `loadgen/loadgen.cc::StartTest` through sanitized settings and scenario/mode dispatch;
2. `IssueQueries` to `loadgen/issue_query_controller.cc::IssueQueryController::StartIssueQueries`;
3. SUT completion through `loadgen/loadgen.cc::QuerySamplesComplete`;
4. recorded latencies into `loadgen/results.cc::PerformanceSummary::ProcessLatencies`.

Describe the actual timestamp and percentile boundaries. Do not claim that MLPerf's workload contract is the universal definition of inference performance.

**Knowledge Check:** Why is a commit hash insufficient when generated artifacts or the worktree differ, and which path establishes completion timing?

**Independent Practice:** Produce the pinned trace, provenance manifest, and exact rerun commands. Mark unexecuted branches `TODO_VERIFY`.

**Feedback Contract:** Record repository, revision, dirty state, exact symbols, execution path, environment, raw output, and static-versus-executed status.

**Learning Outcome:** Package a result so another engineer can audit its lineage, rerun it, and identify unresolved verification.

*(Effort: 40m instruction, 30m practice, source-trace integration)*

---

### Lesson 0.7 — Diagnostic Reasoning and Falsification

**Engineering Question:** What evidence would make the favored explanation wrong?

**Concepts & Definitions:** A useful diagnosis is a ranked causal explanation with alternatives, missing evidence, predicted observations, and a measurement or intervention that can weaken it.

**Mechanism Explanation:**

Use this loop:

```text
SYMPTOM
  → COMPETING HYPOTHESES
  → MISSING EVIDENCE
  → DISCRIMINATING MEASUREMENT OR INTERVENTION
  → RANKED EXPLANATION
  → CHANGE
  → SAME-BOUNDARY REMEASUREMENT
```

Example symptom: treatment B improves mean latency but worsens P99.

Competing hypotheses include cache warmup, batching changes, thermal drift, retries, rare long inputs, load-generator omission, or measurement corruption. GPU utilization, one profiler screenshot, or one correlation is necessary in some investigations but insufficient to select one mechanism. Predict what each hypothesis should change, intervene on one causal link, and preserve alternatives that remain observationally equivalent.

**Worked Example:** If randomizing order removes the mean improvement but leaves the P99 regression, order drift is strengthened for the mean while a separate tail mechanism remains unresolved.

**Knowledge Check:** What observation weakens thermal drift, and why is lower GPU utilization not a unique cause?

**Guided Practice:** Build a matrix of hypotheses, predicted telemetry, discriminating interventions, falsifiers, and unresolved equivalence for the example symptom.

**Feedback Contract:** Require at least three alternatives, aligned evidence, a controlled intervention, ranked support, and explicit uncertainty.

**Learning Outcome:** Produce an evidence-backed explanation that states what was ruled out, what remains uncertain, and what next measurement would change the decision.

*(Effort: 40m instruction, 25m practice)*

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- NIST/SEMATECH Engineering Statistics Handbook: confidence intervals, experimental design, blocking, and outlier assumptions.
- Mytkowicz et al. (ASPLOS 2009), *Producing Wrong Data Without Doing Anything Obviously Wrong*: setup-induced measurement bias.
- Georges, Buytaert, and Eeckhout (OOPSLA 2007), *Statistically Rigorous Java Performance Evaluation*: startup, steady state, and repeated execution.
- Kalibera and Jones (ISMM 2013), *Rigorous Benchmarking in Reasonable Time*: hierarchical variance and experiment dimensioning.
- Wasserstein and Lazar (2016), *The ASA's Statement on p-Values*: limits of p-value interpretation.

**CURRENT IMPLEMENTATION SNAPSHOTS**

- MLCommons Inference LoadGen commit `3fbc329939999c13d0a7b5e67fb2092287e06047`, verified 2026-09-25.
- HdrHistogram commit `de84b0a7de2378abfc405da503bf4898e84ea98e`, verified 2026-09-25.

**WORKLOAD-DEPENDENT**

- Warmup duration, number of repetitions, load model, percentile estimator, block factors, and practical thresholds.
- Whether a closed-loop load generator is representative of the real producer/client system.

**LEGACY / REJECT**

- Best-of-N reporting without a declared estimand.
- Deleting slow observations solely because they look anomalous.
- Treating thousands of correlated requests as independent machine-level replications.
- Claiming a universal sample count for “95% confidence and 5% error.”

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

Every lab follows:

$$\text{PREDICT}\to\text{BUILD}\to\text{MEASURE}\to\text{EXPLAIN}\to\text{BREAK}\to\text{IMPROVE}\to\text{FALSIFY}$$

### LAB A — Event-Level Benchmark Harness

- **Objective**: Build a harness around a controllable noisy operation with event-level timestamps and raw JSONL/CSV output.
- **Pre-Registered Hypothesis**: A response-coupled generator will under-observe latency during an injected stall relative to an independent intended-arrival schedule.
- **Independent Variables**: Load-generation mode, cold/warm state, stall injection, and observation boundary.
- **Dependent Variables**: Intended/actual issue lag, latency, throughput, failures, and declared quantiles.
- **Required controls**: cold versus warm state, timer and clock identity, synchronization boundary, intended and actual issue times, success/failure status, machine metadata, and immutable configuration.
- **Break & Falsify**: Introduce a 2-second pause while comparing response-coupled and independent-arrival generators.
- **Evidence**: raw trace, aggregation code, mean/median/quantiles/failures, quantile convention, and an explanation of coordinated omission.
- **Alignment**: Lessons 0.1, 0.2, and 0.4.
- **Effort Estimate**: 2h implementation, 1h analysis.

### LAB B — Randomized Paired Benchmark

- **Objective**: Compare baseline A and treatment B across independent blocks such as fresh processes or time windows.
- **Independent Variables**: Treatment and randomized within-block order; nuisance-factor levels are block metadata.
- **Dependent Variables**: Paired differences, uncertainty, carryover/order effects, and threshold decisions.
- **Design**: randomize A/B order within each block, record nuisance factors, retain paired differences, and test an A-then-B order as a deliberately biased comparator.
- **Pre-Registered Hypothesis**: State the mechanism, predicted telemetry, practical effect threshold, and falsifier before running.
- **Break & Falsify**: Add a monotonic background load or thermal drift and show when fixed order reverses or exaggerates the conclusion.
- **Alignment**: Lessons 0.3 and 0.5.
- **Effort Estimate**: 2h implementation, 1h analysis.

### LAB C — Uncertainty and Tail Audit

- **Objective**: Compare a classical mean interval, a correctly structured resampling or hierarchical analysis, and a tail-exposure calculation.
- **Pre-Registered Hypothesis**: Independent process starts support a process-level claim more directly than additional correlated requests within one process.
- **Independent Variables**: Requests per process, process starts, resampling unit, and synthetic tail-event probability.
- **Dependent Variables**: Interval behavior, effective sample structure, and tail-exposure probability.
- **Design**: vary requests per process and number of independent process starts while holding total request count similar.
- **Break & Falsify**: Demonstrate that more within-process requests can narrow a naive interval without adding process-level evidence.
- **Evidence**: assumptions, effective experimental unit, interval method, sensitivity analysis, and no percentile guarantee derived from a mean formula.
- **Alignment**: Lessons 0.3–0.5.
- **Effort Estimate**: 2h analysis, 1h report.

### LAB D — Reproducible LoadGen Source Trace

- **Objective**: Build or inspect the pinned MLPerf LoadGen revision and produce a source trace for one scenario.
- **Pre-Registered Hypothesis**: Changing an issue, completion, or failure boundary can change the metric while SUT work is held fixed.
- **Independent Variables**: Scenario, boundary/failure policy, and instrumented versus uninstrumented execution.
- **Dependent Variables**: Issued/completed counts, latency population, aggregate metrics, and trace completeness.
- **Required trace**: settings sanitation, scenario dispatch, request issue, completion timestamp, result processing, and relevant output artifacts.
- **Break & Falsify**: Change one boundary or failure policy and show how the reported metric changes even when the SUT is unchanged.
- **Artifact**: exact commands, patches if any, raw logs, code revision, build environment, and a `TODO_VERIFY` for any path not executed.
- **Alignment**: Lessons 0.2 and 0.6.
- **Effort Estimate**: 2h source trace, 1h boundary experiment.

## 07 Break / Incident Scenarios

### Incident 00.1: The 18% Optimization That Disappeared

A team reports an 18% latency improvement after replacing a runtime component. The baseline always ran first, treatment second. Only successful requests were logged. Each configuration processed 50,000 requests in one long process. A rerun on another machine shows no gain, and production P99 worsens.

**Diagnostic Protocol (Task):**
The learner must:

1. enumerate competing explanations: warmup, thermal/frequency drift, cache state, retry/failure filtering, process-level pseudoreplication, workload mismatch, or a real machine interaction;
2. identify missing raw data and provenance;
3. design a randomized blocked rerun with explicit success/failure semantics;
4. choose estimands and practical thresholds before seeing the new result;
5. use aligned state telemetry to rank explanations;
6. preserve a result that contradicts the original claim;
7. propose a rollback and same-boundary production remeasurement.

The final response must separate immediate rollback from long-term measurement correction and define quantitative evidence that would allow reconsideration.

No single metric is declared the root cause in advance.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Transfer Problem — Defend a Decision-Ready Benchmark

Given two AI inference configurations and a noisy heterogeneous workload, produce a decision-ready benchmark package.

**Required deliverables**

1. Atomic decision claim, target population, O/D/H labels, and minimum practical effect.
2. Measurement contract with timestamp boundaries, units, workload, failures, retries, censoring, and exclusions.
3. Experimental-unit argument and randomized blocked design.
4. Raw event-level data and provenance manifest.
5. Mean and distribution summaries with justified uncertainty methods.
6. A tail audit that states the quantile convention and why the sample supports—or does not support—the tail claim.
7. At least three competing causal explanations and one discriminating intervention.
8. Reanalysis after intentionally introducing order bias or coordinated omission.
9. Pinned production source trace.
10. Decision, limitations, rollback threshold, and reproduction instructions.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit the pinned MLPerf LoadGen trace from Lesson 0.6/Lab D with repository, revision, verification date, exact files and symbols, execution path, static-versus-executed status, commands, and `TODO_VERIFY` markers.

### Rubric Dimensions

- **Claim discipline**: Strong work never promotes O to a universal mechanism, D to an empirical result, or H to verified fact.
- **Measurement validity**: Strong work defines clock and lifecycle boundaries and records all outcomes, not just successes.
- **Design validity**: Strong work identifies the experimental unit, blocks important nuisance factors, randomizes order, and checks carryover.
- **Statistical reasoning**: Strong work states assumptions, reports effect and uncertainty, and refuses to derive tail guarantees from mean formulas.
- **Failure analysis**: Strong work ranks alternatives using discriminating evidence and retains unexplained anomalies.
- **Reproducibility**: Strong work provides pinned versions, raw data, exact commands, and honest `TODO_VERIFY` markers.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Atomic O/D/H claims | 0.1 | Labs B-C | Mastery 1 | Claim table |
| Measurement boundaries | 0.2 | Labs A-D | Mastery 2 | Event schema and trace |
| Randomized blocked design | 0.3 | Lab B | Mastery 3 | Design and randomization record |
| Distribution and tail reasoning | 0.4 | Labs A-C | Mastery 5-6 | Raw outcomes and quantile audit |
| Conditional uncertainty | 0.5 | Lab C | Mastery 5 | Derivation and interval report |
| Artifact provenance | 0.6 | Lab D | Mastery 4, 9-10 | Manifest and reproduction log |
| Falsification diagnosis | 0.7 | All labs | Mastery 7-8 | Competing-hypothesis matrix |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner can exit Module 00 when they can:

1. define what was measured and what decision population it represents;
2. identify the true experimental unit and major nuisance factors;
3. implement a harness that records intended arrivals, actual timing, outcomes, and provenance;
4. explain warmup, state drift, pseudoreplication, tail uncertainty, and coordinated omission;
5. calculate and correctly interpret a conditional mean interval and sample-size approximation;
6. preserve failures and anomalies instead of optimizing the dataset for a desired result;
7. falsify a favored explanation with a controlled intervention;
8. hand another engineer an artifact they can audit and rerun.

### Module Wrap-Up (Final Mental Model Reconstruction)

The final invariant is simple: **a precise number is not strong evidence unless the measurement contract, design, assumptions, and failure semantics make it answer the intended question.**

## 12 Competency Targets

```yaml
competency:
  sfia: 4-5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

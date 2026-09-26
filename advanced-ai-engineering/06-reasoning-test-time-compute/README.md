# Module 06 — Reasoning & Test-Time Compute

## 00 Why This Module Exists

A model can spend inference-time work in several different ways: generate a longer trajectory, sample independent candidates, branch and backtrack, call tools, score final answers, or verify intermediate steps. These mechanisms do not buy the same thing. More generated tokens are not automatically more useful reasoning; an oracle finding that one of many candidates is correct is not a deployable selection result; and parallel work can increase total compute without increasing critical-path latency by the same factor.

This module teaches the complete system:

$$
\text{task} \to \text{candidate generation} \to \text{compute allocation}
\to \text{verification/selection} \to \text{stopping}
\to \text{answer + quality + cost + latency}.
$$

The feedback path matters. A policy that samples more candidates can increase accelerator demand, reduce serving headroom, lengthen queues, trigger timeouts, cancel unfinished branches, and change the candidate distribution seen by the verifier. Those effects are hypotheses until joined request, candidate, verifier, and serving telemetry support them.

The scope is inference-time reasoning, search, verification, budgeting, and diagnosis. Model-behavior uncertainty is developed in Module 07; training and model adaptation in Module 19; serving scheduling and capacity in Module 04; and inference-kernel optimization in Module 05.

**Research cutoff:** 2026-09-26.

**Module orientation**

- **Engineering problem:** choose and operate a reasoning policy that improves declared task utility under quality, cost, latency, capacity, and safety constraints.
- **What you will do:** instrument chain-of-thought behavior; implement self-consistency and best-of-N; derive and break pass@k assumptions; build verifier-guided search; implement budget and stopping controls; test faithfulness and verifier gaming; and diagnose an offline-to-production regression.
- **Evidence rule:** label source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**). A paper result remains scoped to its model, tasks, decoding, verifier, budget, and evaluation boundary.

## 01 Baseline Assumptions

- **Model internals (Module 01):** autoregressive factorization, logits, sampling, temperature, context limits, and the distinction between generated text and latent computation.
- **Serving (Module 04):** TTFT, ITL/TPOT, end-to-end latency, throughput, goodput, queueing, concurrency, and capacity headroom.
- **Inference optimization (Module 05):** generated-token and model-call costs are workload- and implementation-dependent; token counts are not interchangeable with FLOPs, elapsed time, or money.
- **Mathematics:** conditional probability, complements, combinations, binomial distributions, expected value, and explicit units.
- **Evaluation discipline (Module 00):** preregister the population, metric, baseline, stopping rule, and practically important effect; preserve raw candidate-level records.

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

By the end, the learner must be able to:

1. separate visible rationale, candidate generation, oracle opportunity, verifier selection, and user-visible correctness;
2. implement sequential, parallel, and search-based test-time strategies with complete cost accounting;
3. derive pass@k and majority-vote baselines and state exactly when their assumptions fail;
4. evaluate outcome and process verifiers for calibration, discrimination, shift, and exploitation;
5. design fixed and adaptive budget/stopping policies without assuming monotonic returns;
6. distinguish total executed work from critical-path latency and production capacity impact;
7. trace a real generation/stopping implementation at a pinned revision;
8. diagnose correlated errors, selection gaps, unfaithful rationales, verifier gaming, overthinking, and load-induced regressions.

## 03 Knowledge Map

```text
autoregressive generation
        |
        +--> visible chain of thought -----> faithfulness tests
        |
        +--> candidate set
               |-- parallel samples ------> vote / self-consistency
               |-- best-of-N -------------> verifier selection
               `-- tree expansion --------> value, prune, backtrack
                                      |
                                      v
                         outcome/process verifier
                                      |
                                      v
                           selected answer / abstain

budget controller: strategy + sequential limit + parallel width + stop policy
        |
        +--> total work: generator + verifier + tools + orchestration
        +--> critical path: wall-clock dependency chain
        `--> serving impact: queue, capacity, cancellations, SLO-goodput
```

Three measurement boundaries must remain separate:

- **Generator opportunity:** did the candidate set contain a correct answer?
- **Selector quality:** did the vote or verifier choose a correct candidate, conditional on the set?
- **System utility:** did the delivered answer meet application quality, cost, latency, capacity, and safety constraints?

---

### LAYER 1: FOUNDATIONS AND MECHANISMS

## 04 Lessons

### Lesson 6.1 — Chain-of-Thought Is an Output Protocol, Not a Causal Proof

**Engineering question:** What does an intermediate rationale demonstrate, and what does it not demonstrate?

Chain-of-thought prompting elicits intermediate text before a final answer. Wei et al. report improvements on the models and arithmetic, commonsense, and symbolic tasks they evaluated. That observation motivates a workload-specific experiment; it does not prove universal benefit, an emergent capability threshold that transfers unchanged, or faithfulness of the printed rationale.

A reasoning-trace contract records:

- model and prompt revision, exemplars, formatting, and decoding parameters;
- whether the trace is user-visible, hidden, summarized, or discarded;
- how the final answer is parsed and scored;
- token, latency, and cost boundaries;
- interventions used to test whether the rationale acknowledges factors that change the answer.

Turpin et al. and later work on reasoning models provide counterexamples to universal faithfulness: answer-changing cues can be omitted from the explanation. Such experiments falsify “the rationale always reports the cause”; they do not establish that every rationale is false or reveal the model's complete hidden computation.

**Break cases:** misleading few-shot exemplars, spurious hints, answer-format leakage, persuasive but invalid steps, rationale truncation, and a task where direct answering is already reliable.

**Learning outcome:** use visible reasoning as an instrumented output artifact, not privileged ground truth about internal causation.

---

### Lesson 6.2 — Parallel Sampling, Self-Consistency, and pass@k

**Engineering question:** Does generating more candidates improve opportunity, selection, or both?

Self-consistency samples diverse reasoning paths and aggregates their final answers. A production implementation must define answer normalization, invalid parses, abstentions, tie-breaking, duplicate handling, and whether it uses plurality, weighted voting, or a separate verifier.

For $k$ independent candidates with identical correctness probability $p$, oracle success is:

$$
P(\text{at least one correct})=1-(1-p)^k.
$$

This is **D**, an analytical baseline with IID assumptions. Shared prompts, one model, common misconceptions, and similar decoding can correlate failures, so it must not be used as an empirical forecast without testing dependence.

Given $n$ observed candidates containing $c$ correct, the finite-sample probability that a uniformly selected subset of size $k\le n$ contains at least one correct candidate is:

$$
\widehat{pass@k}=1-\frac{\binom{n-c}{k}}{\binom{n}{k}}.
$$

This is an oracle set metric, not selected accuracy. For odd $k$ IID binary votes, strict-majority correctness is:

$$
P(\text{majority correct})=
\sum_{j=(k+1)/2}^{k}\binom{k}{j}p^j(1-p)^{k-j}.
$$

Real self-consistency often uses multi-class plurality: wrong answers may split, one wrong answer may dominate through correlated error, and normalization can merge or fragment answers. Report at least single-sample accuracy, oracle pass@k, selected accuracy, parsing failure, answer entropy, duplicates, cost, and latency.

**Learning outcome:** determine whether extra sampling improves the candidate pool, the selector, or neither.

---

### Lesson 6.3 — Search Over Reasoning States

**Engineering question:** When is explicit branching and backtracking better than independent full answers?

Tree of Thoughts provides a reference mechanism: generate candidate thoughts, evaluate states, choose branches, look ahead, and backtrack. The design space includes breadth/depth, proposal count, state representation, value estimates, pruning, terminal tests, duplicate-state detection, and call scheduling.

Search helps only when three conditions approximately hold:

1. intermediate states capture choices that affect the solution;
2. proposal diversity reaches useful alternatives;
3. the evaluator discriminates promising from misleading states early enough to repay search cost.

The paper's results on Game of 24, Creative Writing, and Mini Crosswords establish scoped observations, not a general law. On tasks without a meaningful partial-state value, search may multiply calls, prune the correct branch, or optimize a proxy.

**Implementation record per node:** parent, depth, state text or structured state, generator configuration, score and scorer revision, expansion timestamp, token/cost ledger, prune reason, terminal result, and independent correctness if available.

**Learning outcome:** treat search as proposal plus value estimation plus resource control, not as a prompt slogan.

---

### Lesson 6.4 — Outcome, Process, and Selection Verifiers

**Engineering question:** What exactly is being verified, and against which truth?

- **Outcome verifier:** scores the final answer or product.
- **Process verifier/reward model:** scores intermediate steps or transitions.
- **Objective judge:** executes a test, checks a proof, or compares with ground truth under declared rules.
- **Learned or model-based judge:** estimates correctness or preference and can be wrong, shifted, or exploited.

Lightman et al. compare outcome and process supervision and report a process-supervision advantage in their MATH setup while releasing PRM800K. This supports studying step-level feedback; it does not establish universal verifier superiority or transfer.

Separate:

$$
\text{oracle set success} \geq \text{selected success}
$$

when both operate on the same candidate set and ground-truth correctness is well-defined. A verifier cannot select a correct item absent from that fixed set, and can fail despite one being present. If verifier-guided search changes the candidate set, the comparison no longer isolates selector quality.

Verifier validation includes score semantics, calibration, ranking/discrimination, false positives and negatives, abstention, subgroup performance, temporal and domain shift, adversarial candidates, and score-quality behavior as search pressure grows. A rising verifier score without rising independent correctness is evidence consistent with gaming, but evaluator noise and distribution shift remain competing explanations.

**Learning outcome:** build verification as a measured subsystem rather than equating its score with truth.

---

### Lesson 6.5 — Compute Ledgers, Budgets, and Stopping

**Engineering question:** How much work was executed, how long did the user wait, and was another unit of work worth doing?

Let every executed generator, verifier, tool, and orchestration action be converted to a declared unit:

$$
C_{total}=C_{generator}+C_{verifier}+C_{tools}+C_{orchestration}.
$$

Possible units include GPU-seconds on a pinned device, accelerator FLOPs with a stated estimator, tokens separated by model class, or monetary cost under a pinned price contract. Tokens from different models and sequence positions are not compute-equivalent. Total work is additive after valid conversion; wall-clock latency follows the dependency critical path under available parallel resources.

A fixed policy may cap new tokens, elapsed time, samples, branches, depth, or cost. An adaptive policy estimates whether marginal expected value justifies marginal cost. One useful decision model is:

$$
U(B)=Q(B)-\lambda C(B),
$$

where $B$ is a declared budget, $Q$ measured task utility, $C$ measured cost, and $\lambda$ a stakeholder trade-off in compatible units. Hard latency, safety, or subgroup constraints may not be reducible to this scalar.

Snell et al. provide evidence that effective allocation varies with prompt difficulty in their evaluated setting. The s1 paper demonstrates one sequential control—truncating thinking or appending “Wait” at an attempted stop—for its trained model and competition-math experiments. A 2026 preprint reports diminishing returns and cases of abandoning correct answers; treat this as frontier evidence to reproduce, not a universal overthinking law.

**Stopping policy requirements:** global and per-branch limits, clock definition, cancellation semantics, completed-work accounting, minimum answer reserve, fallback, timeout behavior, and post-stop parsing. Test both premature stopping and excess continuation.

**Learning outcome:** optimize measured utility across a full compute and critical-path ledger, not rationale length.

---

### Lesson 6.6 — Adaptive Policies and Causal Diagnosis

**Engineering question:** How do we decide which requests deserve which strategy without leaking answers or masking failures?

A router can choose direct response, bounded sequential reasoning, parallel sampling, or search. Difficulty is latent. A proxy derived from benchmark labels, future verifier outcomes, or target answers creates leakage; one calibrated on one model/domain may fail after a prompt, model, or population change.

Evaluate the policy at equal aggregate resource budget and report:

- routing features and their availability time;
- predicted difficulty or marginal value and calibration;
- allocation by task and subgroup;
- single-sample, oracle, selected, and user-visible utility;
- total work, critical-path latency, serving capacity, cancellations, and SLO-goodput;
- temporal and out-of-domain holdouts;
- a uniform-budget and direct-answer baseline.

Use the diagnostic loop:

$$
\text{symptom}\to\text{competing hypotheses}\to\text{missing evidence}
\to\text{discriminating measurement}\to\text{ranked explanation}
\to\text{intervention}\to\text{remeasurement}.
$$

Necessary but insufficient signals include longer rationales, higher verifier score, higher oracle pass@k, more answer diversity, and higher accelerator utilization. Each can coexist with worse selected correctness or production utility.

**Learning outcome:** deploy adaptive compute as a falsifiable control policy with quality and operational feedback.

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) — Wei et al. (2022). Read the prompting method and scoped experiments; do not infer faithfulness.
- [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://arxiv.org/abs/2203.11171) — Wang et al. (ICLR 2023). Read the sampling and answer-marginalization procedure.
- [Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601) — Yao et al. (NeurIPS 2023). Read proposal, evaluation, search, and task-specific configurations.
- [Let's Verify Step by Step](https://arxiv.org/abs/2305.20050) — Lightman et al. (2023). Read the outcome/process supervision comparison and PRM800K scope.
- [Evaluating Large Language Models Trained on Code](https://arxiv.org/abs/2107.03374) — Chen et al. (2021). Read the pass@k estimator and sampling protocol, not only headline benchmark values.

**CURRENT DEFAULT**

- Token, time, stop-string, EOS, and custom stopping controls are common generation primitives. They are resource/protocol stops, not semantic proof that sufficient reasoning occurred.
- Candidate-level telemetry and explicit answer parsing are baseline engineering requirements for any multi-sample policy. No particular self-consistency width or reasoning-token budget is a universal default.

**WORKLOAD-DEPENDENT**

- Chain-of-thought prompting, self-consistency, best-of-N, verifier ranking, and tree search.
- Outcome versus process verification.
- Sequential versus parallel compute and fixed versus adaptive budgets.

**FRONTIER**

- [Scaling LLM Test-Time Compute Optimally can be More Effective than Scaling Model Parameters](https://arxiv.org/abs/2408.03314) — Snell et al. (2024): workload-adaptive allocation in the evaluated setting.
- [s1: Simple test-time scaling](https://arxiv.org/abs/2501.19393) — Muennighoff et al. (2025): budget forcing tied to a trained model and math setup.
- [Reasoning Models Don't Always Say What They Think](https://arxiv.org/abs/2505.05410) — Chen et al. (2025): hint-faithfulness tests on reasoning models.
- [When More Thinking Hurts: Overthinking in LLM Test-Time Compute Scaling](https://arxiv.org/abs/2604.10739) — Zhou et al. (2026 preprint): evidence motivating non-monotonic budget tests; replication remains necessary.

**LEGACY / INSUFFICIENT WHEN USED ALONE**

- Greedy single-path decoding as the only reasoning baseline.
- Reporting oracle pass@k as if a production selector achieved it.
- Treating visible rationale plausibility or length as correctness, faithfulness, or compute.
- Counting only generator output tokens while omitting verifier, tool, orchestration, cancellation, and capacity costs.

**PRODUCTION SOURCE TRACE**

- Repository: `huggingface/transformers`
- Revision: `27166ea03f12c940f23176a904ab1d2ff1a3dcbb`
- Verified: 2026-09-26 by direct static inspection; not executed in this repository.
- Files: `src/transformers/generation/utils.py`, `src/transformers/generation/stopping_criteria.py`.
- Symbols: `GenerationMixin.generate`, `GenerationMixin._prepare_generated_length`, `GenerationMixin._get_stopping_criteria`, `MaxLengthCriteria.__call__`, and `StoppingCriteriaList.__call__`.
- Observed path: `generate` prepares length/configuration, creates stopping criteria, enters the selected decode loop, and evaluates the criteria. `max_new_tokens` determines total `max_length` from input length; the criteria list ORs per-row stop decisions.
- Generalization: this is a pinned implementation snapshot. It demonstrates generic generation controls, not a universal runtime design or a semantic reasoning controller.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

Every lab follows:

$$
\text{PREDICT}\to\text{BUILD}\to\text{MEASURE}\to\text{EXPLAIN}
\to\text{BREAK}\to\text{IMPROVE}\to\text{FALSIFY}.
$$

### LAB A — Sequential Budget Sweep and Faithfulness Probe

- **Objective:** compare direct answers and elicited reasoning across token/time budgets, then test whether rationales acknowledge controlled answer-changing cues.
- **Independent variables:** model/prompt revision, task stratum, direct versus chain-of-thought prompt, new-token cap, stop rule, temperature, and benign versus biasing intervention.
- **Measurements:** parsed correctness, rationale and answer tokens, TTFT/end-to-end latency, cost, finish reason, correctness transitions by prefix, intervention sensitivity, and acknowledgement rate.
- **Break/falsify:** include tasks where direct answering is strong, misleading exemplars, early truncation, forced continuation, repeated loops, and cues that change answers. Falsify monotone “more tokens means better reasoning” if a preregistered budget interval loses utility.
- **Required artifact:** raw generations, parser specification/tests, budget-quality-cost curves with uncertainty, intervention pairs, and a statement of what the faithfulness probe cannot establish.
- **Effort:** 4h.

### LAB B — Self-Consistency, pass@k, and Correlated Error

- **Objective:** implement repeated sampling, oracle pass@k, plurality/majority selection, and a dependence audit.
- **Independent variables:** $k$, temperature/top-p, prompt variant, answer normalizer, tie rule, and task difficulty stratum.
- **Measurements:** single-sample accuracy, finite-sample pass@k, selected accuracy, oracle-selection gap, duplicates, answer entropy, pairwise agreement/error correlation, invalid parses, total work, and critical-path latency at controlled parallelism.
- **Break/falsify:** construct or locate a stratum where candidates confidently repeat one wrong answer; perturb normalization to expose merge/split errors; compare empirical gains with the IID analytical baseline.
- **Required artifact:** derivations with assumptions, tested implementation, candidate table, dependence diagnostics, and a cost-normalized comparison against direct answering.
- **Effort:** 4h.

### LAB C — Verifier-Guided Search Under Adversarial Candidates

- **Objective:** build bounded best-of-N or tree search using an outcome or process verifier and separate candidate opportunity from selection.
- **Independent variables:** candidate count, search breadth/depth, verifier revision, threshold/ranking rule, pruning, domain, and adversarial perturbation strength.
- **Measurements:** oracle set success, selected success, verifier calibration and ranking, false positives/negatives, abstention, score-quality gap, branches expanded/pruned, independent correctness, total work, and critical path.
- **Break/falsify:** inject persuasive wrong solutions, invalid but high-scoring steps, paraphrases, out-of-domain items, and increasing search pressure. Reject the verifier-gaming explanation if independent utility and calibration remain stable under the preregistered stress set.
- **Required artifact:** node/candidate records, verifier card, held-out and adversarial results, source/revision manifest, failure taxonomy, and rollback threshold.
- **Effort:** 5h.

### LAB D — Adaptive Budget Controller Under Cost and SLO

- **Objective:** route requests among direct, sequential, parallel, and search strategies using features available before target outcomes are known.
- **Independent variables:** router features, budget levels, strategy set, load, latency deadline, and aggregate resource envelope.
- **Measurements:** difficulty/value calibration, routing confusion, allocated work, utility, subgroup effects, cost, critical-path latency, cancellation waste, serving queue/capacity signals, and SLO-goodput.
- **Break/falsify:** remove leakage-prone features, apply temporal/domain shift, introduce traffic bursts, and compare against equal-cost uniform policies. Falsify adaptive advantage if it disappears at equal aggregate cost or violates a protected constraint.
- **Required artifact:** controller and manifest, offline replay plus controlled loaded test, leakage audit, policy frontier, operational limits, and canary/rollback design.
- **Effort:** 5h.

## 07 Break / Incident Scenarios

### Incident 06.1 — Offline Reasoning Gain, Production Utility Loss

A release replaces direct decoding with an adaptive policy. It sends difficult-looking requests to parallel candidates and a process verifier, while allowing longer sequential continuation when confidence is low. Offline benchmark selected accuracy improves. In production, tail latency and cost rise, SLO-goodput falls, one user subgroup regresses, and verifier scores keep increasing with search depth even though sampled human audits do not.

The incident does not prescribe one root cause. The learner must:

1. **Form competing hypotheses:** traffic or task shift; correlated candidates; answer-normalization bugs; verifier miscalibration or gaming; difficulty-label leakage; continuation-induced answer reversal; cancellation waste; serving saturation; subgroup routing skew; or an unrelated deployment change.
2. **Identify missing evidence:** joined request/candidate/verifier/resource traces; direct, oracle, selected, and audited correctness; route allocation; score calibration; token/call ledger; queue and capacity metrics; cancellation status; release diff; subgroup and temporal slices.
3. **Discriminate:** replay a pinned population; compare uniform and adaptive policies at equal total cost; sweep width/depth/budget; independently judge selected and rejected candidates; disable features suspected of leakage; repeat under controlled load.
4. **Rank explanations:** use temporal ordering, counterfactual routing, factorial ablations, and effect uncertainty. Do not treat one correlation or aggregate accuracy as causal proof.
5. **Intervene:** gate or roll back the failed route, recalibrate/replace the verifier, repair parsing, cap or cancel work, reserve serving headroom, or change the objective—only as supported by evidence.
6. **Remeasure:** apply the same population, quality, subgroup, cost, latency, capacity, and SLO-goodput contract with preregistered promotion and rollback thresholds.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Transfer Problem — Defend a Test-Time Reasoning Policy

Design a production reasoning policy for a mixed workload containing objectively checkable problems, open-ended analytical tasks, and latency-sensitive requests. You may use direct decoding, bounded sequential reasoning, self-consistency, best-of-N, tree search, outcome verification, or process verification.

Deliver:

1. a pinned task population, model/runtime configuration, serving envelope, cost unit, quality criteria, and subgroup/SLO constraints;
2. a mechanism choice per workload class with evidence classification and explicit non-goals;
3. derivations for IID oracle pass@k, finite-sample pass@k, and a voting baseline, each with assumptions and failure cases;
4. a complete compute ledger separating total executed work from critical-path latency;
5. a candidate and verifier telemetry schema that reconstructs oracle opportunity, selection, pruning, cancellation, and final delivery;
6. an experiment that measures candidate dependence and oracle-selection gap;
7. a verifier validation suite covering calibration, shift, adversarial candidates, abstention, and independent checks;
8. a fixed or adaptive budget/stopping policy with leakage audit and non-monotonic budget tests;
9. a pinned production source trace for generation/stopping behavior and a clear generalizability statement;
10. a canary, capacity plan, rollback rule, and incident playbook that remeasures after intervention.

A benchmark accuracy increase alone does not pass. The defense must show where quality comes from, how the system selects it, what all executed work costs, and which observations would falsify the policy's causal story.

## 09 Required Evidence & Rubric

### Required Artifact: Reasoning-System Trace

For every request, preserve or safely summarize enough data to join:

- request/task stratum and prompt/model/runtime revision;
- route, budgets, stop conditions, and random seeds;
- every candidate/node, parent, parsed answer, finish/prune reason, and timestamps;
- verifier revision, input view, score, selected output, threshold, and abstention;
- independent correctness or audit status where available;
- generator/verifier/tool/orchestration work, cancellations, critical-path latency, and serving context.

Sensitive reasoning traces require an explicit retention and access policy; observability does not authorize unbounded storage of user data.

### Rubric Dimensions

- **Mechanistic reasoning:** separates elicitation, sampling, search, verification, selection, stopping, and delivery.
- **Mathematical discipline:** states IID, subset, majority, and fixed-candidate-set assumptions; does not convert oracle or mean metrics into deployed guarantees.
- **Measurement:** reports opportunity, selection, task utility, total work, critical path, and production impact at declared boundaries.
- **Verification rigor:** treats learned scores as estimates, validates calibration and shift, and uses independent/adversarial checks.
- **Failure diagnosis:** ranks competing causes with discriminating experiments and permits interactions and bottleneck transitions.
- **Operational defense:** includes privacy, cancellation, capacity, subgroup, canary, and rollback controls.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Reasoning traces and faithfulness | Lesson 6.1 | LAB A | Mastery 1, 8 | Intervention pairs and trace contract |
| Self-consistency and pass@k | Lesson 6.2 | LAB B | Mastery 3, 6 | Derivations and candidate-level dependence audit |
| Tree/search mechanics | Lesson 6.3 | LAB C | Mastery 2, 5 | Search graph, node records, pruning evidence |
| Outcome/process verification | Lesson 6.4 | LAB C | Incident / Mastery 6–7 | Verifier card, calibration, adversarial results |
| Compute budget and stopping | Lesson 6.5 | LABs A, D | Mastery 4, 8–10 | Cost ledger and budget-quality frontier |
| Adaptive policy diagnosis | Lesson 6.6 | LAB D | Incident / Mastery 8–10 | Leakage audit, equal-cost replay, canary plan |

## 11 Exit Criteria & Module Wrap-Up

A learner passes when they can:

1. explain why a visible rationale is neither automatic proof of correctness nor faithful causal explanation;
2. implement self-consistency and distinguish oracle pass@k from selected and user-visible accuracy;
3. derive the relevant probability baselines with assumptions and show an empirical dependence counterexample;
4. build bounded search with inspectable proposal, evaluation, pruning, and cost records;
5. validate a verifier beyond its own score and detect a widening score-quality gap;
6. account for generator, verifier, tool, orchestration, cancellation, total-work, and critical-path costs;
7. design stopping and adaptive budgets that test premature stopping and overthinking;
8. trace a pinned runtime implementation without treating it as the definition of reasoning control;
9. diagnose and remediate an offline-to-production regression, then remeasure at the same boundaries.

**Final mental model:** test-time reasoning is controlled generation plus selection under resource constraints. Extra work creates opportunities, not guaranteed value. The engineering task is to expose where opportunity becomes quality, measure what selection loses, charge every executed branch, and stop only when evidence supports the utility trade.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

# Module 16 — Falsification Engineering

## 00 Why This Module Exists

Evaluation estimates how a system performs on a declared population. Falsification engineering asks a different question: **what controlled search could prove our current requirement, oracle, or causal story wrong?** A passing suite is bounded negative evidence; a valid minimized counterexample is a concrete defect and a new regression asset.

```text
requirement / hypothesis
          |
 valid domain + oracle + severity
          |
 examples | properties | metamorphic | differential | mutation | faults
          |
 search + stochastic repetitions + complete lineage
          |
 candidate failure
          |
 validate -> adjudicate -> minimize -> deduplicate -> attribute
          |
 fix / accept risk -> promote regression -> remeasure escapes
```

Module 15 owns evaluation programs, estimates, graders, and release decisions. This module owns systematic counterexample search and test adequacy. Module 18 owns the security threat model and controls; Module 23 owns production reliability operations. Security red teams and chaos experiments use this module's methods but keep their domain policy in those modules.

**Research cutoff:** 2026-09-26.

**Module Orientation**
- **Engineering Problem**: Search systematically for counterexamples without confusing test volume, coverage, agreement, or suite survival with correctness.
- **What You Will Do**: Specify falsification contracts, build generators/state machines/shrinkers, validate metamorphic relations, run differential and mutation tests, inject bounded faults, trace Hypothesis, and operate a counterexample lifecycle.
- **Environment**: Python 3.10+, Hypothesis or an equivalent property-testing engine, isolated model/tool mocks, reproducible seeds, a fault injector, and disposable sandboxes for effectful tests.
- **Evidence Rule**: Keep source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**) separate.

## 01 Baseline Assumptions

- Module 00: claims, assumptions, measurement validity, and uncertainty.
- Modules 12–14: state transitions, attempts, effects, recovery, and fault boundaries.
- Module 15: evaluation contracts, experimental units, grader validity, and release gates.

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
  economics: SELECTIVE
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

The learner must specify properties and oracles; build valid generators and shrinkers; test stateful protocols; justify metamorphic relations; use differential and mutation testing without oracle illusions; quantify stochastic evidence; separate adaptive discovery from estimation; inject bounded faults; operate a counterexample lifecycle; and trace a pinned property-testing engine.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

A falsification contract records:

- requirement/hypothesis and owner;
- system boundary, valid domain, preconditions, and severity;
- predicate or relation and independent adjudication path;
- generator/search/fault policy, seed, budget, and stopping rule;
- stochastic repetition and denominator policy;
- system, model, prompt, tool, data, environment, and oracle versions;
- what failure means and what survival cannot prove.

Coverage is a search signal. Disagreement is a triage signal. Mutation score is suite-sensitivity evidence. None is correctness.

## 04 Lessons

### Lesson 16.1 — From Requirement to Falsifiable Contract

**Engineering Question:**
How can a vague robustness claim become a bounded predicate whose failure and non-claims are explicit?

**Concepts & Definitions:**
- **Valid domain**: inputs/states for which the requirement is intended.
- **Oracle**: procedure that adjudicates the property.
- **Non-claim**: behavior the search budget or generator cannot establish.

**Mechanism Explanation:**

Turn “the agent is robust” into observable contracts: schema is always valid; unauthorized effects never occur; adding irrelevant evidence should not change a cited answer; retry preserves one logical effect; cancellation prevents later effects; a locale transformation preserves a task-specific relation. Mark preconditions and acceptable nondeterminism.

CheckList's minimum-functionality, invariance, and directional-expectation tests are a useful behavioral vocabulary. Build a capability × test-type × risk-slice matrix, but do not treat filled cells as proof of completeness.

For generator support $S_G$ and valid domain $D$, a property test can falsify $P$ only on sampled $x\in S_G\cap D$. Passing says nothing about $D\setminus S_G$. Structural coverage records what was exercised, not whether the oracle checked it correctly.

**Quantitative Model / Derivation:**
For generator support $S_G$ and valid domain $D$, survival covers only sampled points in $S_G\cap D$; it provides no direct evidence about $D\setminus S_G$.

**Worked Example:**
Replace “the agent is robust” with: for authorized transfer intents in a declared schema/domain, duplicate delivery with one logical effect ID never creates more than one receiver-side effect. Record generator, oracle, budget, severity, and the non-claim about unseen providers.

**Knowledge Check:**
1. Why is a filled capability matrix not completeness proof?
2. Which precondition makes an authorization property meaningful?

**Guided Practice:**
Write a falsification contract for one invariant and list three inputs outside generator support.

**Feedback Contract:**
- *Expected Evidence*: Requirement, domain/preconditions, oracle, generator/search budget, stopping rule, severity, and non-claims.
- *Common Failure*: Using case count or coverage as correctness.
- *Diagnostic Hint*: What exact observation falsifies the predicate?
- *Concept to Revisit*: Bounded Negative Evidence.

**Learning Outcome:**
State a predicate precise enough to fail and humble enough not to overclaim.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 16.2 — Property-Based and Stateful Testing

**Engineering Question:**
How do generators, state machines, and shrinkers find small valid failures without changing their causal identity?

**Concepts & Definitions:**
- **Generator**: constructive sampler over structured valid inputs/actions.
- **Stateful model**: commands, preconditions, transitions, invariants, and postconditions.
- **Shrinker**: ordered reduction process that must preserve validity and failure signature.

**Mechanism Explanation:**

QuickCheck established the generator-plus-property pattern. In AI systems, generate structured prompts, schemas, tool responses, event streams, model outcomes, and action sequences—not arbitrary invalid bytes unless parser rejection is the property.

Measure generator support proxies: category/slice frequencies, boundary-value reach, filter/rejection rate, state/action transitions, and failure yield. Use constructive generators instead of filtering when possible. A shrinker must preserve domain validity and the same failure signature; “smallest” is relative to its representation and shrink order.

Stateful tests model commands, preconditions, transitions, invariants, and postconditions. Vary retries, cancellation, duplicate/out-of-order events, stale versions, timeout boundaries, and recovery. Record the whole action/observation/effect trace.

**Production source trace:** at Hypothesis revision `9c55f97e507eae21677457e84737b386fd06e271`, `given`/`find` and `RuleBasedStateMachine`/`run_state_machine_as_test` feed generated tests; `ConjectureRunner` drives exploration and `Shrinker` minimizes interesting examples. Static source inspection only.

**Quantitative Model / Trade-off Comparison:**
Report slice/boundary/transition reach, rejection rate, unique failure clusters, shrink steps/ratio, replay rate, and execution cost. None alone establishes oracle adequacy.

**Worked Example:**
A timeout-after-commit trace fails duplicate-effect safety. Removing the timeout makes the trace smaller but changes the failure; removing an unrelated read while retaining timeout, retry, and duplicate effect is a valid shrink.

**Knowledge Check:**
1. When is constructive generation preferable to filtering?
2. Why is the smallest serialized input not always the smallest causal trace?

**Guided Practice:**
Generate action sequences with cancellation, retries, stale versions, and unknown effects; implement a signature-preserving shrinker.

**Feedback Contract:**
- *Expected Evidence*: Support proxies, preconditions, full trace, seed, shrink trace, failure signature, and replay.
- *Common Failure*: Shrinking out the boundary that caused the defect.
- *Diagnostic Hint*: Does the minimized case fail for the same adjudicated reason?
- *Concept to Revisit*: Valid Shrinking.

**Learning Outcome:**
Produce a valid, small, replayable counterexample rather than an untriageable random transcript.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 16.3 — Behavioral and Metamorphic Relations

**Engineering Question:**
When is a transformation semantics-preserving enough that output inconsistency is evidence of a defect?

**Concepts & Definitions:**
- **Input relation** $R_i$: declared relation between source and follow-up inputs.
- **Output relation** $R_o$: invariant or directional expectation on outcomes.
- **Transformation validity**: independent check that both inputs remain in-domain.

**Mechanism Explanation:**

Metamorphic testing replaces a missing single-input oracle with two obligations:

$$R_i(x,x')\Rightarrow R_o(f(x),f(x')).$$

Both $x,x'$ must remain in-domain; $R_i$ must preserve or deliberately change the relevant semantics; $R_o$ must encode the expected invariant or direction. Examples include permutation invariance only when order is irrelevant, citation preservation under irrelevant distractors, monotonic recall under an enlarged eligible corpus, and equivariance under a semantics-preserving locale transform.

The 1998 technical report is the reference mechanism. A 2025 LLM study catalogs many NLP relations and manually checks reported violations, illustrating both usefulness and oracle risk. Never assume synonym replacement, paraphrase, formatting, answer-order swap, or added context is semantically neutral for every task.

For free-form outputs, separate transformation validity from output-relation grading. Use executable/domain checks where possible and audit any semantic judge independently.

**Quantitative Model / Derivation:**
A valid test checks $R_i(x,x')\Rightarrow R_o(f(x),f(x'))$. A failure of $R_o$ is actionable only after validating $R_i$ and the output oracle.

**Worked Example:**
Valid candidate: reorder a set of independent evidence records when the task contract declares order irrelevant. Invalid candidate: swap answer options in a task where labels refer to position. Test transformation validity separately from result consistency.

**Knowledge Check:**
1. Why is paraphrase not universally semantics-preserving?
2. Can an LLM judge validate both transformation and output relation without circularity?

**Guided Practice:**
Define five relations with preconditions, including one deliberately invalid transformation, and adjudicate both stages independently.

**Feedback Contract:**
- *Expected Evidence*: Relation card, domain proof, independent output oracle, stochastic repetitions, and false-positive taxonomy.
- *Common Failure*: Treating any changed answer as model failure.
- *Diagnostic Hint*: Did the transformation preserve the relevant semantics?
- *Concept to Revisit*: Metamorphic Oracle.

**Learning Outcome:**
Detect inconsistent behavior without inventing false invariants.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 16.4 — Differential and Mutation Testing

**Engineering Question:**
How can disagreement and controlled mutants expose blind spots without pretending either one identifies ground truth automatically?

**Concepts & Definitions:**
- **Differential candidate**: shared input with divergent outcomes across comparable systems.
- **Mutation operator**: controlled injected fault representing a declared defect class.
- **Equivalent/invalid mutant**: mutation that does not change required behavior or violates the test domain.

**Mechanism Explanation:**

Differential testing runs comparable implementations or revisions on the same input. DeepXplore demonstrates this family for neural systems. A disagreement locates a candidate boundary; adjudication decides whether A, B, both, or neither satisfy the contract. Independence matters: related models, shared retrieval, shared prompts, or the same judge can fail together.

Mutation testing injects controlled faults into code, configuration, prompt, policy, data, tool response, or workflow guard. With generated mutants $M$, killed mutants $K$, and adjudicated equivalent/invalid mutants $E$:

$$MS=\frac{K}{M-E}.$$

Report operator distribution, equivalence review, subsumption, execution cost, and which test killed which mutant. Mutation score measures sensitivity to selected fault classes—not production defect prevalence or correctness. Include realistic mutants such as removed authorization, wrong timeout unit, missing citation binding, stale version, dropped terminal event, retry after unknown commit, and changed system prompt.

**Quantitative Model / Derivation:**
With 20 generated mutants, 12 killed, 3 adjudicated equivalent, and 1 invalid, $MS=12/(20-3-1)=0.75$. This describes sensitivity to those operators, not production defect prevalence.

**Worked Example:**
Two backends agree because both share a faulty mock, so differential testing misses the defect. A mutant that decouples external commit from acknowledgement survives, revealing the common-mode oracle gap.

**Knowledge Check:**
1. Why does disagreement require independent adjudication?
2. How do equivalent mutants distort a naive score?

**Guided Practice:**
Build a kill matrix across code, configuration, prompt, policy, retrieval, event-order, retry, and stopping mutants.

**Feedback Contract:**
- *Expected Evidence*: Operator distribution, equivalence review, adjudicated disagreement, common dependencies, kill matrix, and cost.
- *Common Failure*: Declaring one disagreeing implementation wrong by identity.
- *Diagnostic Hint*: Which oracle or dependency do all targets share?
- *Concept to Revisit*: Suite Sensitivity.

**Learning Outcome:**
Expose shared assumptions and measure whether the suite notices controlled breakage.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 16.5 — Stochastic, Adaptive, and Fault-Injection Evidence

**Engineering Question:**
What probability claim is justified after stochastic search, adaptive discovery, or controlled fault injection?

**Concepts & Definitions:**
- **Trial unit**: sample, request, trajectory, user/session, or environment seed.
- **Adaptive discovery**: search distribution changes in response to prior results.
- **Confirmatory sample**: protected evidence not selected by the discovery process.

**Mechanism Explanation:**

Define the trial unit: model sample, full request, trajectory, user/session, or environment seed. Preserve failed attempts. Under IID Bernoulli trials with fixed failure predicate and zero observed failures, the exact one-sided upper bound at confidence $1-\alpha$ is

$$p_U=1-\alpha^{1/n}.$$

This does not cover unseen prompts and fails under correlated retries, shifting systems, adaptive search, or hidden exclusions. Repetitions estimate conditional stochastic behavior; broader generators estimate input variation.

Fuzzers and red teams adapt toward weakness. Their finds are discovery evidence, not unbiased prevalence samples. Freeze promoted regressions and use a separate protected confirmatory sample or an analysis that models adaptive selection.

Fault injection names target boundary, fault, timing, expected invariant, blast radius, abort/cleanup, and recovery oracle. Inject timeout, malformed/partial response, cancellation race, quota, stale cache, duplicate event, tool failure, worker crash, or lost acknowledgement at controlled boundaries. Synthetic faults may miss correlated provider or regional incidents.

**Quantitative Model / Derivation:**
Under fixed IID Bernoulli trials with zero failures, $p_U=1-\alpha^{1/n}$. With $n=100$ and $\alpha=0.05$, this is approximately 0.0295. The bound is invalid for correlated retries, adaptive selection, shifting systems, or hidden exclusions.

**Worked Example:**
Compare 100 independent fixed-prompt draws with 100 retries from ten prompts chosen adaptively after near failures. Only the first matches the stated IID model; neither covers unseen prompt domains automatically.

**Knowledge Check:**
1. Why do repeated generations estimate conditional rather than broad input variation?
2. Which safety controls must accompany fault injection?

**Guided Practice:**
Separate adaptive discovery, protected confirmation, and workload estimation; specify fault boundary, blast radius, abort, cleanup, and recovery oracle.

**Feedback Contract:**
- *Expected Evidence*: Trial unit, independence assumptions, denominator, selection history, system version, fault timing, and cleanup.
- *Common Failure*: Reporting red-team finds as prevalence or applying an IID bound to adaptive trials.
- *Diagnostic Hint*: Did later trials depend on earlier outcomes?
- *Concept to Revisit*: Discovery vs. Estimation.

**Learning Outcome:**
Quantify evidence without converting search success into population rates.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 16.6 — Counterexample Lifecycle and Portfolio Economics

**Engineering Question:**
How should discoveries become durable regression evidence, and which mix of methods is worth its operational cost?

**Concepts & Definitions:**
- **Failure cluster**: cases grouped by adjudicated causal signature.
- **Promotion**: versioned minimized counterexample enters regression protection.
- **Portfolio evidence**: unique consequential yield, overlap, false positives, cost, latency, and escapes.

**Mechanism Explanation:**

Every candidate moves through `VALIDATE → ADJUDICATE → MINIMIZE → DEDUPLICATE → ATTRIBUTE → FIX/ACCEPT → PROMOTE → RETIRE/REFRESH`. Preserve original and minimized cases, transformation/shrink trace, system/oracle versions, raw attempts, effect evidence, root cause, owner, severity, and replay status.

Cluster by causal signature, not surface text alone. A flaky failure remains a stochastic test with a measured reproduction rate; it is not silently deleted. Review obsolete fixtures when product requirements change.

Measure unique adjudicated failure clusters, severity, overlap by method, time/compute/judge/triage cost, false positives, shrink ratio, replay rate, suite latency, false blocks, and escaped incidents. The claim that a risk-weighted portfolio beats one coverage objective is **H**; test it against a preregistered baseline.

**Quantitative Model / Trade-off Comparison:**
For each method report unique adjudicated clusters and severity alongside compute/judge/human/triage cost, overlap, suite latency, false blocks, replay rate, and escaped incidents. A risk-weighted portfolio claim remains a workload hypothesis.

**Worked Example:**
Method A finds 20 cases collapsing to two low-severity clusters; Method B finds four cases across three clusters including one critical defect but costs twice as much. Case count alone ranks A, while risk-weighted unique yield may justify B.

**Knowledge Check:**
1. Why should flaky failures be measured rather than silently deleted?
2. When should a promoted case be retired or refreshed?

**Guided Practice:**
Run `validate → adjudicate → minimize → deduplicate → attribute → fix/accept → promote`, then compare a risk-weighted portfolio with one coverage-maximizing baseline.

**Feedback Contract:**
- *Expected Evidence*: Original/minimized cases, lineage, causal cluster, owner/severity, replay rate, cost, overlap, and escape tracking.
- *Common Failure*: Accumulating unreproducible fixtures with no owner.
- *Diagnostic Hint*: Which unique risk does this test continue to cover?
- *Concept to Revisit*: Counterexample Lifecycle.

**Learning Outcome:**
Turn discoveries into durable evidence without building a counterexample cemetery.

*(Effort: 35m instruction, 25m practice)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [QuickCheck](https://doi.org/10.1145/351240.351266) — Claessen and Hughes, 2000; properties and generators.
- [Metamorphic Testing](https://www.cse.ust.hk/faculty/scc/publ/CS98-01-metamorphictesting.pdf) — Chen, Cheung, and Yiu, 1998; source/follow-up relations.
- [DeepXplore](https://arxiv.org/abs/1705.06640) — Pei et al., 2017; differential testing and coverage-guided search.
- [CheckList](https://aclanthology.org/2020.acl-main.442/) — Ribeiro et al., ACL 2020; capability/test-type matrix.
- [Mutation Testing Survey](https://doi.org/10.1109/TSE.2010.62) — Jia and Harman, 2011.

**CURRENT DEFAULT:** explicit falsification contracts; valid structured generation; stateful invariants; domain-preserving shrink; independent adjudication; discovery/confirmation separation; complete attempt lineage; minimized versioned regression cases; cost and escape accounting.

**WORKLOAD-DEPENDENT:** generator distributions, metamorphic relations, oracle portfolio, repetitions, mutants, fault model, severity, shrink order, coverage signals, and red-team budget.

**FRONTIER:** [Metamorphic Testing of Large Language Models for NLP](https://arxiv.org/abs/2511.02108) (Cho, Ruberto, Terragni, 2025) expands relation evidence; its empirical results and relations require task-specific validation.

**LEGACY / INSUFFICIENT:** random prompts without a domain model; “no failures means safe”; agreement as truth; disagreement as proof one named system is wrong; coverage as correctness; mutation score without equivalent-mutant policy; red-team findings as prevalence; shrinking that changes the failure; chaos without blast-radius controls.

**PRODUCTION SOURCE TRACE**

- Repository: `HypothesisWorks/hypothesis`
- Revision: `9c55f97e507eae21677457e84737b386fd06e271`
- Verified: 2026-09-26; static inspection only.
- Files/symbols: `hypothesis/src/hypothesis/core.py::{given,find}`, `stateful.py::{RuleBasedStateMachine,run_state_machine_as_test}`, `internal/conjecture/engine.py::ConjectureRunner`, and `internal/conjecture/shrinker.py::Shrinker`.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Generators, State Machines, and Shrinking

- **Objective**: Build structured stateful generators and validity-preserving shrinkers for tool-loop invariants.
- **Pre-Registered Hypothesis**: Constructive generation plus signature-aware shrinking will improve valid boundary reach and replayable minimization over filter-heavy generation on the declared domain.
- **Independent Variables**: Generator design, action topology, fault boundary, shrink order, and nondeterminism.
- **Dependent Variables**: Support/transition reach, rejection rate, unique failures, shrink validity/ratio, replay, and cost.

- Specify schema, authorization, cancellation, retry, and effect invariants for a tool-using loop.
- Implement structured generators and a rule-based state machine; pin/replay seeds and trace the Hypothesis execution path.
- Break with invalid overgeneration, heavy filtering, rare transitions, correlated choices, nondeterminism, and a shrinker that changes failure identity.
- Artifact: support/transition report, smallest valid counterexample, shrink trace, and explicit non-claims.
- **Break & Falsify**: Seed a failure that a naive shrinker erases; if the final case changes causal signature, minimization fails.
- **Alignment**: Lessons 16.1–16.2.
- **Effort Estimate**: 3h total.

### LAB B — Metamorphic Oracle Audit

- **Objective**: Validate task-specific input/output relations and measure false positives.
- **Pre-Registered Hypothesis**: Separating transformation validity from output adjudication will reject at least one plausible but invalid relation in the declared set.
- **Independent Variables**: Relation, task slice, transformation, model/judge version, and stochastic repeat.
- **Dependent Variables**: Transformation validity, violations, false positives, stability, latency, and cost.

- Define at least five task-specific relations with preconditions and expected output relations.
- Include invariance, directional, equivariant, and deliberately invalid transformations.
- Validate transformations and output judgments independently; test prompt/model/judge versions and stochastic repetitions.
- Artifact: relation card, false-positive taxonomy, violation examples, and surviving-risk analysis.
- **Break & Falsify**: Include deliberately invalid transformations; acceptance of one as a product defect falsifies oracle discipline.
- **Alignment**: Lesson 16.3.
- **Effort Estimate**: 3h total.

### LAB C — Differential and Mutation Adequacy

- **Objective**: Adjudicate cross-system disagreement and measure sensitivity to realistic mutants.
- **Pre-Registered Hypothesis**: Mutation will expose at least one common-mode blind spot missed by related differential targets on the seeded fault set.
- **Independent Variables**: Model/backend/harness target, shared dependency, mutation operator, and oracle.
- **Dependent Variables**: Adjudicated disagreement, killed/equivalent/invalid/subsumed mutants, common-mode misses, cost, and latency.

- Cross two model versions, two backends, and two harness revisions on identical inputs; independently adjudicate disagreements.
- Mutate authorization, timeout, prompt, schema, retrieval evidence, event order, retry, and stopping logic.
- Measure killed/equivalent/invalid/subsumed mutants, common-mode misses, method overlap, cost, and latency.
- Artifact: differential matrix, mutation operator registry, kill matrix, and gaps ranked by hazard.
- **Break & Falsify**: Mutate authorization, timeout, prompt, schema, retrieval, event order, retry, and stop logic; a high score with critical surviving mutant falsifies aggregate adequacy.
- **Alignment**: Lesson 16.4.
- **Effort Estimate**: 3h total.

### LAB D — Stochastic Search and Fault Campaign

- **Objective**: Separate adaptive discovery, protected confirmation, workload estimation, and bounded fault injection.
- **Pre-Registered Hypothesis**: A risk-weighted portfolio will improve unique consequential yield per declared cost over one coverage-maximizing baseline on the seeded campaign.
- **Independent Variables**: Corpus role, search policy, trial unit, fault/timing, method portfolio, and budget.
- **Dependent Variables**: Conditional failure estimates, confirmed clusters, overlap, false positives, cost, latency, recovery, and escapes.

- Separate adaptive discovery corpus from a protected confirmatory sample and a workload sample.
- Inject boundary-specific transport, tool, state, and worker faults with abort and cleanup controls.
- Estimate conditional failure probabilities only where assumptions hold; preserve all attempts and effects.
- Run the full counterexample lifecycle and compare a risk-weighted portfolio against one coverage-maximizing baseline.
- Artifact: campaign manifest, fault matrix, counterexample ledger, cost/yield/escape report, and TODO_VERIFY list.
- **Break & Falsify**: Inject correlated/adaptive trials and unsafe cleanup assumptions; misuse of IID bounds or failed containment invalidates the campaign.
- **Alignment**: Lessons 16.5–16.6 and Incident 16.1.
- **Effort Estimate**: 3h total.

## 07 Break / Incident Scenarios

### Incident 16.1 — Ten Million Passing Tests, One Duplicate Payment

- **Incident Symptoms**: A suite reports enormous case count and high transition coverage, yet production duplicates a payment after timeout; generation, mocks, shrinking, differential targets, and mutants all excluded the critical acknowledgement window.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: Invalid property, unreachable branch, correlated choices, mock mismatch, missing fault boundary, response-only oracle, retry drift, stale fixture, or telemetry loss.
  2. *Rank Initial Plausibility*: Treat test volume/coverage as search signals, not proof against the observed effect.
  3. *Identify Missing Evidence*: Recover generator distributions/rejections, seeds/shrink trace, state/action reach, contracts, attempt/effect IDs, fault timing, mutation matrix, revisions, and external postconditions.
  4. *Design Discriminating Tests*: Reproduce with a receiver-side oracle, acknowledgement-loss fault, and realistic mutant; state outcomes that falsify each leading hypothesis.
  5. *Execute Causal Diagnosis*: Rank the earliest invalid domain, oracle, mock, shrink, or fault-model boundary.
  6. *Prescribe Mitigation and Prevention*: Fix the boundary, promote the valid minimized trace, diversify common dependencies, and add ownership/retirement policy.
  7. *Remeasure*: Reproduction, shrink validity, mutant kill, unique failure yield, false positives, cost, and escaped incidents.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Falsification Program for an Effectful AI System

Build a falsification program for a stochastic tool-using system with governed external effects and incomplete oracles.

**Required Deliverables**:
1. Falsification contracts, valid domains, oracles, severity, budgets, and non-claims.
2. Property/stateful generators, support evidence, and valid shrinking.
3. Behavioral/metamorphic relation cards and independent adjudication.
4. Differential matrix and realistic mutation operator/kill registry.
5. IID-qualified stochastic analysis and discovery/confirmation separation.
6. Safe fault campaign and versioned counterexample ledger.
7. Pinned Hypothesis trace plus portfolio cost/yield/escape evidence.
8. Diagnosis and promoted regression for Incident 16.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace

Submit a pinned trace from generated tests/state machines through exploration and shrinking. State which engine paths were statically inspected and which stochastic/runtime behaviors were not executed.

### Rubric Dimensions

- **Contract and Generation**: *Insufficient* sends random prompts. *Competent* defines domain/property/oracle and measures support. *Strong* produces valid replayable minimized counterexamples with explicit non-claims.
- **Oracles and Adequacy**: *Insufficient* treats disagreement/coverage as truth. *Competent* validates relations and reviews mutants. *Strong* exposes common-mode and equivalent-mutant blind spots through independent adjudication.
- **Statistics and Fault Safety**: *Insufficient* reports prevalence from adaptive cases or injects uncontrolled faults. *Competent* states trial assumptions and containment. *Strong* separates discovery/confirmation and proves abort, cleanup, and postconditions.
- **Lifecycle and Economics**: *Insufficient* accumulates cases. *Competent* owns, clusters, promotes, and reviews them. *Strong* optimizes unique consequential yield, overlap, latency, triage cost, false blocks, and escapes.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Falsification contract and oracle | 16.1 | LAB A–B | Incident / Mastery | Contract and non-claims |
| Property/stateful generation and shrinking | 16.2 | LAB A | Incident / Mastery | Support, trace, counterexample |
| Metamorphic/differential/mutation testing | 16.3–16.4 | LAB B–C | Mastery | Relation, adjudication, kill matrix |
| Stochastic/adaptive/fault evidence | 16.5 | LAB D | Incident / Mastery | Bounds, split, fault matrix |
| Counterexample lifecycle and portfolio | 16.6 | LAB D | Mastery | Ledger, yield/cost/escape report |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner successfully completing Module 16 must be able to:
1. Falsify a meaningful system claim with a valid minimized case.
2. Defend the domain, oracle, transformation, and non-claims.
3. Measure generator reach, shrink validity, and stochastic assumptions.
4. Expose differential common-mode and mutation blind spots.
5. Inject one bounded fault with abort, cleanup, and recovery evidence.
6. Separate adaptive discovery from confirmatory estimation.
7. Promote and replay counterexamples while measuring portfolio cost and escapes.

### Module Wrap-Up (Final Mental Model Reconstruction)

- **The Core Invariant**: Falsification engineering is disciplined search for disconfirming evidence.
- **The Search Path**: claim/domain/oracle → generated transformations/faults → candidate failure → validate/adjudicate/minimize → fix or accept risk → regression.
- Its output is not a giant test count but a trustworthy counterexample—or a precisely bounded statement about where none was found.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

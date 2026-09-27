# Module 17 — Harness Evolution

## 00 Why This Module Exists

A model harness that passed every test in Module 13 still changes. Providers retire snapshots and deprecate request parameters, aliases move to new weights, prompts are edited, optimizers generate new instructions, and tool or output schemas evolve. Each change can alter behavior without any application code change. Harness evolution is the discipline of changing these components deliberately: know exactly what changed, measure which behaviors regressed, and ship through controlled exposure with a rollback that actually restores a valid state.

```text
harness manifest v_n
   |  change (forced: retirement / parameter deprecation | elective: model, prompt, optimizer, schema)
   v
compatibility check -> paired offline evaluation -> (optional) re-optimization + sealed test
   |
shadow traffic -> randomized canary vs concurrent control -> staged ramp -> full rollout
   |                                   |
 migration ledger <------------- guardrail breach -> state-aware rollback
```

Module 13 owns the single-version model I/O boundary. Module 15 owns general evaluation programs and release gates; Module 16 owns counterexample search. This module owns **version-to-version change**: manifests, migration evaluation, prompt optimization governance, and progressive rollout. Economics of routing belongs to Module 22 and production SLO operations to Module 23.

**Research cutoff:** 2026-09-27.

**Module Orientation**
- **Engineering Problem**: Change models, prompts, and harness components on your schedule—or a provider's—without shipping hidden regressions or breaking persisted state.
- **What You Will Do**: Build a harness manifest and change classifier, run paired migration evaluations, run and govern an automated prompt optimizer, quantify selection bias, size a canary, design rollback, trace DSPy optimizer source, and diagnose a forced-migration incident.
- **Environment**: Python 3.10+, access to at least two model versions (hosted or local), a frozen evaluation set with a validated grader, DSPy or an equivalent optimizer, and a traffic-replay or simulated canary harness.
- **Evidence Rule**: Keep source observations (**O**), explicit derivations (**D**), and telemetry-dependent hypotheses (**H**) separate.

## 01 Baseline Assumptions

- Module 13: canonical versus effective requests, provider adapters, capability negotiation, validation ladders, and replay manifests.
- Module 15: evaluation contracts, experimental units, grader validity, paired inference, and release gates.
- Module 16: falsification contracts and discovery-versus-confirmation separation.
- Module 14: durable state and why routing changes do not undo persisted effects.

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
  research_connection: SELECTIVE

estimated_effort:
  instruction: 4h
  guided_practice: 3h
  labs: 12h
  assessment: 3h
  source_trace: 2h
  total: 24h
```

The learner must pin every behavior-bearing component, classify changes, evaluate migrations with paired flips, govern automated prompt optimization against selection bias, size and run a canary against a concurrent control, and plan a rollback that accounts for persisted state.

---

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

Harness Manifest $\to$ Change Classification (forced / elective) $\to$ Compatibility Check $\to$ Paired Offline Evaluation (flips, slices) $\to$ Optional Re-optimization (train / validation / sealed test) $\to$ Shadow $\to$ Canary vs. Control $\to$ Staged Ramp $\to$ Full Rollout or State-Aware Rollback $\to$ Migration Ledger. Every arrow is a decision with evidence; no single aggregate score authorizes a migration.

## 04 Lessons

### Lesson 17.1 — The Harness Manifest and the Change Surface

**Engineering Question:**
Which components must be versioned so that any production response can be tied to one reproducible configuration, and which changes arrive without our consent?

**Concepts & Definitions:**
- **Harness manifest**: pinned model identifier or snapshot, provider/runtime and region, request parameters, system and task prompts, few-shot demonstrations, tool and output schemas, parser/validator versions, retry/fallback policy, and routing rules.
- **Change surface**: every manifest field whose change can alter the effective request, the model receiving it, or the interpretation of its output (**D**, CLM-001).
- **Forced change**: imposed by a dependency—e.g., under Anthropic's documented lifecycle, deprecated models get a retirement date and replacement, and requests to retired models fail (**O**, CLM-002).
- **Elective change**: chosen by the team—new model, prompt edit, optimizer output, schema change.
- **Alias versus snapshot**: an alias can move to new weights; a dated snapshot identifier is stable until its retirement. Provider-side serving changes behind a stable identifier may be visible only through behavior.

**Mechanism Explanation:**
A manifest hash is attached to every request log. A change classifier diffs two manifests and routes the change: parameter or schema changes first go through **request-validity checks** per target; model changes go through capability negotiation (Module 13) and then behavior evaluation. Forced changes carry a deadline, so the ledger tracks lifecycle dates as dependencies.

A concrete failure class: Anthropic documents that `temperature`, `top_p`, and `top_k` return a 400 error when set to non-default values on Claude Opus 4.7 and later, and the Python SDK v1.0+ removes those parameters (**O**, CLM-003). A harness that sets `temperature=0.2` works on the old model and is rejected on the replacement—before any quality question arises.

**Quantitative Model / Derivation:**
If a harness has $k$ independently versioned behavior-bearing components, a response's behavior is attributable only if all $k$ versions are logged. Missing one component makes two different configurations indistinguishable in logs; a regression can then be attributed only by time, which Lesson 17.5 shows is confounded.

**Worked Example:**
Manifest A: `model=snapshot-2025-10`, `temperature=0.2`, `prompt=v14`, `schema=order_v3`, `parser=2.1`. Manifest B changes only the model to `snapshot-2026-06`. The classifier flags: (1) forced? yes, A's snapshot has a retirement date; (2) parameter validity: `temperature` is rejected by B's model family → adapter must drop it and the evaluation must treat decoding as changed; (3) behavior evaluation required on prompt v14 under B.

**Knowledge Check:**
1. Why is a moving alias a manifest bug even when behavior currently looks unchanged?
2. Which manifest fields can change output interpretation without changing model output?

**Guided Practice:**
Write the manifest for a production endpoint you know. Mark each field as pinned, aliased, or unlogged, and list which forced changes could arrive in the next six months.

**Feedback Contract:**
- *Expected Evidence*: Complete field list, pin status, lifecycle dates, and the validity check each change type triggers.
- *Common Failure*: Versioning only the prompt text and treating the model name as fixed.
- *Diagnostic Hint*: Could two logged responses with identical prompt versions come from different weights?
- *Concept to Revisit*: Canonical vs. Effective Request (Module 13).

**Learning Outcome:**
Pin and log every behavior-bearing component and classify a change before evaluating it.

*(Effort: 35m instruction, 25m practice)*

---

### Lesson 17.2 — Paired Migration Evaluation and Regression Flips

**Engineering Question:**
How do we decide whether a new model or prompt is compatible when aggregate accuracy looks the same or better?

**Concepts & Definitions:**
- **Paired evaluation**: old and new manifests run on the same frozen items, with the same grader, and—for stochastic outputs—repeated samples per item.
- **Regression flip** ($b$): item passes under old, fails under new. **Improvement flip** ($c$): the reverse.
- **Behavior drift**: Chen, Zaharia, and Zou reported substantial task-dependent changes between March and June 2023 versions of GPT-3.5/GPT-4, including chain-of-thought responsiveness, refusals, and code formatting (**O**, CLM-004).
- **Prompt transfer**: Sclar et al. report up to 76 accuracy points of spread from meaning-preserving few-shot format changes on LLaMA-2-13B and weak cross-model correlation of format performance (**O**, CLM-005). A prompt tuned for one model carries no guarantee for its replacement.

**Mechanism Explanation:**
Build a 2×2 table per slice (task type, language, customer tier, tool path). Report $b$, $c$, net change, and the regression examples themselves. Because behavior changes are task-dependent, a migration can improve reasoning while breaking output format; slice tables catch this where the global mean does not.

**Quantitative Model / Derivation:**
With $n$ items, $a$ both-pass, $b$ old-pass/new-fail, $c$ old-fail/new-pass, $d$ both-fail:
$$\Delta\text{acc}=\frac{(a+c)-(a+b)}{n}=\frac{c-b}{n},\qquad r_{reg}=\frac{b}{n}$$
Concordant items cancel. Equal accuracy implies only $b=c$ (**D**, CLM-006). A paired test on discordant pairs (e.g., an exact binomial test on $c$ out of $b+c$ with $p=0.5$) uses the correct dependence structure; unpaired comparison of two means discards it.

**Worked Example:**
$n=2{,}000$, old accuracy 90.0%, new 90.5%. Discordant counts: $b=120$, $c=130$. $\Delta\text{acc}=10/2000=0.5$ pp, but $r_{reg}=6\%$ of items—120 behaviors that users relied on now fail. If 90 of the 120 are in the "structured refund" slice, the migration is blocked for that slice regardless of the global gain.

**Knowledge Check:**
1. Why does unpaired comparison of two accuracies lose power relative to the discordant-pair analysis?
2. With stochastic decoding, what must be repeated before a flip is counted?

**Guided Practice:**
Given $b=40$, $c=65$ on $n=1{,}500$, compute $\Delta\text{acc}$ and $r_{reg}$, then state what additional slice and severity data a release gate needs.

**Feedback Contract:**
- *Expected Evidence*: $\Delta\text{acc}=25/1500\approx1.67$ pp; $r_{reg}=40/1500\approx2.67\%$; slice breakdown, severity weights, and per-item repeat policy.
- *Common Failure*: Shipping on net gain without reading regression items.
- *Diagnostic Hint*: Who experiences the 40 regressions?
- *Concept to Revisit*: Paired Inference (Module 15).

**Learning Outcome:**
Evaluate a migration with paired flips and slice-level regression gates.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 17.3 — Automated Prompt Optimization as Search

**Engineering Question:**
What does a prompt optimizer actually search over, and what does it need from us to produce a trustworthy candidate?

**Concepts & Definitions:**
- **LM program**: declarative modules whose prompts and demonstrations are parameters; DSPy compiles such programs with optimizers that bootstrap demonstrations and tune prompts to maximize a supplied metric (**O**, CLM-007).
- **MIPRO**: optimizes per-module instructions and demonstrations without module-level labels, using program/data-aware proposals and a surrogate model learned from stochastic minibatch evaluations (**O**, CLM-008).
- **GEPA**: reflective prompt evolution that critiques sampled trajectories in natural language and keeps a Pareto set of candidates; its authors report gains over GRPO and MIPROv2 with fewer rollouts on evaluated tasks (**O**, CLM-009; FRONTIER, author-reported).
- **Metric**: the optimizer's only definition of "good". An invalid grader is optimized as faithfully as a valid one.

**Mechanism Explanation:**
All three are search loops: propose candidates → evaluate on (mini)batches → update proposals or surrogate → keep the best by validation score. The search consumes evaluation calls; its output is the argmax of a noisy score. Therefore: (1) metric validity is a precondition (Module 15); (2) data must be split into train (demonstration source), validation (selection), and sealed test (decision); (3) the result must pass the same paired regression gate as any migration.

**Quantitative Model / Derivation:**
Optimization cost $\approx N_{trials}\times B_{eval}\times(\text{calls per example})\times(\text{cost per call})$ plus proposal-model calls. Minibatch evaluation lowers cost per trial but raises score noise, which increases selection bias (Lesson 17.4).

**Worked Example:**
A 3-module RAG program, 40 trials, minibatch 35, 3 LM calls per example: $40\times35\times3=4{,}200$ calls plus periodic full-validation evaluations. With 300 validation items and full evaluation every 5 trials, add $8\times300\times3=7{,}200$ calls. The full-evaluation schedule dominates cost; state it in the budget.

**Knowledge Check:**
1. Why does an optimizer amplify a grader bias instead of averaging it out?
2. Why must re-optimized prompts for a new model still face the paired regression gate?

**Guided Practice:**
Specify an optimization run: program, metric and its validation evidence, train/validation/sealed-test sizes, trial budget, and the regression gate. State the hypothesis that re-optimization beats prompt carry-over and its falsifier (**H**, CLM-016).

**Feedback Contract:**
- *Expected Evidence*: Split sizes, candidate count, call budget, metric validity evidence, sealed-test policy, and falsifying observation.
- *Common Failure*: Reporting the optimizer's own best validation score as the expected production gain.
- *Diagnostic Hint*: Which data did the optimizer never see?
- *Concept to Revisit*: Grader Validity (Module 15).

**Learning Outcome:**
Frame prompt optimization as budgeted search against a validated metric with explicit data roles.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 17.4 — Selection Bias, Holdout Reuse, and the Optimizer Source Trace

**Engineering Question:**
By how much does "best validation score" overstate a candidate's true quality, and how do we keep repeated optimization from consuming the holdout?

**Concepts & Definitions:**
- **Winner's curse**: selecting the max of noisy estimates biases the winner's estimate upward (**D**, CLM-010).
- **Adaptive reuse**: each decision informed by the holdout makes later holdout results less valid; Dwork et al. show this can invalidate guarantees and propose a reusable-holdout mechanism under stated conditions (**O**, CLM-011).
- **Sealed test**: data evaluated once per release decision and never used for selection.

**Quantitative Model / Derivation:**
Stylized model: $K$ candidates with equal true score $\theta$ and independent validation noise $\sigma Z_k$, $Z_k\sim N(0,1)$. The selected score is $\theta+\sigma\max_k Z_k$:
$$\mathbb{E}[\text{bias}]=\sigma\,\mathbb{E}[\max_{k\le K}Z_k]\approx 0.56\sigma,\;1.16\sigma,\;1.54\sigma,\;1.87\sigma\quad(K=2,5,10,20)$$
For accuracy on $m$ validation items, $\sigma\approx\sqrt{p(1-p)/m}$. Correlated candidates reduce the magnitude; unequal true quality means some of the gain is real—but the direction of bias persists when selection and reporting use the same data.

**Worked Example:**
$m=200$, $p\approx0.80$: $\sigma\approx\sqrt{0.16/200}\approx0.028$ (2.8 pp). With $K=20$ equal candidates, expected bias $\approx1.87\times2.8\approx5.3$ pp. A "+5 pp" optimizer result on this validation set is compatible with no real improvement. Only the sealed test can distinguish.

**Mechanism Explanation — Pinned DSPy Trace (O, CLM-012):**
At `stanfordnlp/dspy` commit `9c900c7de0a3cc3114c23fe8202ebe48e2206ce1` (static inspection, 2026-09-27):
1. `MIPROv2.compile(student, trainset=..., valset=None)` calls `_set_and_validate_datasets`, which—when `valset` is omitted—takes the last `min(1000, int(0.8*len(trainset)))` training examples as validation and the rest as train.
2. `_optimize_prompt_parameters` tracks `best_score`/`best_program` by validation score across trials.
3. `BootstrapFewShot.compile` → `_bootstrap` iterates the trainset for up to `max_rounds`, keeping traces that pass `metric`/`metric_threshold` as demonstrations.
4. `Evaluate(devset=..., metric=..., failure_score=0.0)` assigns `failure_score` to examples that raise errors.
Implications: the optimizer's reported score is a selection score; a separate sealed test is not created for you; and errored examples are scored rather than excluded, so an error-rate change moves the metric.

**Knowledge Check:**
1. Why does a larger $K$ make the sealed test more, not less, important?
2. How can `failure_score` make a flaky provider look like a worse prompt?

**Guided Practice:**
For $m=500$, $p=0.7$, $K=10$, compute the expected bias. Then propose a policy: split sizes, maximum optimization rounds per sealed-test refresh, and how candidate count is recorded in the ledger.

**Feedback Contract:**
- *Expected Evidence*: $\sigma\approx\sqrt{0.21/500}\approx0.0205$; bias $\approx1.54\times2.05\approx3.2$ pp; explicit refresh and logging policy.
- *Common Failure*: Treating the validation set as a test set after 30 optimization runs.
- *Diagnostic Hint*: How many candidates has this validation set chosen between, across all runs?
- *Concept to Revisit*: Discovery vs. Confirmation (Module 16).

**Learning Outcome:**
Quantify selection bias and govern holdout use for repeated prompt optimization.

*(Effort: 45m instruction, 15m practice; source trace 2h)*

---

### Lesson 17.5 — Shadow, Canary, and Staged Rollout

**Engineering Question:**
How large and how long must a canary be to detect the regression we care about, and why is before/after comparison not enough?

**Concepts & Definitions:**
- **Shadow traffic**: duplicate production inputs to the new version without returning its output; measures validity, latency, cost, and offline-graded quality without user impact. Side-effecting tool calls must be stubbed or blocked.
- **Canary**: the SRE Workbook defines canarying as a partial, time-limited deployment evaluated against a control population, recommends starting from SLIs with few metrics, and warns that before/after comparison is confounded by time (**O**, CLM-013).
- **Guardrail metric**: predeclared metric whose breach aborts rollout—validity/parse rate, refusal rate, error rate, latency tail, cost per request, graded quality on sampled traffic, and user-signal proxies.

**Mechanism Explanation:**
Randomize assignment (by user or session, not by request, when conversations are multi-turn) into control and canary at the same time. Compute guardrails per arm. Fast guardrails (errors, parse failures, latency) gate early stages; slow guardrails (graded quality, user outcomes) gate later stages because they need more samples. Ramp only after each stage's predeclared sample size is reached.

**Quantitative Model / Derivation:**
To detect an absolute success-rate drop $\delta$ with two-sided level $\alpha$ and power $1-\beta$ (**D**, CLM-014):
$$n_{arm}\approx\frac{(z_{1-\alpha/2}+z_{1-\beta})^2\,[p_1(1-p_1)+p_2(1-p_2)]}{\delta^2}$$
Duration $=n_{arm}/(\text{canary request rate}\times\text{graded fraction})$. Randomizing by session makes requests clustered; inflate $n$ by the design effect or analyze at session level. Repeatedly peeking without a sequential method inflates false alarms.

**Worked Example:**
$p_1=0.95$, $p_2=0.93$, $\delta=0.02$, $\alpha=0.05$, power 0.8: $(1.96+0.84)^2\approx7.85$; variance sum $0.0475+0.0651=0.1126$; $n_{arm}\approx7.85\times0.1126/0.0004\approx2{,}210$ graded requests. At 5% canary of 40 req/s with 10% graded: $2210/(2\times0.1)\approx11{,}050$ s ≈ 3.1 h. A 20-minute canary cannot detect this regression.

**Knowledge Check:**
1. Why is randomization by session preferred for multi-turn assistants, and what does it cost statistically?
2. Why can shadow traffic not validate user-facing quality alone?

**Guided Practice:**
Design a four-stage rollout (shadow → 1% → 10% → 50% → 100%) with per-stage guardrails, sample-size targets, abort thresholds, and who can override.

**Feedback Contract:**
- *Expected Evidence*: Assignment unit, concurrent control, guardrails per stage, $n$ and duration calculations, sequential-monitoring rule, and abort/override policy.
- *Common Failure*: Comparing today's canary with last week's baseline.
- *Diagnostic Hint*: What else changed between last week and today?
- *Concept to Revisit*: Offline–Online Validity (Module 15).

**Learning Outcome:**
Design a powered, concurrently controlled rollout with predeclared guardrails.

*(Effort: 40m instruction, 20m practice)*

---

### Lesson 17.6 — Compatibility, Rollback State, and the Migration Ledger

**Engineering Question:**
What must remain true for "rollback" to restore a valid system, and how do we meet forced migration deadlines without skipping evidence?

**Concepts & Definitions:**
- **Rollback state**: in-flight sessions, cached prompt prefixes, stored structured outputs, tool-call schemas, and downstream consumers of new-version artifacts (**D**, CLM-015).
- **Dual-read compatibility**: the old version can read artifacts written by the new one (or they are migrated/quarantined).
- **Migration ledger**: per change—manifest diff, change class, lifecycle deadline, compatibility checks, paired offline results, optimizer runs and candidate counts, sealed-test result, shadow/canary evidence, rollout stage, and rollback decisions.

**Mechanism Explanation:**
Routing controls only future requests. If the new version wrote `order_v4` JSON, the old parser must accept it or the records must be converted. If a conversation started on the new model, rolling back mid-session changes persona and format; pin sessions to a version or define a hand-off. Forced migrations are scheduled backward from the retirement date: compatibility and offline evaluation first, then enough canary time to reach the powered sample size, plus a contingency window. Anthropic documents at least 60 days' retirement notice for publicly released models on its platforms (**O**, CLM-002); partner platforms set their own schedules.

**Quantitative Model / Trade-off Comparison:**
Latest safe start $=T_{retire}-(T_{compat}+T_{offline}+T_{reopt}+\sum_s T_{stage,s}+T_{contingency})$, where each canary stage time comes from Lesson 17.5. If the latest safe start is already past, the plan must cut scope (fewer slices, larger $\delta$) explicitly and record the accepted risk.

**Worked Example:**
Retirement in 60 days. Compatibility 3 d, offline paired evaluation 5 d, re-optimization and sealed test 7 d, four canary stages needing 2, 3, 4, 4 d, contingency 10 d: total 38 d. Latest safe start is day 22 after notice. Starting on day 40 forces a documented choice: skip re-optimization or accept a coarser $\delta$.

**Knowledge Check:**
1. Give two artifacts that survive a traffic rollback.
2. Why should forced-migration plans be scheduled backward from the retirement date?

**Guided Practice:**
For an agent that stores tool-call plans in a durable workflow (Module 14), list every artifact the new version writes and the rollback action for each.

**Feedback Contract:**
- *Expected Evidence*: Artifact inventory, reader compatibility, migration/quarantine steps, session pinning, and ledger entries.
- *Common Failure*: Declaring rollback complete when the router flag flips.
- *Diagnostic Hint*: What did the new version write that someone else will read tomorrow?
- *Concept to Revisit*: Durable State and Effects (Module 14).

**Learning Outcome:**
Plan state-aware rollback and schedule forced migrations with explicit evidence and accepted risk.

*(Effort: 35m instruction, 25m practice)*

---

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**
- [DSPy](https://arxiv.org/abs/2310.03714) — Khattab et al., 2023; declarative LM programs compiled by metric-driven optimizers.
- [How is ChatGPT's behavior changing over time?](https://arxiv.org/abs/2307.09009) — Chen, Zaharia, Zou, 2023; longitudinal behavior change behind stable product names.
- [Quantifying LMs' Sensitivity to Spurious Features in Prompt Design](https://arxiv.org/abs/2310.11324) — Sclar et al., ICLR 2024; format sensitivity and FormatSpread.
- [The reusable holdout](https://www.science.org/doi/10.1126/science.aaa9375) — Dwork et al., Science 2015; adaptive data analysis.
- [Canarying Releases](https://sre.google/workbook/canarying-releases/) — Google SRE Workbook; canary versus control.

**CURRENT DEFAULT:** pinned snapshots and full manifests; paired offline flips per slice; sealed test for optimizer output; shadow before exposure; concurrent randomized canary with predeclared guardrails; state-aware rollback; lifecycle-date tracking.

**WORKLOAD-DEPENDENT:** [MIPRO](https://arxiv.org/abs/2406.11695) (Opsahl-Ong et al., EMNLP 2024) and other optimizers; split sizes; canary $\delta$ and stage durations; session pinning; slice definitions.

**FRONTIER:** [GEPA](https://arxiv.org/abs/2507.19457) (Agrawal et al., ICLR 2026) reflective prompt evolution; author-reported comparisons not independently reproduced here.

**PROVIDER-SPECIFIC:** [Anthropic model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations) — lifecycle states, notice period, and parameter deprecations; other providers differ.

**LEGACY / INSUFFICIENT:** floating aliases in production; prompt-only versioning; before/after dashboards as canary; optimizer validation score as expected gain; rollback by router flag only.

**PRODUCTION SOURCE TRACE**
- Repository: `stanfordnlp/dspy`
- Revision: `9c900c7de0a3cc3114c23fe8202ebe48e2206ce1`
- Verified: 2026-09-27; static inspection only.
- Files/symbols: `dspy/teleprompt/mipro_optimizer_v2.py::{MIPROv2.compile, MIPROv2._set_and_validate_datasets}`, `dspy/teleprompt/bootstrap.py::{BootstrapFewShot.compile, BootstrapFewShot._bootstrap}`, `dspy/evaluate/evaluate.py::{Evaluate.__init__, Evaluate.__call__}`.

---

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Manifest, Change Classifier, and Compatibility Gate
- **Objective**: Build a manifest schema, per-request manifest hashing, and a classifier that diffs manifests and runs request-validity checks per target.
- **Pre-Registered Hypothesis**: Logging a complete manifest hash lets every regression in a seeded change set be attributed to one component; prompt-only versioning leaves at least one seeded change unattributable.
- **Independent Variables**: Manifest completeness, change type (model, parameter, prompt, schema, parser), alias versus snapshot.
- **Dependent Variables**: Attribution accuracy, invalid-request rate caught pre-deploy, time to classify.
- **Break & Falsify**: Seed an alias move and a deprecated parameter; if the gate passes either, the compatibility check is incomplete.
- **Alignment**: Lesson 17.1.
- **Effort Estimate**: 3h total.

### LAB B — Paired Migration Report
- **Objective**: Evaluate two model versions on a frozen set with repeated samples and produce slice-level flip tables.
- **Pre-Registered Hypothesis**: At least one slice shows regression rate above the gate threshold while global accuracy changes by less than the minimum important effect.
- **Independent Variables**: Model version, prompt (carried over versus format variants), slice, samples per item.
- **Dependent Variables**: $a,b,c,d$ per slice, $\Delta\text{acc}$, $r_{reg}$, discordant-pair test, format-variant spread.
- **Break & Falsify**: Swap in an unpaired comparison and show the lost information; if no slice regresses, report that as a valid negative result with its power.
- **Alignment**: Lesson 17.2.
- **Effort Estimate**: 3h total.

### LAB C — Optimizer Under Holdout Governance
- **Objective**: Run an automated prompt optimizer (DSPy or equivalent) with explicit train/validation/sealed-test roles and measure the validation-to-test gap.
- **Pre-Registered Hypothesis**: The best-validation candidate's score exceeds its sealed-test score by an amount consistent with the winner's-curse model for the recorded $K$ and $m$.
- **Independent Variables**: Candidate count $K$, validation size $m$, minibatch size, metric version (valid versus deliberately gamed).
- **Dependent Variables**: Validation score, sealed-test score, gap, paired flips versus carried-over prompt, call cost.
- **Break & Falsify**: Optimize against a gameable metric (e.g., keyword presence) and show sealed-test regression; test CLM-016 by comparing re-optimization versus carry-over on the sealed test.
- **Alignment**: Lessons 17.3–17.4.
- **Effort Estimate**: 3h total (plus 2h source trace).

### LAB D — Powered Canary and State-Aware Rollback
- **Objective**: Simulate shadow and staged canary with a concurrent control, then execute a rollback with persisted artifacts.
- **Pre-Registered Hypothesis**: A canary sized by the power formula detects a seeded 2-pp drop at the declared rate; a 20-minute canary does not.
- **Independent Variables**: Canary fraction, graded fraction, assignment unit, seeded $\delta$, peeking policy.
- **Dependent Variables**: Detection rate, false-alarm rate, time to decision, rollback completeness (artifacts readable by old version).
- **Break & Falsify**: Run before/after comparison across a seeded traffic-mix shift; write `v_new` artifacts before rollback and verify the old parser; any unreadable artifact falsifies rollback completeness.
- **Alignment**: Lessons 17.5–17.6 and Incident 17.1.
- **Effort Estimate**: 3h total.

---

## 07 Break / Incident Scenarios

### Incident 17.1 — The Deadline Migration

- **Incident Symptoms**: Two days before a provider retirement date, the team switches the model identifier. Offline accuracy on the regression suite rose 1.2 pp. Within an hour of full rollout: structured-extraction parse failures rise from 0.3% to 4%, refunds processed by a downstream service drop, a subset of requests return HTTP 400, and support reports a changed tone in ongoing conversations. Rolling back is impossible for the retired model; rolling to an intermediate model restores tone but parse failures persist for records written in the last hour.
- **Diagnostic Protocol (Task)**:
  1. *Formulate Competing Hypotheses*: deprecated decoding parameter; prompt format sensitivity on the new model; schema drift in outputs; grader/suite not covering the extraction slice; downstream consumer tied to old output quirks; session carry-over across versions; unrelated traffic-mix shift.
  2. *Rank Initial Plausibility*: Use the timing of 400s versus parse failures; do not assume one cause.
  3. *Identify Missing Evidence*: Manifest diffs, per-request manifest hashes, 400 response bodies, paired flips by slice, sample failed parses, artifact versions written, session start versions, canary records (if any).
  4. *Design Discriminating Tests*: Replay failing inputs under old/new manifests with parameter stripped; format-variant spread; parser against stored records; session-level split.
  5. *Execute Causal Diagnosis*: Rank interacting causes and state which alternatives the evidence excludes.
  6. *Prescribe Mitigation and Prevention*: Adapter parameter fix, slice-specific prompt or re-optimization with sealed test, artifact repair/quarantine, session pinning, backward-scheduled migration plan, powered canary.
  7. *Remeasure*: Parse rate, 400 rate, slice flips, downstream refund reconciliation, and artifact readability.

---

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem: Multi-Model Estate Under Forced Retirements

You own twelve LLM endpoints across two providers, using five model snapshots. Two snapshots receive retirement notices with 60 and 90 days' notice. Three endpoints persist structured outputs; one is a multi-turn agent with durable workflows. The team wants to use the forced migrations to adopt an automated prompt optimizer.

**Required Deliverables**:
1. Manifest inventory with pin status and lifecycle dates.
2. Change classification and compatibility checks for each affected endpoint.
3. Paired evaluation design with slices, repeat policy, and regression gates.
4. Optimizer plan with split roles, candidate budget, winner's-curse estimate, and sealed-test policy.
5. Rollout plan with shadow, stage sizes from the power formula, guardrails, and abort rules.
6. State-aware rollback plan and backward-scheduled timeline with contingency.
7. Pinned DSPy (or equivalent) source trace.
8. Diagnosis and prevention plan for Incident 17.1.

## 09 Required Evidence & Rubric

### Required Artifact: Production Source Trace
Trace the pinned DSPy path from `MIPROv2.compile` through dataset splitting and best-candidate selection, plus `BootstrapFewShot` demonstration filtering and `Evaluate` error scoring. State what was statically inspected and what was not executed.

### Rubric Dimensions
- **Versioning and Change Control**: *Insufficient* versions prompts only. *Competent* pins full manifests and classifies changes. *Strong* catches forced incompatibilities before deadlines and attributes every regression.
- **Migration Evidence**: *Insufficient* compares aggregates. *Competent* reports paired flips by slice. *Strong* ties regressions to severity and blocks by slice with power stated.
- **Optimization Governance**: *Insufficient* reports validation gains. *Competent* uses a sealed test. *Strong* quantifies selection bias and limits holdout reuse.
- **Rollout and Rollback**: *Insufficient* uses before/after dashboards. *Competent* runs a concurrent canary. *Strong* sizes stages, controls peeking, and proves rollback completeness for persisted state.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Manifest and change classification | 17.1 | LAB A | Incident / Mastery 1–2 | Manifest, classifier, compatibility report |
| Paired migration evaluation | 17.2 | LAB B | Mastery 3 / Incident | Flip tables by slice |
| Prompt optimization as search | 17.3 | LAB C | Mastery 4 | Optimization plan and budget |
| Selection bias and holdout governance | 17.4 | LAB C | Mastery 4, 7 | Validation–test gap, DSPy trace |
| Powered canary rollout | 17.5 | LAB D | Mastery 5 | Stage sizes, guardrails |
| State-aware rollback and ledger | 17.6 | LAB D | Mastery 6 / Incident | Artifact inventory, timeline |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria
A learner successfully completing Module 17 must be able to:
1. Pin every behavior-bearing component and attribute a regression to one manifest change.
2. Detect forced incompatibilities (retired models, deprecated parameters) before deployment.
3. Report paired regression flips by slice and gate on them.
4. Run a prompt optimizer with a sealed test and quantify selection bias.
5. Size and run a concurrently controlled canary.
6. Execute a rollback that keeps persisted artifacts valid.

### Module Wrap-Up (Final Mental Model Reconstruction)
- **The Core Invariant**: A harness is a versioned configuration; behavior changes whenever any component changes, whether or not you changed it.
- **The Change Path**: manifest diff → validity → paired flips → governed optimization → shadow → powered canary → ramp or state-aware rollback → ledger.
- Aggregate scores approve nothing on their own; paired regressions, sealed tests, concurrent controls, and rollback state do.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

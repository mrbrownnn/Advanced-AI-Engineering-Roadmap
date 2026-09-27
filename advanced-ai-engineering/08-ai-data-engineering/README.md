# Module 08 — AI Data Engineering

## 00 Why This Module Exists

Model behavior is bounded by the data pipeline that created and evaluates it. A corpus can be syntactically valid yet semantically wrong, reproducible in name yet mutable in bytes, “deduplicated” under one detector yet contaminated under another, or improved on average while losing rare but important tails.

This module engineers the lifecycle:

$$
\text{source}\to\text{immutable intake}\to\text{contract and lineage}
\to\text{validate/quarantine}\to\text{transform/deduplicate/split}
\to\text{label or synthesize}\to\text{release}\to\text{monitor/rollback}.
$$

It covers provenance, validation, drift, leakage, deduplication, contamination, synthetic data, and active learning. Retrieval indexing belongs to Module 09, model adaptation to Module 19, evaluation-program architecture to Module 15, and operational telemetry to Module 23.

**Research cutoff:** 2026-09-27.

**Module Orientation**

- **Engineering problem:** release data that is fit, traceable, replayable, leakage-controlled, measurable under shift, and safely replaceable.
- **What you will do:** build immutable manifests and lineage; implement layered validation and quarantine; diagnose drift and training-serving skew; red-team leakage and decontamination; control synthetic mixtures; operate a human-label and active-learning loop; and defend promotion, deletion, and rollback.
- **Environment:** Python 3.10+ with dataframe, hashing, validation, and statistical tooling; an object-store or filesystem snapshot fixture; optional distributed processing for scale tests. Pin source snapshots, schemas, transforms, tokenizers, detectors, label policy, and execution environment.
- **Evidence rule:** distinguish source observation (**O**), assumption-backed derivation (**D**), and telemetry-dependent hypothesis (**H**). A passed check is evidence only for the invariant that check actually measures.

## 01 Baseline Assumptions

- Module 00: experimental units, provenance, leakage, sampling, uncertainty, and falsification.
- Module 07: behavior events, calibration, distribution shift, slices, and regression.
- Data fundamentals: schemas, joins, hashes, partitions, snapshots, transactions, and access controls.
- ML fundamentals: train/calibration/test boundaries, features/labels, fitting, and target-population risk.

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
  security: REQUIRED
  economics: SELECTIVE
  architecture_tradeoff: REQUIRED
  research_connection: SELECTIVE

estimated_effort:
  instruction: 5h
  guided_practice: 3h
  labs: 18h
  assessment: 3h
  source_trace: 2h
  total: 31h
```

The learner must build replayable manifests, layered validation, drift experiments, leak-resistant splits, auditable dedup/decontamination, controlled synthetic mixtures, cost-aware active learning, and safe promotion/rollback. They must also preserve consent/license/authority and retention/deletion obligations as executable release constraints rather than documentation-only fields.

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER

## 03 Knowledge Map

```text
source bytes + authority + collection time
                 |
          immutable identity
                 |
     contract + datasheet + lineage DAG
                 |
  structural -> statistical -> relational -> semantic -> task validation
                 |
           quarantine / approve
                 |
 normalize -> deduplicate -> split -> label/synthesize
                 |
       versioned release + consumer compatibility
                 |
     drift + delayed labels + incident feedback
                 `----> new candidate release, never silent mutation
```

## 04 Lessons

### Lesson 8.1 — Data Contracts, Identity, and Lineage

**Engineering Question:**
Can this exact dataset be explained and rebuilt?

**Concepts & Definitions:**

Datasheets document motivation, composition, collection, processing, use, distribution, and maintenance. An executable manifest adds source snapshot/hash, stable record IDs, parent-child lineage, transform revision/configuration/environment, schema, ordering, seeds, filters, split assignments, label provenance, quality results, and consumer compatibility.

**Quantitative Model / Derivation:**
A dataset display name or mutable URI is not identity. If output is $D=f(S,C,E,R)$ for source bytes $S$, code/config $C$, execution semantics $E$, and randomness/external results $R$, replay requires those arguments or an explicit statement that replay is impossible. Documentation can be stale; hashes do not establish consent, license, authority, retention permission, quality, or fitness. Keep both, and propagate deletion/expiry obligations through derived releases.

**Break cases:** upstream overwrite, nondeterministic file order, unpinned tokenizer, mutable API response, undocumented manual edit, changed normalization, and a seed that does not cover distributed nondeterminism.

**Worked Example:**
Two releases with the same display name but different source hashes are different datasets. Conversely, identical bytes can still have different legal or temporal eligibility when authority, consent, or retention metadata changes.

**Knowledge Check:**
1. Why does a content hash not prove that a record may be retained or trained on?
2. Which inputs besides source bytes must be pinned for replay?

**Guided Practice:**
Build a manifest for one release, replay it twice, compare bytes/counts/order, then issue one source deletion and trace every affected derivative and consumer.

**Feedback Contract:**
- *Expected Evidence*: Immutable IDs, complete inputs/environment, lineage edges, replay diff, eligibility metadata, and deletion acknowledgement.
- *Common Failure*: Calling a mutable URI or seed alone reproducible.
- *Diagnostic Hint*: Which unrecorded input can still change output bytes or eligibility?
- *Concept to Revisit*: Dataset Identity and Lineage Closure.

**Learning Outcome:**
Produce a manifest that supports diff, rollback, deletion propagation, and incident reconstruction.

*(Effort: 45m instruction, 30m practice)*

### Lesson 8.2 — Layered Validation and Quarantine

**Engineering Question:**
What can a passed validation gate actually rule out?

**Concepts & Definitions:**

**Mechanism Explanation:**
Validation layers include:

1. byte/container readability and checksums;
2. schema, type, presence, range, enumeration, and cardinality;
3. statistical distributions and slice counts;
4. relational keys, join multiplicity, referential and temporal integrity;
5. semantic units, language, encoding, evidence authority, and label meaning;
6. duplicates, leakage, policy/privacy and task-level canaries.

Infer schemas for exploration, then review and version them. A bad baseline can make inferred anomalies look normal. Failed batches enter quarantine with reason codes; overrides require owner, justification, scope, expiry, and revalidation. A schema-green batch may still contain milliseconds interpreted as seconds or a many-to-many join explosion.

**Source trace:** current TFDV exposes schema/statistics validation and drift/skew records. This is one implementation, not the definition of data quality.

**Worked Example:**
A timestamp column can pass integer type/range checks while milliseconds are interpreted as seconds. A semantic unit check or temporal canary detects a failure that schema validation cannot.

**Knowledge Check:**
1. Why is a schema-green batch not necessarily task-valid?
2. What governance must accompany a manual override?

**Guided Practice:**
Inject one defect at each validation layer and record whether the gate detects, quarantines, overrides, and revalidates it.

**Feedback Contract:**
- *Expected Evidence*: Check-to-invariant matrix, quarantined samples, false pass/fail audit, owner, expiry, and rollback action.
- *Common Failure*: Treating inferred schema or aggregate distributions as semantic truth.
- *Diagnostic Hint*: State exactly which bad worlds remain possible after the check passes.
- *Concept to Revisit*: Layered Validation Boundaries.

**Learning Outcome:**
Map every check to its invariant, blind spots, action, owner, and rollback.

*(Effort: 40m instruction, 25m practice)*

### Lesson 8.3 — Drift, Skew, and Alert Evidence

**Engineering Question:**
What changed, and does the change harm the target?

**Concepts & Definitions:**

**Quantitative Model / Derivation:**
Use the factorization:

$$P(X,Y)=P(Y|X)P(X)=P(X|Y)P(Y).$$

- covariate shift: $P(X)$ changes;
- label shift: $P(Y)$ changes under additional assumptions;
- concept shift: $P(Y|X)$ changes;
- training-serving skew: pipeline or population differs between training and inference.

Unlabeled $X$ monitoring cannot generally identify label or concept shift. Drift signals include missingness/cardinality, quantiles, category shares, distances/tests, embedding or classifier two-sample methods, and slice/task canaries. High-dimensional projections may miss rare slices; many tests create false alerts; huge samples make tiny differences statistically detectable.

For identical positive bins:

$$PSI=\sum_i(p_i-q_i)\ln\frac{p_i}{q_i}.$$

PSI changes with bins and smoothing. There is no universal cutoff or quality mapping. Calibrate alerts against incidents, downstream labels, lead time, operator cost, and simpler baselines.

**Worked Example:**
A marginal language-share shift can raise PSI while task quality remains stable; a rare severe-label change can harm utility while the aggregate PSI stays small. Neither observation identifies concept shift without labels or a justified causal proxy.

**Knowledge Check:**
1. Can unlabeled $X$ monitoring identify $P(Y\mid X)$ change in general?
2. Why does a universal PSI threshold lack a stable quality meaning?

**Guided Practice:**
Create controlled covariate, label, concept, and pipeline-skew cases. Cross alert signals with delayed labels and rare slices.

**Feedback Contract:**
- *Expected Evidence*: Population/window, bins/smoothing, multiple-test policy, task labels where available, and alert-to-incident utility.
- *Common Failure*: Naming every distribution change “concept drift.”
- *Diagnostic Hint*: Which factor in $P(X,Y)$ is actually observed?
- *Concept to Revisit*: Shift Identifiability.

**Learning Outcome:**
Report symptom → competing shift hypotheses → missing labels/evidence → discriminating measurement → action → remeasurement.

*(Effort: 50m instruction, 30m practice)*

### Lesson 8.4 — Leakage, Deduplication, and Contamination

**Engineering Question:**
Did unavailable or evaluation information cross the learning boundary?

**Concepts & Definitions:**

Leakage can enter through future timestamps, same entity/session across splits, relatives/derivatives, fitted preprocessing before splitting, label-derived features, target-aware filtering, repeated prompt templates, benchmark items in pre/post-training, or selection on the test set.

**Mechanism Explanation:**
Split by the deployment unit and time before fitting transformations. Then audit exact and near relations across all splits and training stages. Exact hash, normalized text, n-gram, MinHash, embedding, and semantic matchers each define different equivalence relations. Record thresholds, clusters, canonical policy, removals, false-merge audits, and residual risk.

Lee et al. provide scoped evidence that deduplication reduced memorized output and overlap in studied LM corpora. Dedup is not monotonically beneficial: legitimate templates, quotations, repeated events, and minority patterns can be removed. A clean result means “no match under this detector on these accessible snapshots,” not “never seen.”

**Worked Example:**
An exact matcher can miss a translated benchmark item; a loose semantic matcher can remove legitimate same-topic documents. Audit detector precision/recall on labeled pairs and report accessible-corpus scope instead of “zero contamination.”

**Knowledge Check:**
1. Why must grouped/time splits precede fitting transforms?
2. What does a clean detector result fail to prove?

**Guided Practice:**
Construct exact, normalized, paraphrase, translation, entity-relative, and unrelated-hard-negative pairs. Compare matchers and inspect false merges/misses.

**Feedback Contract:**
- *Expected Evidence*: Equivalence definition, thresholds, labeled audit sample, clusters, removals, downstream deltas, and residual-risk statement.
- *Common Failure*: Treating one matcher as complete or deduplication as monotonically beneficial.
- *Diagnostic Hint*: Which transformation preserves benchmark information while defeating the detector?
- *Concept to Revisit*: Detector-Relative Contamination.

**Learning Outcome:**
Defend evaluation independence without claiming perfect contamination detection.

*(Effort: 45m instruction, 30m practice)*

### Lesson 8.5 — Synthetic Data as a Versioned Dependency

**Engineering Question:**
Does generated data add coverage or amplify the generator's blind spots?

**Concepts & Definitions:**

**Mechanism Explanation:**
Each synthetic record needs seed/parent links, generator and tokenizer revision, system/user prompt, decoding/seed, tools/evidence, filters, judge/labeler, policy decisions, and generation time. Preserve real versus synthetic origin and mixture weights.

Shumailov et al. show model-collapse behavior in recursive regimes; Seddik et al. analyze conditions for synthetic-only versus mixed regimes. These results reject “synthetic data is free real data,” not all augmentation. Test generator families, real-data anchors, repeated generations, tail/subgroup retention, novelty, privacy/memorization, judge correlation, and downstream utility at equal training budget.

**Break cases:** same model generates and judges; easy-example amplification; rare-mode loss; benchmark leakage through prompts; synthetic duplicates; error feedback; and quality filters that narrow style/diversity.

**Worked Example:**
Compare real-only with 10%, 30%, and 60% synthetic mixtures at equal training work. A mean gain with rare-slice loss is a trade-off, not universal improvement; correlated generator/judge error is a competing explanation.

**Knowledge Check:**
1. Why is synthetic ancestry required for rollback?
2. What risk arises when the same model family generates and filters examples?

**Guided Practice:**
Run a controlled mixture ablation with a real-data anchor, independent audit labels, duplicate/novelty checks, and tail slices.

**Feedback Contract:**
- *Expected Evidence*: Complete generation provenance, mixture weights, equal-budget comparison, independent audit, tail results, and removable lineage.
- *Common Failure*: Discarding origin or selecting only by the generator's own judge.
- *Diagnostic Hint*: Which errors are shared by generator, filter, and downstream model?
- *Concept to Revisit*: Synthetic Feedback Correlation.

**Learning Outcome:**
Ship synthetic data only as an ablated, lineage-complete mixture with removal and rollback.

*(Effort: 40m instruction, 30m practice)*

### Lesson 8.6 — Active Learning and Human Label Systems

**Engineering Question:**
Which example is worth labeling next, at what cost?

**Concepts & Definitions:**

**Mechanism Explanation:**
The loop scores a target pool, selects a batch, labels/adjudicates it, updates the model/data, and evaluates on an independent representative holdout. Query strategies can use uncertainty, expected change/error reduction, diversity, density, disagreement, coverage, or hybrids. The label system also versions guidelines, qualification, blind assignment, abstention, disagreement, adjudication, reviewer drift, privacy controls, and correction propagation.

Uncertainty-only batches can be redundant, out-of-distribution, adversarial noise, or artifacts of early miscalibration. Batch selection needs diversity and representativeness; the objective needs annotation time, expertise, disagreement, abstention, and delay—not item count alone.

Plot representative holdout utility against cumulative annotation cost. Keep initialization, model-update schedule, annotator policy, and test set fixed when comparing strategies. The adaptively queried pool is selection-biased and is not the target test distribution without a justified estimator.

**Worked Example:**
If strategy A gains the same holdout utility using fewer items but twice the expert minutes, it is not more label-efficient under a time/cost objective. Report utility against cumulative annotation cost, not item count alone.

**Knowledge Check:**
1. Why is the adaptively selected pool not a representative test set?
2. Which disagreement indicates ambiguous guidance rather than annotator failure?

**Guided Practice:**
Compare random, uncertainty, and diversity-aware batches under equal annotation minutes. Blind a shared subset, adjudicate disagreements, and measure representative holdout utility.

**Feedback Contract:**
- *Expected Evidence*: Selection logs, guideline/annotator revisions, time and disagreement, adjudication, independent holdout, and cost-normalized curve.
- *Common Failure*: Evaluating on the selected pool or ignoring expert time.
- *Diagnostic Hint*: Hold the evaluation population fixed while acquisition changes.
- *Concept to Revisit*: Adaptive Sampling and Label-System QA.

**Learning Outcome:**
Operate a query-label-audit loop that improves target utility rather than its own queue metric.

*(Effort: 45m instruction, 30m practice)*

## 05 Literature & Production Source Map

**REFERENCE / BASELINE**

- [Datasheets for Datasets](https://arxiv.org/abs/1803.09010) — Gebru et al. (CACM 2021).
- [Data Validation for Machine Learning](https://proceedings.mlsys.org/paper_files/paper/2019/file/928f1160e52192e3e0017fb63ab65391-Paper.pdf) — Breck et al. (MLSys 2019).
- [Deduplicating Training Data Makes Language Models Better](https://arxiv.org/abs/2107.06499) — Lee et al. (ACL 2022).
- [A Survey of Active Learning for Natural Language Processing](https://arxiv.org/abs/2210.10109) — Zhang, Strubell, and Hovy (EMNLP 2022).

**CURRENT DEFAULT:** immutable versions, lineage, layered gates, split-before-fit, cross-split duplicate audits, separately identifiable synthetic/feedback data, quarantine, canary, and rollback.

**WORKLOAD-DEPENDENT:** drift metric/threshold, semantic match rule, synthetic mixture/filter, active-learning query strategy, and human adjudication policy.

**FRONTIER:** post-training contamination detection, semantic decontamination, automated data valuation, learned quality filters, and adaptive synthetic mixtures. Treat 2025–2026 results as scoped experiments, not defaults.

**LEGACY:** one mutable “latest” dataset; schema-only quality; a universal PSI threshold; random row split for grouped/time data; exact-match-only decontamination; synthetic origin discarded.

**PRODUCTION SOURCE TRACE**

- Repository: `tensorflow/data-validation`
- Revision: `aaa0ae1902262cac7b306358588d61e8521f3b25`
- Verified: 2026-09-26, static inspection only.
- Files/symbols: `api/validation_api.py::validate_statistics`, `statistics/stats_options.py::StatsOptions`, `utils/display_util.py::get_drift_skew_dataframe`.
- Scope: proves behavior of this pinned upstream snapshot, not every TFX pipeline or universal validation semantics.

### LAYER 2: ENGINEERING PRACTICE LAYER

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Replayable Release and Layered Gate

- **Objective:** ingest two sources into immutable snapshots and build lineage, layered gates, quarantine, controlled overrides, deletion propagation, and a consumer canary.
- **Pre-Registered Hypothesis:** replay with all declared inputs pinned will reproduce the release, while at least one schema-green semantic defect will require a deeper gate.
- **Independent Variables:** source version/order, transform environment, defect type, override, and deletion event.
- **Dependent Variables:** byte/count/order equality, gate detection, false pass/fail, quarantine, replay time, deletion convergence, and canary outcome.
- **Break & Falsify:** inject unit, encoding, join multiplicity, missing-ID, mutable-source, eligibility, and schema-green semantic failures.
- **Alignment:** Lessons 8.1 and 8.2.
- **Effort Estimate:** 4.5h.

### LAB B — Drift Without a Single-Score Story

- **Objective:** create controlled shift families and connect alert evidence to delayed task consequences.
- **Pre-Registered Hypothesis:** no one marginal drift score will correctly identify every injected cause or harm state.
- **Independent Variables:** covariate/label/concept/pipeline shift, magnitude, binning/smoothing, sample size, and slice prevalence.
- **Dependent Variables:** distribution signals, task quality, alert precision/lead time, false alerts, and slice harm.
- **Break & Falsify:** include high-signal/no-harm and low-aggregate-signal/rare-severe-harm counterexamples.
- **Alignment:** Lesson 8.3.
- **Effort Estimate:** 4h.

### LAB C — Leakage and Decontamination Red Team

- **Objective:** construct leakage and related-item attacks, compare detectors, rebuild clean splits, and bound residual risk.
- **Pre-Registered Hypothesis:** exact matching will miss declared semantic derivatives, while looser detectors will introduce false merges.
- **Independent Variables:** relation type, detector, threshold, split unit/time, and accessible corpus scope.
- **Dependent Variables:** detector precision/recall on audited pairs, clusters/removals, score change, tail loss, and residual uncertainty.
- **Break & Falsify:** include entity/time leakage, fit-before-split, exact/near/paraphrase/translation derivatives, legitimate templates, and minority patterns.
- **Alignment:** Lesson 8.4.
- **Effort Estimate:** 4.5h.

### LAB D — Synthetic Mixtures and Active Labeling

- **Objective:** compare controlled synthetic mixtures and active-label strategies at equal training and annotation budgets.
- **Pre-Registered Hypothesis:** mixture and query-policy gains will be workload-dependent and at least one correlated-error/rare-mode stress will reverse an aggregate gain.
- **Independent Variables:** real/synthetic ratio, recursive round, generator/judge, query policy, annotation budget, and shift.
- **Dependent Variables:** representative holdout and slice utility, novelty/duplicates, ancestry, audit agreement, annotation time/disagreement, and cost-normalized learning curve.
- **Break & Falsify:** inject correlated generator/judge error, rare modes, early miscalibration, redundant uncertain cases, and temporal shift.
- **Alignment:** Lessons 8.5 and 8.6.
- **Effort Estimate:** 5h.

## 07 Break / Incident Scenarios

### Incident 08.1 — The “Clean” Dataset Release Regresses Production

**Incident Symptoms:**
A new release passes schema checks, removes more duplicates, adds synthetic rare-class examples, and reports low aggregate drift. After training, one language slice regresses, benchmark score rises suspiciously, online labels arrive with changed semantics, and one deletion request is absent from a derived release.

**Diagnostic Protocol (Task):**
1. **Form Competing Hypotheses:** join/unit error, over-deduplication, leakage/contamination, synthetic feedback/judge bias, label-policy change, deletion-lineage gap, concept shift, evaluator change, or unrelated training variance.
2. **Identify Missing Evidence:** manifests, hashes, transform/environment and detector revisions, duplicate clusters, split/entity/time lineage, synthetic ancestry, eligibility/deletion ledger, label guides, slice statistics, and paired outputs.
3. **Design Discriminating Measurements:** replay the release, audit units/joins and labels, rescore old/new outputs with fixed evaluators, inspect removed clusters, and rebuild factorial candidates varying one component.
4. **Rank Explanations:** use matched deltas, temporal ordering, audited labels/matches, repeated training where needed, and uncertainty; permit interacting causes.
5. **Intervene:** quarantine or roll back the smallest supported component, repair lineage/labels/detectors, restore independent anchors, and propagate required deletion.
6. **Remeasure:** repeat byte/count replay, label/slice quality, contamination audit, deletion convergence, downstream utility, and rollback criteria under the same contract.

### LAYER 3: MASTERY / ASSESSMENT LAYER

## 08 Mastery Assessment

### Enterprise Transfer Problem — Govern a Multilingual AI Data Platform

Design a data platform for a multilingual assistant with pretraining documents, SFT examples, preferences, evaluation anchors, retrieval content, production feedback, and synthetic augmentation.

**Required Deliverables:**
1. contracts, immutable identities, source authority/consent/license/retention fields, and lineage with deletion propagation;
2. replay proof and a declared gap for every non-replayable dependency;
3. structural, statistical, relational, semantic, policy, and task gates with quarantine/override lifecycle;
4. drift and delayed-label telemetry with competing hypotheses and calibrated alerts;
5. leak-resistant split, deduplication, and contamination audits with false-merge/miss evidence;
6. synthetic ancestry, equal-budget mixture ablation, and correlated generator/judge stress;
7. label guide, qualification, blind assignment, disagreement/adjudication, correction, and active-learning economics;
8. pinned production source trace and generalizability boundary;
9. promotion, canary, rollback, deletion, and incident procedures;
10. an experiment separating data, training variance, model, and evaluator causes.

## 09 Required Evidence & Rubric

### Required Artifact: Versioned Data Release Dossier

Submit manifests and lineage, replay and deletion-convergence output, validation/quarantine records, split/dedup/contamination ledger, label/adjudication log, synthetic/active-learning ancestry, source trace, and promotion/rollback decision.

### Rubric Dimensions

- **Provenance:** *Insufficient* uses names/URIs only. *Competent* identifies content and transformations. *Strong* proves replay, eligibility, deletion propagation, and explicit gaps.
- **Validation:** *Insufficient* reports schema pass. *Competent* maps each layered gate to action. *Strong* measures blind spots, overrides, false decisions, and consumer canaries.
- **Quantitative Discipline:** *Insufficient* quotes drift/dedup scores. *Competent* states units/bins/populations/uncertainty. *Strong* ties signals to delayed outcomes and decision cost.
- **Independence:** *Insufficient* splits rows casually. *Competent* aligns split/fit/dedup boundaries with deployment. *Strong* red-teams semantic relatives and residual risk.
- **Feedback Safety:** *Insufficient* mixes synthetic/feedback data irreversibly. *Competent* preserves ancestry. *Strong* isolates correlated error, tail effects, human-label QA, and removal.
- **Diagnosis:** *Insufficient* blames “bad data.” *Competent* uses diffs/audits/ablations. *Strong* ranks interacting causes and remeasures after a narrow intervention.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Contracts and lineage | 8.1 | LAB A | Mastery | Manifest and replay proof |
| Validation and quarantine | 8.2 | LAB A | Incident | Layered gate and overrides |
| Drift diagnosis | 8.3 | LAB B | Incident / Mastery | Controlled shifts and alert evidence |
| Leakage/dedup/contamination | 8.4 | LAB C | Mastery | Detector audit and ledger |
| Synthetic data | 8.5 | LAB D | Incident / Mastery | Ancestry and mixture ablation |
| Active learning | 8.6 | LAB D | Mastery | Cost-normalized holdout curve |

## 11 Exit Criteria & Module Wrap-Up

### Exit Criteria

A learner passes when they can replay a release, break schema-green data, distinguish shift hypotheses, expose leakage beyond exact matches, quantify dedup false merges, isolate synthetic ancestry, operate a reproducible label/adjudication loop, evaluate active learning on a representative holdout, propagate deletion, trace pinned validation code, and defend promotion/rollback.

### Module Wrap-Up (Final Mental Model Reconstruction)

Data engineering is controlled evidence transformation. Preserve identity, authority, eligibility, and ancestry; validate progressively; keep evaluation independent; measure drift against consequences; and never let feedback data erase the distinction between observation and model-generated belief.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

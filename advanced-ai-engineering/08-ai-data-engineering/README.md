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

**Research cutoff:** 2026-09-26.

- **Engineering problem:** release data that is fit, traceable, replayable, leakage-controlled, measurable under shift, and safely replaceable.
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
  falsification: REQUIRED
```

The learner must build replayable manifests, layered validation, drift experiments, leak-resistant splits, auditable dedup/decontamination, controlled synthetic mixtures, cost-aware active learning, and safe promotion/rollback.

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

**Question:** Can this exact dataset be explained and rebuilt?

Datasheets document motivation, composition, collection, processing, use, distribution, and maintenance. An executable manifest adds source snapshot/hash, stable record IDs, parent-child lineage, transform revision/configuration/environment, schema, ordering, seeds, filters, split assignments, label provenance, quality results, and consumer compatibility.

A dataset display name or mutable URI is not identity. If output is $D=f(S,C,E,R)$ for source bytes $S$, code/config $C$, execution semantics $E$, and randomness/external results $R$, replay requires those arguments or an explicit statement that replay is impossible. Documentation can be stale; hashes do not establish consent, quality, or fitness. Keep both.

**Break cases:** upstream overwrite, nondeterministic file order, unpinned tokenizer, mutable API response, undocumented manual edit, changed normalization, and a seed that does not cover distributed nondeterminism.

**Outcome:** produce a manifest that supports diff, rollback, deletion propagation, and incident reconstruction.

### Lesson 8.2 — Layered Validation and Quarantine

**Question:** What can a passed validation gate actually rule out?

Validation layers include:

1. byte/container readability and checksums;
2. schema, type, presence, range, enumeration, and cardinality;
3. statistical distributions and slice counts;
4. relational keys, join multiplicity, referential and temporal integrity;
5. semantic units, language, encoding, evidence authority, and label meaning;
6. duplicates, leakage, policy/privacy and task-level canaries.

Infer schemas for exploration, then review and version them. A bad baseline can make inferred anomalies look normal. Failed batches enter quarantine with reason codes; overrides require owner, justification, scope, expiry, and revalidation. A schema-green batch may still contain milliseconds interpreted as seconds or a many-to-many join explosion.

**Source trace:** current TFDV exposes schema/statistics validation and drift/skew records. This is one implementation, not the definition of data quality.

**Outcome:** map every check to its invariant, blind spots, action, owner, and rollback.

### Lesson 8.3 — Drift, Skew, and Alert Evidence

**Question:** What changed, and does the change harm the target?

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

**Outcome:** report symptom → competing shift hypotheses → missing labels/evidence → discriminating measurement → action → remeasurement.

### Lesson 8.4 — Leakage, Deduplication, and Contamination

**Question:** Did unavailable or evaluation information cross the learning boundary?

Leakage can enter through future timestamps, same entity/session across splits, relatives/derivatives, fitted preprocessing before splitting, label-derived features, target-aware filtering, repeated prompt templates, benchmark items in pre/post-training, or selection on the test set.

Split by the deployment unit and time before fitting transformations. Then audit exact and near relations across all splits and training stages. Exact hash, normalized text, n-gram, MinHash, embedding, and semantic matchers each define different equivalence relations. Record thresholds, clusters, canonical policy, removals, false-merge audits, and residual risk.

Lee et al. provide scoped evidence that deduplication reduced memorized output and overlap in studied LM corpora. Dedup is not monotonically beneficial: legitimate templates, quotations, repeated events, and minority patterns can be removed. A clean result means “no match under this detector on these accessible snapshots,” not “never seen.”

**Outcome:** defend evaluation independence without claiming perfect contamination detection.

### Lesson 8.5 — Synthetic Data as a Versioned Dependency

**Question:** Does generated data add coverage or amplify the generator's blind spots?

Each synthetic record needs seed/parent links, generator and tokenizer revision, system/user prompt, decoding/seed, tools/evidence, filters, judge/labeler, policy decisions, and generation time. Preserve real versus synthetic origin and mixture weights.

Shumailov et al. show model-collapse behavior in recursive regimes; Seddik et al. analyze conditions for synthetic-only versus mixed regimes. These results reject “synthetic data is free real data,” not all augmentation. Test generator families, real-data anchors, repeated generations, tail/subgroup retention, novelty, privacy/memorization, judge correlation, and downstream utility at equal training budget.

**Break cases:** same model generates and judges; easy-example amplification; rare-mode loss; benchmark leakage through prompts; synthetic duplicates; error feedback; and quality filters that narrow style/diversity.

**Outcome:** ship synthetic data only as an ablated, lineage-complete mixture with removal and rollback.

### Lesson 8.6 — Active Learning and Human Label Systems

**Question:** Which example is worth labeling next, at what cost?

The loop scores a target pool, selects a batch, labels/adjudicates it, updates the model/data, and evaluates on an independent representative holdout. Query strategies can use uncertainty, expected change/error reduction, diversity, density, disagreement, coverage, or hybrids.

Uncertainty-only batches can be redundant, out-of-distribution, adversarial noise, or artifacts of early miscalibration. Batch selection needs diversity and representativeness; the objective needs annotation time, expertise, disagreement, abstention, and delay—not item count alone.

Plot representative holdout utility against cumulative annotation cost. Keep initialization, model-update schedule, annotator policy, and test set fixed when comparing strategies. The adaptively queried pool is selection-biased and is not the target test distribution without a justified estimator.

**Outcome:** operate a query-label-audit loop that improves target utility rather than its own queue metric.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Replayable Release and Layered Gate

- Ingest two raw sources into immutable snapshots; build manifest/lineage, schema, semantic and relational checks, quarantine, overrides, and a consumer canary.
- Break units, encoding, join multiplicity, missing IDs, mutable source, and a schema-green semantic field.
- Artifact: replay script, manifests, validation results, quarantined samples, override audit, source trace, and byte/count equality check.

### LAB B — Drift Without a Single-Score Story

- Create controlled covariate, label, concept, pipeline-skew, and rare-slice changes.
- Measure missingness, category/quantile shifts, PSI sensitivity to bins, a two-sample method, task quality with delayed labels, alert precision/lead time, and multiple slices.
- Falsify any claim that one marginal signal identifies the cause or predicts harm universally.

### LAB C — Leakage and Decontamination Red Team

- Construct entity/time leakage, fitted-preprocessor leakage, exact/near/paraphrase duplicates, translations, and benchmark derivatives.
- Compare detectors; audit false merges/misses; rebuild splits; measure score change and residual uncertainty.
- Artifact: contamination ledger and a bounded claim, never “zero contamination.”

### LAB D — Synthetic Mixtures and Active Labeling

- Compare real-only, controlled synthetic mixtures, recursive rounds, random labeling, uncertainty sampling, and diversity-aware selection at equal training/annotation cost.
- Measure task/tail/subgroup utility, novelty/duplicates, judge agreement, mixture ancestry, label time/disagreement, and held-out label-efficiency curves.
- Break with correlated generator/judge error, rare modes, early miscalibration, redundant uncertain examples, and temporal shift.

## 07 Break / Incident Scenarios

### Incident 08.1 — The “Clean” Dataset Release Regresses Production

A new release passes schema checks, removes more duplicates, adds synthetic rare-class examples, and reports low aggregate drift. After training, one language slice regresses, benchmark score rises suspiciously, and online labels arrive with changed semantics.

The learner must rank: join/unit errors, over-dedup, leakage/contamination, synthetic feedback/judge bias, label-policy change, concept shift, evaluator change, or unrelated training variance. Recover manifests, source hashes, transform and detector versions, duplicate clusters, split/entity/time lineage, synthetic ancestry, label guidelines, slice statistics, and paired model outputs. Rebuild factorial releases, audit labels/matches, restore independent anchors, intervene narrowly, then remeasure the same data and model contracts.

## 08 Mastery Assessment

Design a data platform for a multilingual assistant with pretraining documents, SFT examples, preferences, evaluation anchors, retrieval content, production feedback, and synthetic augmentation. Deliver contracts, immutable identities, lineage/deletion propagation, layered gates, drift/label telemetry, leakage and contamination audits, dedup thresholds, synthetic controls, active-learning economics, source trace, promotion/canary/rollback, and an incident experiment that separates data from model/evaluator causes.

## 09 Required Evidence & Rubric

- **Provenance:** exact content and every transformation/input can be identified or the replay gap is explicit.
- **Validation:** each gate names invariant, blind spot, failure action, owner, and override lifecycle.
- **Quantitative discipline:** drift/dedup/label-efficiency equations have units, bins, populations, and uncertainty.
- **Independence:** splitting, fitting, dedup, contamination, and selection boundaries match deployment.
- **Feedback safety:** synthetic and active-learning records remain separable, auditable, and removable.
- **Diagnosis:** competing hypotheses are discriminated through release diffs, label audits, ablations, and remeasurement.

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

Pass requires the learner to replay a release, break schema-green data, distinguish shift hypotheses, expose leakage beyond exact matches, quantify dedup false merges, isolate synthetic ancestry, evaluate active learning on a representative holdout, trace pinned validation code, and defend promotion/rollback.

**Final mental model:** data engineering is controlled evidence transformation. Preserve identity and ancestry; validate progressively; keep evaluation independent; measure drift against consequences; and never let feedback data erase the distinction between observation and model-generated belief.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

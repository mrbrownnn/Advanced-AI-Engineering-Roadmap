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

**Research cutoff:** 2026-09-26. Source observations (**O**), derivations (**D**), and telemetry-dependent hypotheses (**H**) remain separate.

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
  falsification: REQUIRED
```

The learner must specify properties and oracles; build valid generators and shrinkers; test stateful protocols; justify metamorphic relations; use differential and mutation testing without oracle illusions; quantify stochastic evidence; separate adaptive discovery from estimation; inject bounded faults; operate a counterexample lifecycle; and trace a pinned property-testing engine.

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

Turn “the agent is robust” into observable contracts: schema is always valid; unauthorized effects never occur; adding irrelevant evidence should not change a cited answer; retry preserves one logical effect; cancellation prevents later effects; a locale transformation preserves a task-specific relation. Mark preconditions and acceptable nondeterminism.

CheckList's minimum-functionality, invariance, and directional-expectation tests are a useful behavioral vocabulary. Build a capability × test-type × risk-slice matrix, but do not treat filled cells as proof of completeness.

For generator support $S_G$ and valid domain $D$, a property test can falsify $P$ only on sampled $x\in S_G\cap D$. Passing says nothing about $D\setminus S_G$. Structural coverage records what was exercised, not whether the oracle checked it correctly.

**Outcome:** state a predicate precise enough to fail and humble enough not to overclaim.

### Lesson 16.2 — Property-Based and Stateful Testing

QuickCheck established the generator-plus-property pattern. In AI systems, generate structured prompts, schemas, tool responses, event streams, model outcomes, and action sequences—not arbitrary invalid bytes unless parser rejection is the property.

Measure generator support proxies: category/slice frequencies, boundary-value reach, filter/rejection rate, state/action transitions, and failure yield. Use constructive generators instead of filtering when possible. A shrinker must preserve domain validity and the same failure signature; “smallest” is relative to its representation and shrink order.

Stateful tests model commands, preconditions, transitions, invariants, and postconditions. Vary retries, cancellation, duplicate/out-of-order events, stale versions, timeout boundaries, and recovery. Record the whole action/observation/effect trace.

**Production source trace:** at Hypothesis revision `9c55f97e507eae21677457e84737b386fd06e271`, `given`/`find` and `RuleBasedStateMachine`/`run_state_machine_as_test` feed generated tests; `ConjectureRunner` drives exploration and `Shrinker` minimizes interesting examples. Static source inspection only.

**Outcome:** produce a valid, small, replayable counterexample rather than an untriageable random transcript.

### Lesson 16.3 — Behavioral and Metamorphic Relations

Metamorphic testing replaces a missing single-input oracle with two obligations:

$$R_i(x,x')\Rightarrow R_o(f(x),f(x')).$$

Both $x,x'$ must remain in-domain; $R_i$ must preserve or deliberately change the relevant semantics; $R_o$ must encode the expected invariant or direction. Examples include permutation invariance only when order is irrelevant, citation preservation under irrelevant distractors, monotonic recall under an enlarged eligible corpus, and equivariance under a semantics-preserving locale transform.

The 1998 technical report is the reference mechanism. A 2025 LLM study catalogs many NLP relations and manually checks reported violations, illustrating both usefulness and oracle risk. Never assume synonym replacement, paraphrase, formatting, answer-order swap, or added context is semantically neutral for every task.

For free-form outputs, separate transformation validity from output-relation grading. Use executable/domain checks where possible and audit any semantic judge independently.

**Outcome:** detect inconsistent behavior without inventing false invariants.

### Lesson 16.4 — Differential and Mutation Testing

Differential testing runs comparable implementations or revisions on the same input. DeepXplore demonstrates this family for neural systems. A disagreement locates a candidate boundary; adjudication decides whether A, B, both, or neither satisfy the contract. Independence matters: related models, shared retrieval, shared prompts, or the same judge can fail together.

Mutation testing injects controlled faults into code, configuration, prompt, policy, data, tool response, or workflow guard. With generated mutants $M$, killed mutants $K$, and adjudicated equivalent/invalid mutants $E$:

$$MS=\frac{K}{M-E}.$$

Report operator distribution, equivalence review, subsumption, execution cost, and which test killed which mutant. Mutation score measures sensitivity to selected fault classes—not production defect prevalence or correctness. Include realistic mutants such as removed authorization, wrong timeout unit, missing citation binding, stale version, dropped terminal event, retry after unknown commit, and changed system prompt.

**Outcome:** expose shared assumptions and measure whether the suite notices controlled breakage.

### Lesson 16.5 — Stochastic, Adaptive, and Fault-Injection Evidence

Define the trial unit: model sample, full request, trajectory, user/session, or environment seed. Preserve failed attempts. Under IID Bernoulli trials with fixed failure predicate and zero observed failures, the exact one-sided upper bound at confidence $1-\alpha$ is

$$p_U=1-\alpha^{1/n}.$$

This does not cover unseen prompts and fails under correlated retries, shifting systems, adaptive search, or hidden exclusions. Repetitions estimate conditional stochastic behavior; broader generators estimate input variation.

Fuzzers and red teams adapt toward weakness. Their finds are discovery evidence, not unbiased prevalence samples. Freeze promoted regressions and use a separate protected confirmatory sample or an analysis that models adaptive selection.

Fault injection names target boundary, fault, timing, expected invariant, blast radius, abort/cleanup, and recovery oracle. Inject timeout, malformed/partial response, cancellation race, quota, stale cache, duplicate event, tool failure, worker crash, or lost acknowledgement at controlled boundaries. Synthetic faults may miss correlated provider or regional incidents.

**Outcome:** quantify evidence without converting search success into population rates.

### Lesson 16.6 — Counterexample Lifecycle and Portfolio Economics

Every candidate moves through `VALIDATE → ADJUDICATE → MINIMIZE → DEDUPLICATE → ATTRIBUTE → FIX/ACCEPT → PROMOTE → RETIRE/REFRESH`. Preserve original and minimized cases, transformation/shrink trace, system/oracle versions, raw attempts, effect evidence, root cause, owner, severity, and replay status.

Cluster by causal signature, not surface text alone. A flaky failure remains a stochastic test with a measured reproduction rate; it is not silently deleted. Review obsolete fixtures when product requirements change.

Measure unique adjudicated failure clusters, severity, overlap by method, time/compute/judge/triage cost, false positives, shrink ratio, replay rate, suite latency, false blocks, and escaped incidents. The claim that a risk-weighted portfolio beats one coverage objective is **H**; test it against a preregistered baseline.

**Outcome:** turn discoveries into durable evidence without building a counterexample cemetery.

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

## 06 Engineering Labs

All labs follow `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`.

### LAB A — Generators, State Machines, and Shrinking

- Specify schema, authorization, cancellation, retry, and effect invariants for a tool-using loop.
- Implement structured generators and a rule-based state machine; pin/replay seeds and trace the Hypothesis execution path.
- Break with invalid overgeneration, heavy filtering, rare transitions, correlated choices, nondeterminism, and a shrinker that changes failure identity.
- Artifact: support/transition report, smallest valid counterexample, shrink trace, and explicit non-claims.

### LAB B — Metamorphic Oracle Audit

- Define at least five task-specific relations with preconditions and expected output relations.
- Include invariance, directional, equivariant, and deliberately invalid transformations.
- Validate transformations and output judgments independently; test prompt/model/judge versions and stochastic repetitions.
- Artifact: relation card, false-positive taxonomy, violation examples, and surviving-risk analysis.

### LAB C — Differential and Mutation Adequacy

- Cross two model versions, two backends, and two harness revisions on identical inputs; independently adjudicate disagreements.
- Mutate authorization, timeout, prompt, schema, retrieval evidence, event order, retry, and stopping logic.
- Measure killed/equivalent/invalid/subsumed mutants, common-mode misses, method overlap, cost, and latency.
- Artifact: differential matrix, mutation operator registry, kill matrix, and gaps ranked by hazard.

### LAB D — Stochastic Search and Fault Campaign

- Separate adaptive discovery corpus from a protected confirmatory sample and a workload sample.
- Inject boundary-specific transport, tool, state, and worker faults with abort and cleanup controls.
- Estimate conditional failure probabilities only where assumptions hold; preserve all attempts and effects.
- Run the full counterexample lifecycle and compare a risk-weighted portfolio against one coverage-maximizing baseline.
- Artifact: campaign manifest, fault matrix, counterexample ledger, cost/yield/escape report, and TODO_VERIFY list.

## 07 Break / Incident Scenarios

### Incident 16.1 — Ten Million Passing Tests, One Duplicate Payment

A property suite reports enormous case count and high transition coverage. Production later duplicates a payment after timeout. The test generator filtered unknown-commit states, the mock coupled acknowledgement and effect atomically, the shrinker removed the timeout race, differential targets shared the same mock, and the mutation set never altered acknowledgement ordering.

Competing explanations include invalid property, unreachable generator branch, correlated random choices, mock/production semantic mismatch, missing fault boundary, oracle checking response but not effect, retry-policy change, stale regression fixture, or telemetry loss. Recover generator distributions/rejections, seeds and shrink trace, state/action coverage, mock and provider contracts, attempt/effect IDs, fault timing, mutation kill matrix, system revisions, and production postconditions. Reproduce with a receiver-side effect oracle, add a targeted acknowledgement-loss fault and realistic mutant, fix the earliest violated boundary, promote the minimized trace, and remeasure reproduction, mutant kill, unique failure yield, and escaped incidents.

## 08 Mastery Assessment

Build a falsification program for a stochastic tool-using AI system. Deliver contracts and non-claims; property/stateful generators and valid shrinking; behavioral/metamorphic relations; differential adjudication; realistic mutation operators; IID-qualified stochastic bounds; adaptive discovery/confirmation separation; safe fault injection; a versioned counterexample ledger; pinned Hypothesis trace; portfolio cost/yield metrics; and an evidence-backed diagnosis of Incident 16.1.

## 09 Required Evidence & Rubric

- **Contract:** domain, predicate/relation, oracle, severity, budget, and non-claim are explicit.
- **Generation:** support, rejection, boundaries, transitions, seeds, and shrink validity are measured.
- **Oracles:** transformation validity, output relation, disagreement, and independent adjudication stay separate.
- **Adequacy:** mutation/coverage definitions and blind spots are reported without correctness claims.
- **Statistics:** trial unit, independence, denominator, adaptive selection, and confirmatory data are honest.
- **Fault safety:** boundary, blast radius, abort, cleanup, recovery, and postconditions are proven.
- **Lifecycle:** failures are reproducible, minimized, deduplicated, owned, promoted, and reviewed.
- **Economics:** unique consequential yield, overlap, triage cost, suite latency, and escapes drive the portfolio.

## 10 Capability Traceability Matrix

| Capability | Taught | Practiced | Assessed | Evidence |
|---|---|---|---|---|
| Falsification contract and oracle | 16.1 | LAB A–B | Incident / Mastery | Contract and non-claims |
| Property/stateful generation and shrinking | 16.2 | LAB A | Incident / Mastery | Support, trace, counterexample |
| Metamorphic/differential/mutation testing | 16.3–16.4 | LAB B–C | Mastery | Relation, adjudication, kill matrix |
| Stochastic/adaptive/fault evidence | 16.5 | LAB D | Incident / Mastery | Bounds, split, fault matrix |
| Counterexample lifecycle and portfolio | 16.6 | LAB D | Mastery | Ledger, yield/cost/escape report |

## 11 Exit Criteria & Module Wrap-Up

Pass requires a learner to falsify a meaningful system claim with a valid minimized case; show why the oracle and transformation apply; measure generator reach and stochastic assumptions; expose common-mode and mutation blind spots; inject one safe boundary fault; separate discovery from estimation; and demonstrate that promoted failures remain reproducible across a fix.

**Final mental model:** falsification engineering is disciplined search for disconfirming evidence. Its output is not a giant test count but a trustworthy counterexample—or a precisely bounded statement about where none was found.

## 12 Competency Targets

```yaml
competency:
  sfia: 5
  bloom: Evaluate -> Create
  solo: Relational -> Extended Abstract
  dreyfus: Competent
```

# Advanced AI Engineering — Instructional Schema

This document defines the generic instructional architecture governing all core curriculum modules. Do NOT encode domain-specific (e.g., KV-cache) knowledge here. This is the structural template.

## 1. FIRST-OCCURRENCE TEACHING RULE
Any concept, mechanism, abstraction, metric, or technical term that is required for the module's target mastery AND is not explicitly included in the curriculum's Baseline Assumptions MUST be taught when it first becomes necessary. 
Do not merely name it. Teach what it is, why it exists, how it works, its invariants, and its trade-offs.

## 2. SOURCE VERIFICATION RULE
Do not fabricate paper metadata, source paths, or execution paths. External reading must deepen understanding, not substitute for missing curriculum definitions. If a production source path or specific paper claim is not recently verified, it must be marked `TODO_VERIFY`.

## 3. MODULE SCHEMA (13 Sections)
Every module uses the section names and order of the golden reference, [Module 04](04-serving-scheduling-capacity/README.md). Section numbers are part of the heading (`## 00 Why This Module Exists`). Sections 00–02 are the module preamble; the three layer headings are `###` separators placed exactly where Module 04 places them.

```text
## 00 Why This Module Exists          (includes **Module Orientation**)
## 01 Baseline Assumptions
## 02 Target Mastery                  (depth contract + estimated_effort)
### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER
## 03 Knowledge Map
## 04 Lessons
## 05 Literature & Production Source Map
### LAYER 2: ENGINEERING PRACTICE LAYER
## 06 Engineering Labs
## 07 Break / Incident Scenarios
### LAYER 3: MASTERY / ASSESSMENT LAYER
## 08 Mastery Assessment
## 09 Required Evidence & Rubric
## 10 Capability Traceability Matrix
## 11 Exit Criteria & Module Wrap-Up
## 12 Competency Targets
```

- **00 Why This Module Exists**: the engineering problem; **Module Orientation** states the problem, what the learner will do, the environment, and the evidence rule.
- **01 Baseline Assumptions**: prerequisites that an earlier module actually teaches, cited by module and lesson. A prerequisite not taught earlier must be taught in this module (Section 1).
- **02 Target Mastery**: the 16-field depth contract and effort metadata (Section 4).
- **03 Knowledge Map**: the conceptual progression.
- **04 Lessons**: see the Lesson Schema (Section 5).
- **05 Literature & Production Source Map**: sources grouped as REFERENCE / BASELINE, CURRENT DEFAULT, WORKLOAD-DEPENDENT, FRONTIER, and LEGACY (see `agent.md` §4), each with scope, plus a pinned production source trace (repository, full commit, files/symbols, static vs executed). Frontier and optional extensions live here; there is no separate frontier section.
- **06 Engineering Labs**: hypothesis, independent and dependent variables, break/falsify condition, lesson alignment, effort. Synthetic data is labeled.
- **07 Break / Incident Scenarios**: symptoms, competing hypotheses, missing evidence, discriminating tests, diagnosis, intervention, remeasurement.
- **08 Mastery Assessment**: a transfer problem with explicit constraints and numbered deliverables.
- **09 Required Evidence & Rubric**: required artifacts (including the source trace when `source_code: REQUIRED`) and rubric dimensions with Insufficient / Competent / Strong levels.
- **10 Capability Traceability Matrix**: each capability → lesson taught → lab practiced → deliverable or incident step assessed → evidence artifact.
- **11 Exit Criteria & Module Wrap-Up**: observable capabilities and the final mental-model reconstruction.
- **12 Competency Targets**: SFIA, Bloom, SOLO, Dreyfus (see `COMPETENCY.md`).

## 4. EFFORT METADATA
Components must sum exactly to `total`; source-trace time is counted once even when a lesson and a lab share it. Lesson effort lines should sum to `instruction` and `guided_practice`, and lab estimates to `labs`; state any overlap explicitly. `tools/validate_curriculum.py` checks the component sum.
```yaml
estimated_effort:
  instruction: <hours>
  guided_practice: <hours>
  labs: <hours>
  assessment: <hours>
  source_trace: <hours>   # when source_code is REQUIRED
  total: <hours>
```
The schedule is planned in hours against the 26-week budget (see [ROADMAP.md](ROADMAP.md)); a module is not assumed to equal one week.

## 5. LESSON SCHEMA (The Scaffolding Progression)
Substantial lessons reduce scaffolding gradually (I DO → WE DO → YOU DO → TRANSFER).

Required in every lesson (as in Module 04):
- **Engineering Question**: The concrete problem the lesson answers.
- **Concepts & Definitions**: Precise boundaries between nearby concepts. Never left empty.
- **Mechanism Explanation**: State → operation → state transition.
- **Quantitative Model / Derivation** (or **Trade-off Comparison**): equations with variables, units, and assumptions, when the topic has a meaningful model. Do not force a meaningless equation.
- **Worked Example**: a solved example (I DO): input → intermediate steps → result → interpretation and limits. Numbers are recomputed; synthetic inputs are labeled. A problem left for the learner is practice, not a worked example.
- **Knowledge Check**: 2–4 formative questions testing reasoning.
- **Guided Practice**: a partially scaffolded problem (WE DO).
- **Feedback Contract**: expected evidence, a common failure, a diagnostic hint, and the concept to revisit.
- **Learning Outcome**: an observable, transferable capability.
- An effort line: `*(Effort: <instruction>, <practice>)*`.

Optional where they add information: **Mental Model**, **Independent Practice** (YOU DO), **Failure Modes / Counterexamples**, **Common Incorrect Mental Models**, **Production / Literature Connection**.

## 6. ENGINEERING LOOP
All major labs must follow this progression:
`Predict → Build → Measure → Break → Explain → Improve → Falsify → Defend`
Measurements without hypotheses and predictions are insufficient evidence.

## 7. ASSESSMENT MODEL
- **Formative Assessment**: Knowledge checks, guided practice. Purpose is to identify gaps, provide feedback, and retry.
- **Summative / Mastery Assessment**: Labs, incidents, engineering reports. Purpose is to demonstrate independent transfer of mental models under unfamiliar constraints.

## 8. EVIDENCE & RUBRIC MODEL
Assessments must be evaluated on engineering capability, not artifact completion. Each rubric dimension defines Insufficient, Competent, and Strong levels that two reviewers can apply consistently. Rubrics should evaluate dimensions like:
- **Mechanistic Reasoning**: Explains causal mechanisms (Competent) vs connects across boundaries (Strong).
- **Quantitative Reasoning**: Derives estimates (Competent) vs predicts and measures discrepancy (Strong).
- **Experiment Design**: Tests a hypothesis (Competent) vs discriminates between hypotheses (Strong).
- **Failure Diagnosis**: Forms multiple hypotheses (Competent) vs systematically eliminates them (Strong).
- **Falsification**: Tests counterexamples (Competent) vs constructs adversarial conditions (Strong).
- **Architecture Decision**: Explains trade-offs (Competent) vs states uncertainty and rollback conditions (Strong).
- **Source Reasoning**: Traces execution paths (Competent) vs connects implementation to system behavior (Strong).

## 9. MODULE WRAP-UP
A concise reconstruction of the final mental model, typically placed in Exit Criteria. It connects major concepts, identifies key failure modes, and bridges to the next module.

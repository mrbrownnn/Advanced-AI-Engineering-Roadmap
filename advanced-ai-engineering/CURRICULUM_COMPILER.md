# Curriculum Generation Contract

## Inputs
1. Module specification
2. Frozen evidence registry
3. Knowledge compilation output
4. Golden structural schema (`INSTRUCTIONAL_SCHEMA.md` & `03-kv-cache-engineering/README.md`)
5. Prerequisite/module dependency information

## Responsibilities
Generate the module curriculum (`README.md`).

The Curriculum Compiler MUST NOT:
- Perform open-ended research.
- Introduce external factual claims absent from the evidence registry.
(If missing evidence is needed, insert: `TODO VERIFY: <missing evidence>`).

## Output Sections
Generate the following sections. Do not allow the compiler contract and golden reference to disagree.
00 Module Orientation
01 Baseline Assumptions
02 Target Mastery
03 Knowledge Map
04 Lessons
05 Literature & Production Source Map
06 Engineering Labs
07 Break / Incident Scenarios
08 Mastery Assessment
09 Required Evidence & Rubric
10 Capability Traceability Matrix
11 Exit Criteria & Module Wrap-Up
12 Competency Targets
[Optional] Frontier / Optional

## Lesson Generation Contract
For major concepts, use the following progression where applicable:
I DO → WE DO → YOU DO → TRANSFER
A lesson must contain enough instructional structure to satisfy its Depth Contract and capability target. However, irrelevant fields may be marked `NOT_APPLICABLE` with a reason. Do not mechanically force every field. Prevent the template from becoming a content factory.
Typical fields (use applicability rules): Engineering Question, Concepts & Definitions, Mechanism, Quantitative Model, Worked Example, Knowledge Check, Guided Practice, Independent Practice, Feedback Contract, Learning Outcome.

## Lab & Assessment Contracts
- **Labs**: Use `PREDICT → BUILD → MEASURE → EXPLAIN → BREAK → IMPROVE → FALSIFY`. Define variables, workload, expected vs falsifying observations. No fabricated results.
- **Incident**: Do NOT reveal the root cause. Require hypotheses, rank, missing evidence, discriminating experiment, and intervention.
- **Mastery**: Must test TRANSFER to an unfamiliar scenario. Do not leak intended technologies.
- **Traceability Matrix**: A HARD GATE. `Capability | Taught | Practiced | Assessed | Evidence`. No capability may be assessed without being taught and practiced.
- **Depth Contract**: Must be executable. If `source_code: REQUIRED`, a source-trace artifact must exist.

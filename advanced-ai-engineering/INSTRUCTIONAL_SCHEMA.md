# Advanced AI Engineering — Instructional Schema

This document defines the generic instructional architecture governing all core curriculum modules. Do NOT encode domain-specific (e.g., KV-cache) knowledge here. This is the structural template.

## 1. FIRST-OCCURRENCE TEACHING RULE
Any concept, mechanism, abstraction, metric, or technical term that is required for the module's target mastery AND is not explicitly included in the curriculum's Baseline Assumptions MUST be taught when it first becomes necessary. 
Do not merely name it. Teach what it is, why it exists, how it works, its invariants, and its trade-offs.

## 2. SOURCE VERIFICATION RULE
Do not fabricate paper metadata, source paths, or execution paths. External reading must deepen understanding, not substitute for missing curriculum definitions. If a production source path or specific paper claim is not recently verified, it must be marked `TODO VERIFY`.

## 3. MODULE SCHEMA (13 Sections)
Every module must strictly adhere to the following 13-section interface, divided into 3 layers:

### LAYER 1: KNOWLEDGE / INSTRUCTIONAL LAYER
- **00 Why This Module Exists**: The engineering problem addressed.
- **01 Baseline Assumptions**: Explicit prerequisites.
- **02 Target Mastery**: The Depth Contract and Effort Metadata.
- **03 Knowledge Map**: The conceptual progression of the module.
- **04 Lessons**: The core instructional content (see Lesson Schema).
- **05 Literature & Production Source Map**: Canonical, Production, and Frontier references.

### LAYER 2: ENGINEERING PRACTICE LAYER
- **06 Engineering Labs**: Integrated labs following the Engineering Loop.
- **07 Break / Incident Scenarios**: Ambiguous production failures.

### LAYER 3: MASTERY / ASSESSMENT LAYER
- **08 Mastery Assessment**: Unfamiliar transfer problems.
- **09 Required Evidence**: Artifacts and Rubric.
- **10 Exit Criteria**: Observable capabilities and Module Wrap-up.
- **11 Competency Targets**: SFIA, Bloom, SOLO mappings.
- **12 Frontier / Optional**: Specialist extensions.

## 4. EFFORT METADATA
Include estimated workload to ensure the module fits the 26-week (730-780 hour) global curriculum budget.
```yaml
estimated_effort:
  instruction: <hours>
  guided_practice: <hours>
  labs: <hours>
  assessment: <hours>
  total: <hours>
```

## 5. LESSON SCHEMA (The Scaffolding Progression)
Substantial lessons must reduce scaffolding gradually (I DO → WE DO → YOU DO → TRANSFER).
- **Engineering Question**: The concrete problem the lesson answers.
- **Concepts & Definitions**: Precise boundaries between nearby concepts.
- **Mental Model**: A compact conceptual representation supporting prediction.
- **Mechanism Explanation**: State → operation → state transition.
- **Quantitative Model / Derivation**: Derived equations and assumptions.
- **Worked Example**: A concrete, internally validated numerical example (I DO).
- **Knowledge Check**: 2-4 formative questions testing reasoning.
- **Guided Practice**: Partially scaffolded problem (WE DO).
- **Independent Practice**: Unscaffolded related problem (YOU DO).
- **Feedback Contract**: Expected evidence, diagnostic hints, and concepts to revisit.
- **Failure Modes / Counterexamples**: When the mechanism fails or degrades.
- **Common Incorrect Mental Models**: Explicitly identified misconceptions.
- **Production / Literature Connection**: Verified links to source code/papers.
- **Learning Outcome**: An observable, transferable capability.

## 6. ENGINEERING LOOP
All major labs must follow this progression:
`Predict → Build → Measure → Break → Explain → Improve → Falsify → Defend`
Measurements without hypotheses and predictions are insufficient evidence.

## 7. ASSESSMENT MODEL
- **Formative Assessment**: Knowledge checks, guided practice. Purpose is to identify gaps, provide feedback, and retry.
- **Summative / Mastery Assessment**: Labs, incidents, engineering reports. Purpose is to demonstrate independent transfer of mental models under unfamiliar constraints.

## 8. EVIDENCE & RUBRIC MODEL
Assessments must be evaluated on engineering capability, not artifact completion. Rubrics should evaluate dimensions like:
- **Mechanistic Reasoning**: Explains causal mechanisms (Competent) vs connects across boundaries (Strong).
- **Quantitative Reasoning**: Derives estimates (Competent) vs predicts and measures discrepancy (Strong).
- **Experiment Design**: Tests a hypothesis (Competent) vs discriminates between hypotheses (Strong).
- **Failure Diagnosis**: Forms multiple hypotheses (Competent) vs systematically eliminates them (Strong).
- **Falsification**: Tests counterexamples (Competent) vs constructs adversarial conditions (Strong).
- **Architecture Decision**: Explains trade-offs (Competent) vs states uncertainty and rollback conditions (Strong).
- **Source Reasoning**: Traces execution paths (Competent) vs connects implementation to system behavior (Strong).

## 9. MODULE WRAP-UP
A concise reconstruction of the final mental model, typically placed in Exit Criteria. It connects major concepts, identifies key failure modes, and bridges to the next module.

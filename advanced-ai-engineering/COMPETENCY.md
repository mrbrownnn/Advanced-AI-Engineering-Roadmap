# Competency Framework

## Purpose

Every module checkpoint and major exercise carries competency metadata. This metadata describes the **type of technical responsibility being practiced**, not a certification or credential.

> ⚠️ **Important**: Course completion does not automatically grant an SFIA level. SFIA metadata represents the technical responsibility level practiced in each exercise.

## Target Progression

| Entry | Target | Stretch |
|-------|--------|---------|
| Junior+ / SFIA 4 | Strong Middle / SFIA 5 | Selected SFIA 6 exercises |

## Metadata Format

Every checkpoint should include a competency block:

```yaml
competency:
  sfia: 5
  bloom: Evaluate
  solo: Relational
  dreyfus: Competent
```

---

## Framework Definitions

### SFIA (Skills Framework for the Information Age)

| Level | Responsibility | In This Course |
|-------|---------------|----------------|
| 4 | Enable — works under general direction, exercises substantial personal responsibility | Entry baseline: can implement given a clear specification |
| 5 | Ensure, Advise — broad direction, accountable for technical decisions, influences organization | Target: independently diagnoses, decides, and defends engineering trade-offs |
| 6 | Initiate, Influence — defined authority, accountable for actions and decisions of others | Stretch: designs systems under ambiguous constraints, mentors others |

### Bloom's Revised Taxonomy

| Level | Verb | In This Course |
|-------|------|----------------|
| Remember | Recall | Identify components, name concepts |
| Understand | Explain | Describe how PagedAttention works |
| Apply | Use | Implement a component given documentation |
| Analyze | Differentiate | Profile and identify bottlenecks |
| Evaluate | Judge | Compare strategies, select based on evidence |
| Create | Design | Architect a system under novel constraints |

### SOLO Taxonomy (Structure of Observed Learning Outcomes)

| Level | Description | In This Course |
|-------|-------------|----------------|
| Prestructural | Missing the point | — |
| Unistructural | One relevant aspect | Can state what KV cache is |
| Multistructural | Several independent aspects | Can list KV cache strategies |
| Relational | Integrated understanding | Can explain how KV cache interacts with scheduling and memory |
| Extended Abstract | Generalizes to new domains | Can predict KV behavior for an unseen architecture |

### Dreyfus Skill Acquisition Model

| Level | Description | In This Course |
|-------|-------------|----------------|
| Novice | Follows rules | Can follow a tutorial |
| Advanced Beginner | Recognizes aspects | Can identify relevant factors in a problem |
| Competent | Prioritizes, plans | Can independently diagnose and fix a performance issue |
| Proficient | Sees the whole picture | Can anticipate problems from architecture choices |
| Expert | Intuitive | Can design systems in novel domains — stretch only |

---

## Checkpoint Progression Example

The same topic at different competency levels:

**SFIA 4 / Bloom: Apply / SOLO: Multistructural / Dreyfus: Advanced Beginner**
> "Implement prefix caching for a given workload."

**SFIA 5 / Bloom: Evaluate / SOLO: Relational / Dreyfus: Competent**
> "Prefix caching performs poorly under this workload. Diagnose the root cause, propose alternatives, and validate experimentally."

**SFIA 6 / Bloom: Create / SOLO: Extended Abstract / Dreyfus: Proficient**
> "Given this workload, SLO, hardware budget, and expected growth, decide whether prefix caching belongs in the architecture and defend the decision with quantitative evidence."

---

## Usage

1. Each module's `checkpoint.md` includes a `competency:` block
2. Exercises are tagged so learners can self-assess against the frameworks
3. The capstone and graduation require SFIA 5 across multiple domains
4. SFIA 6 exercises are explicitly marked as stretch

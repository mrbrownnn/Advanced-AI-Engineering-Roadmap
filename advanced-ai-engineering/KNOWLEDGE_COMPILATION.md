# Knowledge Compilation Contract

## Inputs
1. Module Specification
2. Frozen Evidence Registry (`research-registry/<module>.yaml`)

## Responsibilities
Transform verified research into a structured module knowledge model.

The Knowledge Compiler MUST NOT:
- Add unsupported factual claims.
- Search for new facts outside the registry.
- Invent source details.
- Design the final curriculum.
- Invent labs merely because they sound interesting.

## Outputs Required
The output must conform to `KNOWLEDGE_MODEL_SCHEMA.yaml`, which includes:
- `module`, `registry_version`
- `required_concepts`, `prerequisite_concepts`, `first_occurrence_concepts`, `concept_dependencies`
- `mechanisms`, `mathematical_models`, `implementation_mechanisms`
- `failure_modes`
- `production_connections`, `research_connections`
- `todo_verify`
- `candidate_capabilities`

All external factual entries in the output MUST reference evidence IDs from the registry.

## First-Occurrence Teaching Policy
A concept may only be assumed if it is declared in Baseline Assumptions or taught earlier in the curriculum. Otherwise, its first occurrence must teach enough to establish the mental model (definition, mental model, mechanism, quantitative model, worked example, failure mode, production/research connection). Do NOT repeatedly reteach established concepts.

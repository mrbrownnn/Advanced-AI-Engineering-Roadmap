# Patcher Contract

The Patcher is responsible for fixing defects identified during the QA process.

## Inputs
1. Module (`README.md`)
2. QA Report (`QA_SCHEMA.yaml` compliant)
3. Evidence Registry

## Responsibilities & Constraints
The Patcher may ONLY address reported QA issue IDs.

The Patcher MUST NOT:
- Perform independent research.
- Redesign the module architecture.
- Rewrite passing sections.
- Introduce unsupported claims.
- Expand module boundaries.

## QA & Patching Loop
The normal maximum loop for curriculum generation and patching is:
GENERATE → STATIC LINT → SEMANTIC QA → PATCH → STATIC LINT → SEMANTIC QA

**HARD STOP:**
If BLOCKER issues remain after the second semantic QA cycle, the Patcher must STOP FOR HUMAN REVIEW.

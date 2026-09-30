# Static Linter Contract

This document defines the STATIC / DETERMINISTIC VALIDATION boundary.
The static linter must NOT claim to validate technical truth or semantic correctness. 
Semantic and technical correctness are validated by the Semantic QA process.

## Minimum Static Checks

1. **Structure**
   - Required structural sections exist (00 through 12).
   - Valid Depth Contract exists.
   - Traceability matrix exists.
   - Competency metadata exists.
   
2. **References & Consistency**
   - References to existing lessons/labs are valid.
   - Effort arithmetic reconciles (sum of components == total).

3. **Provenance**
   - Provenance enum validity.
   - Evidence ID resolution (IDs used must exist in registry).
   - `TODO_VERIFY` markers are visible and explicitly marked.
   - VERIFIED source-code claims require a pinned revision (commit/SHA).
   - Registry version compatibility.

4. **Assessment & Traceability**
   - Required source-trace artifact exists when `source_code: REQUIRED`.
   - No empty evidence field for REQUIRED capability in the traceability matrix.

## Implementation

`tools/validate_curriculum.py` implements the schema, provenance, cross-file ID, lesson/lab target, O/D/H label, concept-dependency, H2-spine, and effort-total checks for every module that has a registry and authored lessons. Negative fixtures in `tools/fixtures/` (`bad_*`) must be rejected and positive fixtures accepted; this keeps the schema holes for VERIFIED_FACT claims without evidence, unpinned source-code evidence, and empty published knowledge models closed. A knowledge model may declare `status: draft` to opt out of the non-empty requirement while it is being written.

Run from `advanced-ai-engineering/`: `python tools/validate_curriculum.py [module-number ...]`. Exit code 0 means no static errors; it does not certify technical correctness.

# Schema Review

## Purpose
Following the pilot depth upgrade of 8 representative modules, this document evaluates the applied curriculum schema before scaling it to the rest of the repository (Phase 5 and 6).

## Schema Implemented
The new schema replaces the static `Topics` lists with the `Engineering Mastery Loop`:
1. **BUILD** (Implementation)
2. **MEASURE** (Quantitative Reasoning)
3. **BREAK** (Falsification & Failure)
4. **EXPLAIN** (Diagnosis)
5. **IMPROVE** (Optimization)
6. **DEFEND** (Production Trade-offs)

Additionally, explicit `Source-Code Reading` requirements have been integrated into every module.

## Review Findings

1. **Technical Depth**: The shift from "understanding" to "building and breaking" dramatically raises the depth. By forcing learners to intentionally cause OOMs or fragment memory, we ensure the knowledge is operational.
2. **Duplication**: The new schema strictly isolates the failure modes (e.g., separating harness grammar failures in Module 13 from agent infinite loops in Module 12).
3. **Workload**: The workload per module has increased significantly. Building mechanisms from scratch (e.g., PagedAttention) takes time. We must ensure that the remaining modules keep the "minimal implementation" scope strictly contained.
4. **Consistency**: The 6-step loop applies universally well to both Systems modules (e.g., KV Cache) and Application modules (e.g., Evaluation Engineering).
5. **Specialist Scope Creep**: The schema naturally bounds the scope. Because the learner must complete the loop and defend a decision, they cannot wander off into endless theoretical reading.
6. **Dependency Correctness**: The sequence holds up. You cannot break a KV cache (Module 03) without understanding token mechanics (Module 01).
7. **Production Relevance**: The `DEFEND` step perfectly replicates real-world engineering review meetings (e.g., choosing between FP16 and INT8 based on empirical quality metrics).

## Conclusion
The schema is validated. It perfectly operationalizes the Depth Contract. We are clear to proceed with Phase 5 (Core Depth Pass) and Phase 6 (Satellite Depth Pass) using this exact template.

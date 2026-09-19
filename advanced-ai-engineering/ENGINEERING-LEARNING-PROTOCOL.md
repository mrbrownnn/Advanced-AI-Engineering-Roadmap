# Engineering Learning Protocol

## Purpose
This protocol defines the standard methodology for how an Advanced AI Engineer learns and masters new concepts within this curriculum. 

We do not simply "read and complete exercises". We execute rigorous engineering loops.

## The Engineering Mastery Loop

For any core architectural concept or system component, learning proceeds through the following loop:

### 1. BUILD
Implement the mechanism. This means writing the code from scratch or assembling the system components, deliberately avoiding high-level abstractions that hide the underlying mechanics.

### 2. MEASURE
Instrument the system. Establish a baseline metric. This must include quantitative reasoning: deriving bounds, predicting behavior, and rigorous benchmarking (e.g., measuring p99 latency under load).

### 3. BREAK
Create a workload specifically designed to invalidate the system's assumptions or exceed its capacity. Push the mechanism until it fails, fragments memory, OOMs, or hallucinates.

### 4. EXPLAIN
Diagnose the failure. Formulate a falsifiable hypothesis explaining exactly why the system broke at that specific point.

### 5. IMPROVE
Apply an optimization, adaptation, or architectural change based on the evidence collected during the failure analysis.

### 6. DEFEND
Present the final engineering decision (the trade-off chosen). Defend this decision with empirical evidence, comparing alternatives and acknowledging remaining uncertainties.

---

## The Literature Loop

For reading academic papers and source code, learning proceeds through the following loop:

### 1. READ
Identify the core mechanism, the assumptions made, and the claimed metrics.

### 2. REPRODUCE
Implement a minimal version of the paper's core mechanism or trace the mechanism in an existing open-source repository (e.g., vLLM, transformers).

### 3. CHALLENGE
Identify the limitations of the paper. Where would this approach fail in production? What are the hidden costs (latency, memory, complexity)?

### 4. CONNECT
Map the mechanism to neighboring subsystems. How does this paper's attention variant affect the downstream scheduler's batching strategy?

### 5. APPLY
Integrate the insights into the current engineering project or experiment report.

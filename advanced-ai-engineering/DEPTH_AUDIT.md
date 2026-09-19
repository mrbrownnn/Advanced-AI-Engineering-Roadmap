# Phase 1: Core Curriculum Depth Audit

This audit evaluates the 25 core modules to transition them from a 'structured list of topics' to a 'rigorous mastery-oriented curriculum'.

## 00-scientific-ai-engineering

- **Current Purpose**: Defines concepts and topics related to scientific ai engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 01-foundation-model-internals

- **Current Purpose**: Defines concepts and topics related to foundation model internals.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 02-inference-gpu-fundamentals

- **Current Purpose**: Defines concepts and topics related to inference gpu fundamentals.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding the hardware/software boundary.
- **Missing Mechanisms**: CUDA/Memory interactions.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading vLLM, transformers, or NCCL source code.
- **Missing Instrumentation**: Using PyTorch Profiler or Nsight Systems.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 03-kv-cache-engineering

- **Current Purpose**: Defines concepts and topics related to kv cache engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding the hardware/software boundary.
- **Missing Mechanisms**: CUDA/Memory interactions.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading vLLM, transformers, or NCCL source code.
- **Missing Instrumentation**: Using PyTorch Profiler or Nsight Systems.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `KEEP`

## 04-serving-scheduling-capacity

- **Current Purpose**: Defines concepts and topics related to serving scheduling capacity.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding the hardware/software boundary.
- **Missing Mechanisms**: CUDA/Memory interactions.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading vLLM, transformers, or NCCL source code.
- **Missing Instrumentation**: Using PyTorch Profiler or Nsight Systems.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 05-inference-optimization

- **Current Purpose**: Defines concepts and topics related to inference optimization.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding the hardware/software boundary.
- **Missing Mechanisms**: CUDA/Memory interactions.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading vLLM, transformers, or NCCL source code.
- **Missing Instrumentation**: Using PyTorch Profiler or Nsight Systems.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `RESTRUCTURE`

## 06-reasoning-test-time-compute

- **Current Purpose**: Defines concepts and topics related to reasoning test time compute.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 07-model-behavior-uncertainty

- **Current Purpose**: Defines concepts and topics related to model behavior uncertainty.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `RESTRUCTURE`

## 08-ai-data-engineering

- **Current Purpose**: Defines concepts and topics related to ai data engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `MOVE_TO_OPTIONAL`

## 09-retrieval-engineering

- **Current Purpose**: Defines concepts and topics related to retrieval engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 10-advanced-rag

- **Current Purpose**: Defines concepts and topics related to advanced rag.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `MERGE`

## 11-context-memory-engineering

- **Current Purpose**: Defines concepts and topics related to context memory engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `RESTRUCTURE`

## 12-agent-loop-engineering

- **Current Purpose**: Defines concepts and topics related to agent loop engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 13-harness-engineering

- **Current Purpose**: Defines concepts and topics related to harness engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `KEEP`

## 14-durable-agent-runtime

- **Current Purpose**: Defines concepts and topics related to durable agent runtime.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 15-evaluation-engineering

- **Current Purpose**: Defines concepts and topics related to evaluation engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 16-falsification-engineering

- **Current Purpose**: Defines concepts and topics related to falsification engineering.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `KEEP`

## 17-harness-evolution

- **Current Purpose**: Defines concepts and topics related to harness evolution.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `RESTRUCTURE`

## 18-ai-security

- **Current Purpose**: Defines concepts and topics related to ai security.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `MOVE_TO_OPTIONAL`

## 19-model-adaptation

- **Current Purpose**: Defines concepts and topics related to model adaptation.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 20-distributed-inference

- **Current Purpose**: Defines concepts and topics related to distributed inference.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding the hardware/software boundary.
- **Missing Mechanisms**: CUDA/Memory interactions.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading vLLM, transformers, or NCCL source code.
- **Missing Instrumentation**: Using PyTorch Profiler or Nsight Systems.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `DEEPEN`

## 21-multimodal-ai-systems

- **Current Purpose**: Defines concepts and topics related to multimodal ai systems.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `MOVE_TO_OPTIONAL`

## 22-ai-economics

- **Current Purpose**: Defines concepts and topics related to ai economics.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `KEEP`

## 23-observability-reliability

- **Current Purpose**: Defines concepts and topics related to observability reliability.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `MERGE`

## 24-model-system-codesign

- **Current Purpose**: Defines concepts and topics related to model system codesign.
- **Existing Strengths**: Good high-level topic coverage and theoretical questions.
- **Current Technical Depth**: Currently tutorial-level. Lists concepts but doesn't require building from scratch or breaking.
- **Baseline Material (Do Not Dominate)**: Basic definitions, API usage, or superficial tutorials.
- **Shallow Sections**: Lacks deep dive into extreme edge cases or hardware-level limitations.
- **Missing Mental Models**: Understanding non-deterministic failure boundaries.
- **Missing Mechanisms**: Internal state representations.
- **Missing Quantitative Reasoning**: Calculating FLOPs, memory bandwidth, or token throughput bounds.
- **Missing Implementation Depth**: Building a minimal version from scratch instead of using a library.
- **Missing Source-Code Reading**: Reading LangChain/LlamaIndex source code to see how abstractions fail.
- **Missing Instrumentation**: Setting up OpenTelemetry and distributed tracing.
- **Missing Experiments**: Ablation studies on specific optimizations.
- **Missing Benchmark Rigor**: Measuring p99 latency under varying load, not just p50.
- **Missing Failure Analysis**: Analyzing what happens when memory is exhausted or inputs are malformed.
- **Missing Falsification**: Creating a test designed to intentionally break the system's assumptions.
- **Missing Architecture Reasoning**: Trade-offs between different system designs in production.
- **Missing Production Connections**: Linking this module to real production incidents and scale.
- **Duplicated Content**: Potential overlap with adjacent modules.
- **Dependency Problems**: Internal module numbering in the README is incorrect (e.g. says Module X but folder is Y).
- **Scope-Creep Risks**: Risk of becoming a general tutorial rather than an advanced engineering task.
- **Recommended Changes**: `KEEP`


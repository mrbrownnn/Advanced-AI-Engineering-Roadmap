# Agents — Paper Index

## ReAct: Synergizing Reasoning and Acting in Language Models

Authors: Yao, Zhao, Yu, Du, Shafran, Narasimhan, Cao
Year: 2023
URL: https://arxiv.org/abs/2210.03629
Category: Agent Architectures
Priority: MUST

Why read:
Establishes the reasoning + acting loop that is the foundation of most LLM agent frameworks. Understanding ReAct is essential for reasoning about agent execution patterns, failure modes, and the design of durable agent runtimes.

Read specifically:
- Section 3 — the ReAct formulation (interleaving thought and action)
- Examples of reasoning traces
- Failure analysis — where and why ReAct fails

Questions:
1. How does interleaving reasoning with actions improve task completion over action-only agents?
2. What are the failure modes of the ReAct pattern?
3. How does ReAct relate to the execution graph / state machine models in Module 11?

Related module: 11-durable-agent-runtime

---

## Toolformer: Language Models Can Teach Themselves to Use Tools

Authors: Schick, Dwivedi-Yu, Dessì, Raber, Zamber, Lomeli, Zettlemoyer, Cancedda, Hofmann
Year: 2023
URL: https://arxiv.org/abs/2302.04761
Category: Tool Use
Priority: SHOULD

Why read:
Demonstrates self-supervised tool use learning. Important for understanding the capabilities and limitations of tool-augmented LLMs, and for reasoning about when tool use is appropriate vs when the model should rely on its own parameters.

Read specifically:
- Section 3 — the self-supervised approach to learning tool calls
- Which tools benefit most from this approach
- Limitations and failure cases

Questions:
1. How does the model learn when to invoke a tool vs answer directly?
2. What types of tools are most effectively learned?
3. How does this relate to the tool evaluation challenges in Module 12?

Related module: 11-durable-agent-runtime, 12-evaluation-engineering

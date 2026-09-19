# Post-Training — Paper Index

## LoRA: Low-Rank Adaptation of Large Language Models

Authors: Hu, Shen, Wallis, Allen-Zhu, Li, Wang, Wang, Chen
Year: 2022
URL: https://arxiv.org/abs/2106.09685
Category: Parameter-Efficient Fine-Tuning
Priority: MUST

Why read:
Introduces low-rank adaptation: freezing pretrained weights and training small rank-decomposed matrices. This is the dominant PEFT method. Essential for understanding when and how to adapt models, and for reasoning about the LoRA-vs-prompting-vs-retrieval decision space.

Read specifically:
- Section 3 — the low-rank parameterization
- Section 4 — which weight matrices to adapt
- Rank selection and its impact on quality
- Section 7 — understanding the low-rank updates

Questions:
1. Why does low-rank adaptation work? What does this imply about the structure of task-specific updates?
2. How does rank selection affect quality and training cost?
3. Which weight matrices benefit most from LoRA adaptation?

Related module: 15-model-adaptation

---

## QLoRA: Efficient Finetuning of Quantized LLMs

Authors: Dettmers, Pagnoni, Holtzman, Zettlemoyer
Year: 2023
URL: https://arxiv.org/abs/2305.14314
Category: Parameter-Efficient Fine-Tuning
Priority: MUST

Why read:
Combines 4-bit quantization with LoRA to enable fine-tuning of large models on consumer GPUs. Introduces NormalFloat4 and double quantization. Important for understanding the practical economics of model adaptation.

Read specifically:
- Section 3 — NormalFloat4 data type
- Double quantization
- Paged optimizers for memory management
- Quality comparisons with full fine-tuning and standard LoRA

Questions:
1. How does 4-bit quantization of the base model affect LoRA training quality?
2. What is NormalFloat4 and why is it better than standard INT4 for this application?
3. What are the memory savings? When is QLoRA sufficient vs when do you need full LoRA?

Related module: 15-model-adaptation

---

## Direct Preference Optimization: Your Language Model is Secretly a Reward Model

Authors: Rafailov, Sharma, Mitchell, Manning, Ermon, Finn
Year: 2023
URL: https://arxiv.org/abs/2305.18290
Category: Preference Optimization
Priority: MUST

Why read:
Simplifies RLHF by eliminating the explicit reward model. Shows that the optimal policy can be extracted directly from preference data using a classification loss. Critical for understanding modern alignment and preference optimization pipelines.

Read specifically:
- Section 3 — derivation of the DPO objective from the RLHF objective
- The implicit reward model interpretation
- Comparison with PPO-based RLHF in terms of quality and stability

Questions:
1. How does DPO avoid training a separate reward model?
2. What assumptions does DPO make about the preference data?
3. When might PPO-based RLHF be preferable to DPO?

Related module: 15-model-adaptation

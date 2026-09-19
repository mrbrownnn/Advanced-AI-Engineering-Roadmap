# 06 — VLM Evaluation

## Purpose

Understand evaluation frameworks, benchmarks, and error taxonomies specific to vision-language models.

## Topics

- Multimodal hallucination: object hallucination, attribute binding errors, spatial relationship errors
- Benchmarks: POPE (object presence), MME, MMBench, DocVQA, MathVista
- Automated evaluation: LLM-as-judge for multimodal tasks
- Human evaluation protocols for perceptual accuracy
- Distinguishing vision encoder failures from language generator hallucinations

## Key Questions

- What causes a VLM to hallucinate non-existent objects when shown an image?
- How does the POPE benchmark isolate hallucination from language prior bias?
- How do we debug whether an error stems from the vision encoder, projector, or language model?

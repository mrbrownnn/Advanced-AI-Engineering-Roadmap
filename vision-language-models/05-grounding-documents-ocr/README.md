# 05 — Grounding, Documents, OCR

## Purpose

Understand fine-grained spatial grounding, bounding box prediction, and document understanding without dedicated OCR pipelines.

## Topics

- Coordinate representation: normalized coordinates as text tokens (`[ymin, xmin, ymax, xmax]`)
- Visual grounding and referring expression comprehension
- Document layout understanding: receipts, tables, forms, infographics
- OCR-free document parsing vs pipeline architectures (OCR engine + LLM)
- High-resolution processing for fine text legibility

## Key Questions

- How do language models represent and generate continuous 2D spatial coordinates?
- Under what conditions does an end-to-end VLM outperform an OCR + RAG pipeline?
- How does high-resolution document processing challenge standard token context limits?

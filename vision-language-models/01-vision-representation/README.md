# 01 — Vision Representation

## Purpose

Understand how continuous 2D images are transformed into discrete or tokenized representations suitable for transformer backbones.

## Topics

- Image patchification: splitting images into non-overlapping grid patches (e.g. 14x14, 16x16)
- Vision Transformer (ViT) architecture: patch projection, position embeddings, self-attention
- Resolution strategies: fixed resolution vs dynamic patching/tiling (AnyRes)
- Feature extraction: intermediate layer activations vs final pooling
- Computational complexity: quadratic scaling with image resolution

## Key Questions

- Why do vision transformers tokenize patches rather than individual pixels?
- How does image resolution affect token count and attention complexity?
- What are the trade-offs of fixed-size image resizing versus dynamic multi-crop tiling?

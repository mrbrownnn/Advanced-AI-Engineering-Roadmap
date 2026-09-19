# 02 — Speech-to-Text

## Purpose

Understand Automated Speech Recognition (ASR) model architectures, alignment loss functions, and diarization systems.

## Topics

- ASR Architectures: Connectionist Temporal Classification (CTC), RNN-Transducer (RNN-T), Encoder-Decoder (Whisper)
- Cross-attention speech decoding and autoregressive transcript generation
- Timestamp estimation and word-level alignment
- Multilingual ASR and translation capabilities
- Speaker diarization pipelines: voice activity detection (VAD), speaker embedding extraction, clustering

## Key Questions

- When is CTC or RNN-T preferred over autoregressive encoder-decoder architectures?
- How does Whisper handle multi-task conditioning (transcribe, translate, timestamps) via prefix prompting?
- What are the common failure modes of ASR models in noisy or accented environments?

# 05 — Speech Evaluation

## Purpose

Understand quantitative evaluation metrics and benchmarks for speech recognition accuracy and synthetic speech quality.

## Topics

- ASR metrics: Word Error Rate (WER), Character Error Rate (CER), handling text normalization/punctuation
- TTS metrics: Mean Opinion Score (MOS), UTMOS, speaker similarity (cosine similarity on speaker embeddings)
- Audio quality metrics: PESQ, POLQA, MCD (Mel-Cepstral Distortion)
- Latency and performance benchmarks: Real-Time Factor (RTF), latency percentiles (p50, p95, p99)
- Robustness evaluation across accents, reverberation, and signal-to-noise ratios (SNR)

## Key Questions

- Why can WER be misleading without standardized text normalization?
- How do automated MOS predictors (like UTMOS) compare with human listening tests?
- What test suites reveal catastrophic edge cases in production voice applications?

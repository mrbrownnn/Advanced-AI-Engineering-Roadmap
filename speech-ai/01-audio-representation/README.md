# 01 — Audio Representation

## Purpose

Understand audio signal processing fundamentals, time-frequency transforms, and modern learned audio tokenizers.

## Topics

- Continuous waveforms: sampling rates (16kHz, 24kHz, 44.1kHz), bit depth, quantization
- Fourier Transform and Short-Time Fourier Transform (STFT)
- Mel spectrograms and log-mel filterbanks
- Neural audio codecs and tokenizers (EnCodec, DAC, SoundStream)
- Residual Vector Quantization (RVQ) for audio discretization

## Key Questions

- Why are mel spectrograms preferred over raw waveforms for speech recognition backbones?
- How does RVQ compress high-bandwidth continuous audio into discrete token streams?
- What trade-offs exist between neural codec bitrates and perceptual audio reconstruction quality?

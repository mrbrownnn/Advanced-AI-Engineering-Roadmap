# Speech AI — 80/20 Architectural Literacy

An architectural literacy track covering speech AI. The goal is to understand audio representation, ASR, TTS, and voice agent systems well enough to read papers, evaluate production systems, and identify where the domain connects to the core AI engineering stack.

## Five Fundamental Questions

1. **Representation**: Waveform → sampling → STFT → spectrogram → mel features → learned representations
2. **Architecture**: Encoder-decoder ASR (Whisper), CTC/RNN-T, autoregressive/diffusion/flow TTS, neural vocoders
3. **Learning**: CTC loss, sequence-to-sequence, diffusion denoising, flow matching, speaker conditioning
4. **Evaluation**: WER, CER, MOS, real-time factor, latency metrics
5. **Production**: Streaming, VAD, real-time factor, time-to-first-audio, end-to-end turn latency, cost

## Modules

| Module | Focus |
|--------|-------|
| [01 — Audio Representation](01-audio-representation/) | Waveforms, spectrograms, mel features |
| [02 — Speech-to-Text](02-speech-to-text/) | ASR architectures, Whisper, streaming, diarization |
| [03 — Text-to-Speech](03-text-to-speech/) | TTS pipeline, vocoders, diffusion/flow TTS |
| [04 — Streaming Speech](04-streaming-speech/) | Real-time ASR/TTS, buffering, latency |
| [05 — Speech Evaluation](05-speech-evaluation/) | WER, MOS, RTF, latency metrics |
| [06 — Voice Agent Systems](06-voice-agent-systems/) | STT → LLM → TTS pipeline, turn latency |

## Connection to Core Track

```
Speech → streaming (Module 04)
       → real-time agents (Module 12, 14)
       → latency engineering (Module 02)
       → queueing (Module 04)
       → cost (Module 22)
       → multimodal systems (Module 21)
```

## Recommended Timing

Month 3, alongside core Modules 06–08.

## Competency Target

```yaml
competency:
  bloom: Understand → Apply → Analyze
  solo: Multistructural → Relational
  dreyfus: Advanced Beginner
```

## Status

🏗️ Skeleton established. Content to be developed.

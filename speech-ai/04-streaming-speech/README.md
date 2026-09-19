# 04 — Streaming Speech

## Purpose

Understand real-time audio streaming engineering, chunked inference, Voice Activity Detection (VAD), and latency budgets.

## Topics

- Chunked and sliding-window ASR: context buffering, lookahead trade-offs
- Streaming TTS: sentence-boundary chunking, streaming vocoder inference
- Real-Time Factor (RTF) and Time-To-First-Audio (TTFA)
- Voice Activity Detection (Silero VAD, WebRTC VAD) and interruption handling (barge-in)
- Network protocols for audio streaming: WebSockets, WebRTC, chunked HTTP

## Key Questions

- How do you balance chunk size between speech recognition accuracy and end-to-end latency?
- How is smooth audio playback maintained on the client while streaming variable-length TTS chunks?
- How does barge-in detection distinguish user interruptions from background noise or echo?

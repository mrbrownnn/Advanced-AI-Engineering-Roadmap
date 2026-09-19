# 06 — Voice Agent Systems

## Purpose

Understand full-duplex conversational voice agents combining VAD, streaming ASR, LLM reasoning, streaming TTS, and interruption management.

## Topics

- End-to-end voice loop: User Audio → VAD → Streaming ASR → LLM Engine → Streaming TTS → Audio Output
- Latency budget allocation: targeting sub-500ms conversational turn turnaround
- Turn-taking management: end-of-turn detection vs mid-turn pauses
- Full-duplex audio and client-side echo cancellation (AEC)
- Native Speech-to-Speech models (e.g. GPT-4o voice, Gemini Live concepts) vs cascaded pipelines

## Key Questions

- Where do latency bottlenecks accumulate in a cascaded (STT + LLM + TTS) architecture?
- How do native speech-to-speech architectures compare in latency and expressiveness with cascaded architectures?
- What state machine governs turn-taking, barge-in, and recovery in conversational voice agents?

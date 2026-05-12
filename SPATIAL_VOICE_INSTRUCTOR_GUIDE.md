# Spatial Voice Instructor Demo Guide

## Goal

Provide a reusable baseline for building XR training flows where a voice instructor guides the user through procedural tasks.

This guide documents architecture and options without shipping scene/source demo assets in this ecosystem hub repository.

## Runtime Flow (Conceptual)

`Voice Input -> Command Event -> LLBridge -> Instructor Message -> Voice Output -> Training Step Progress`

Main building blocks:

- Training scenario runner and step validation.
- Voice command ingestion (keyboard/microphone/STT).
- LLM bridge layer (local or cloud model provider).
- Instructor orchestration and prompt shaping.
- Voice output (human-like provider with TTS fallback).

## LLM Possibilities

- **OpenAI-compatible endpoint** for cloud responses.
- **Ollama local endpoint** for fully local inference.
- **Template fallback** when provider is unavailable.

## Voice Output Possibilities

- **Human-like cloud voice** (for polished demos).
- **Human-like local voice** (for offline/local deployments, e.g. Piper CLI).
- **Local TTS fallback** (Windows speech backends) for resilience.

## Input Possibilities

- Keyboard simulation for deterministic demos.
- Push-to-talk microphone capture with local transcription (Whisper CLI flow).
- Future path: always-on listening with confidence/intent gating.

## Recommended Demo Sequence

1. Intro briefing.
2. Delay before first instruction.
3. Step-by-step guidance (`look -> grab -> place` style baseline).
4. Contextual “what’s next” request.
5. Completion feedback and loop/reset.

## Integration Notes

- Keep public contracts stable (`IXR*` interfaces and event signatures).
- Keep provider choice swappable at runtime.
- Keep fallback paths enabled so demo never hard-fails.
- Keep all user-facing narration in one language per build.

## What To Add Later

- Demo video link.
- Screenshot/GIF of instructor flow.
- Scenario-specific customization examples.


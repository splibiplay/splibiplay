# Spatial Voice Instructor - Developer Guide

## Purpose

This guide explains how to use the `Demo_SpatialVoiceInstructor` scene as a reusable base to build new XR training experiences with:

- procedural training steps,
- voice-driven guidance,
- local/cloud LLM routing,
- human-like voice output (or TTS fallback),
- optional microphone transcription.

Primary scene:

- `Assets/XRCoreAuthoring/Samples/Demo_SpatialVoiceInstructor.unity`

---

## Runtime Architecture

The scene is a vertical slice built from loosely coupled components:

1. **Training flow**
   - `XRTrainingScenarioRunner`
   - Scenario is created by `IndustrialTrainingDemoBootstrap`.
2. **Instructor orchestration**
   - `XRSpatialVoiceInstructorRuntime`
   - Converts commands to guidance and controls startup sequence.
3. **Voice command ingress**
   - Keyboard simulation (`N`, `R`, `T`) and optional microphone (`XRWhisperMicrophoneInput` with push-to-talk `V`).
4. **LLM bridge**
   - `XRLLBridgeRuntime` + provider (`XROpenAiCompatibleLlmProvider` or `XROllamaLocalLlmProvider`).
5. **Voice output**
   - `XRRuntimeTextToSpeechSpeaker`:
     - OpenAI Speech (optional),
     - Piper local CLI (optional),
     - Windows local fallback (`System.Speech` / PowerShell SAPI).
6. **Training interactions**
   - `IndustrialTrainingInteractionEmitter` publishes object-focused and interaction events.

---

## Core Scripts You Will Touch

- `Assets/XRCoreAuthoring/Runtime/Demos/XRSpatialVoiceInstructorDemoBootstrap.cs`
- `Assets/XRCoreAuthoring/Runtime/Demos/XRSpatialVoiceInstructorRuntime.cs`
- `Assets/XRCoreAuthoring/Runtime/Demos/XRRuntimeTextToSpeechSpeaker.cs`
- `Assets/XRCoreAuthoring/Runtime/Demos/XRWhisperMicrophoneInput.cs`
- `Assets/XRCoreLLBridge/Runtime/XRLLBridgeRuntime.cs`
- `Assets/XRCoreLLBridge/Runtime/XROpenAiCompatibleLlmProvider.cs`
- `Assets/XRCoreLLBridge/Runtime/XROllamaLocalLlmProvider.cs`
- `Packages/com.xrcore.training/Runtime/Training/Demo/IndustrialTrainingDemoBootstrap.cs`
- `Packages/com.xrcore.training/Runtime/Training/Demo/IndustrialTrainingInteractionEmitter.cs`

---

## How To Build A New Training On Top

## 1) Duplicate scene and rename

- Duplicate `Demo_SpatialVoiceInstructor.unity`.
- Use a feature-specific name, for example:
  - `Demo_IndustrialValveMaintenance.unity`.

## 2) Replace domain objects

- Replace `Wrench` and `WrenchStation` with your own objects.
- Ensure colliders exist for gaze/raycast targeting.
- Keep one clear target per step while authoring the first version.

## 3) Define scenario steps

In `IndustrialTrainingDemoBootstrap`, `EnsureScenario()` currently builds three steps (`look`, `grab`, `place`).

- Replace with your own steps and events.
- Keep event names explicit and stable.
- Keep step count small (3-6) for first reviewable version.

## 4) Map interactions to events

In `IndustrialTrainingInteractionEmitter`:

- publish signals for the actions you need (`interaction.grab.object`, `interaction.place.object`, custom signals, etc.),
- avoid overloading one signal for multiple intents.

## 5) Adapt instructor language

In `XRSpatialVoiceInstructorRuntime`:

- adjust startup briefing,
- adjust command handling rules,
- adjust LLM prompt template in `BuildPrompt(...)`.

Keep responses concise and action-oriented.

---

## LLM Backend Selection

In `XRSpatialVoiceInstructorDemoBootstrap` set `autoProviderMode`:

- `OpenAiCompatible`
- `OllamaLocal`
- `None` (template fallback only)

In `XRLLBridgeRuntime`:

- `Provider Component` is injected by bootstrap.
- `fallbackToTemplateResponse` controls graceful degradation.

---

## Voice Output Strategy

`XRRuntimeTextToSpeechSpeaker` supports layered output:

1. **Human provider** (optional)
   - OpenAI Speech API
   - Piper local CLI
2. **Local fallback**
   - Windows local TTS
3. **Utterance queue**
   - ensures sequential playback and reduces overlap.

Recommended for production demos:

- keep queue enabled,
- keep gap small (`0.05-0.15s`),
- keep speech short and imperative.

---

## Microphone Input (Whisper Local)

`XRWhisperMicrophoneInput` supports push-to-talk transcription:

- hold `V` to record,
- release to transcribe,
- transcript is passed to `EmitVoiceCommand(...)`.

Required configuration:

- `whisperExecutablePath`
- `whisperModelPath`

Optional:

- `whisperLanguage`, `additionalArguments`, timeout.

If transcription fails, keyboard controls remain available as fallback.

---

## Startup And Sequencing

The startup is intentionally staged:

1. Intro briefing voice
2. Delay
3. Scenario step 1 starts

This is controlled in `XRSpatialVoiceInstructorRuntime` by:

- `playStartupBriefing`
- `startupBriefingMessage`
- `delayBeforeFirstStepSeconds`

---

## Troubleshooting

## No voice heard

- Check `XRRuntimeTextToSpeechSpeaker.enableSpeech`.
- Check selected human provider credentials/paths.
- Check fallback backend availability on Windows.

## LLM responses never change

- Verify provider component assignment in `XRLLBridgeRuntime`.
- Verify API key or local endpoint availability.
- Check provider timeout and model field.

## Microphone commands not recognized

- Verify OS microphone permissions.
- Verify Whisper executable/model paths.
- Use short, direct commands first.

## Step order starts incorrectly

- Ensure scenario auto-start is disabled in bootstrap-controlled mode.
- Use `BeginIntroAndStartScenario()` flow.

---

## Extension Ideas

- Add role-based instruction variants (novice/expert).
- Add safety warnings based on interaction mistakes.
- Add multi-tool workflows with branching validation.
- Add assessment scoring and completion rubric.
- Add localization profiles while keeping canonical signal names.

---

## Completion Checklist For A New Training

- [ ] Scene duplicated and renamed.
- [ ] Domain objects replaced.
- [ ] Steps and validators updated.
- [ ] Signals mapped and tested.
- [ ] Instructor copy and prompt tuned.
- [ ] Voice backend configured (human + fallback).
- [ ] Microphone path validated.
- [ ] One full run completed without runtime errors.
- [ ] Demo script documented for recording.

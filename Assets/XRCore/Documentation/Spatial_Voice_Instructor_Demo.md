# XRCore Spatial Voice Instructor Demo

## Goal

Deliver a demo-visible product slice where a technician receives contextual voice guidance while completing an industrial training procedure.

Developer guide:

- `Assets/XRCore/Documentation/Spatial_Voice_Instructor_Developer_Guide.md`

## Included Runtime Chain

`Voice command -> LLBridge response -> agent instruction signal -> training progression`

- Voice input simulation emits `XRVoiceCommandEvent`.
- The instructor runtime listens to voice commands and builds contextual prompts.
- LLBridge returns a concise response (`XRLLBridgeExchangeEvent`).
- Instructor runtime publishes `xr.agent.instruction` and updates on-screen guidance.
- Training toolkit validates the physical steps (`look`, `grab`, `place`).
- Runtime TTS speaker reads instructor messages in Windows Editor/Standalone.

## One-click Scene Creation

- `Tools -> XRCore -> Commercial -> Create Spatial Voice Instructor Demo Scene`
- Or open `Tools -> XRCore -> Commercial -> Demo Studio` and use `SpatialVoiceInstructor`.

Created scene:

- `Assets/XRCoreAuthoring/Samples/Demo_SpatialVoiceInstructor.unity`

## Play Mode Controls

- Voice simulation:
  - `N`: Ask next instruction.
  - `R`: Repeat current instruction.
  - `T`: Ask status.
  - `V` (hold): Push-to-talk microphone capture (Whisper local CLI).
- Interaction:
  - `G`: Grab wrench (while looking at it).
  - `P`: Place wrench on station.

## LLBridge Model Wiring

By default, LLBridge keeps a safe template fallback response. For real LLM responses:

- In `XRSpatialVoiceInstructorDemoBootstrap`, choose `autoProviderMode`:
  - `OpenAiCompatible` -> component `XROpenAiCompatibleLlmProvider`.
  - `OllamaLocal` -> component `XROllamaLocalLlmProvider`.
- OpenAI-compatible:
  - Set `apiKey` or env var `XRCORE_LLM_API_KEY`.
  - Optional: change endpoint/model.
- Ollama local:
  - Default endpoint: `http://127.0.0.1:11434/api/generate`
  - Set model tag (example: `llama3.2:3b`).

If no provider key is available, demo stays functional using fallback responses.

## Microphone Input (Whisper Local)

Component: `XRWhisperMicrophoneInput` (auto-created on the demo root).

Required fields:

- `whisperExecutablePath`: local Whisper CLI executable path (for example `whisper-cli.exe`).
- `whisperModelPath`: local model file path.
- Optional: `whisperLanguage`, `additionalArguments`, `processTimeoutSeconds`.

Flow:

- Hold `V` to record from PC microphone.
- Release `V` to transcribe.
- Transcript is routed to `XRSpatialVoiceInstructorRuntime.EmitVoiceCommand(...)`.

## Voice Output Modes

`XRRuntimeTextToSpeechSpeaker` supports layered output:

- Human-like provider (optional): OpenAI Speech API (`/v1/audio/speech`), WAV playback.
- Human-like local provider (optional): `PiperLocalCli` (CLI + local voice model).
- Local fallback: Windows TTS (`System.Speech` or PowerShell/SAPI).
- Queue mode: utterances are spoken sequentially to avoid overlap.

## What This Demo Proves

- Product value in one vertical scene (not isolated module tests).
- Event-driven interoperability across XRCore SDK, Training, Voice, and LLBridge.
- Release-safe extension on top of the stabilized architecture and contracts.

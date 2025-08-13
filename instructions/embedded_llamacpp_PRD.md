# LocalCoder IDE — Embedded llama.cpp Backend PRD

Owner: Aaryan Anand

Status: Draft — Ready for implementation

## Objective
Enable LocalCoder IDE to run open-weight LLMs entirely locally via an embedded llama.cpp backend, optimized for 18 GB RAM Apple Silicon machines.

## Scope
Includes backend integration, model management, settings, core AI commands, onboarding, resilience features, privacy safeguards.

Out of scope (initial): cloud backends, remote inference, marketplace publishing.

## Target Hardware
- macOS Apple Silicon M-series Pro baseline (18 GB RAM)
- Default models: 7–8B Q4 quant
- Optional heavy mode: 13–14B Q4

## Key Requirements
1. **Backend & Process Architecture**
   - Dedicated worker process for llama.cpp
   - IPC connection between IDE and worker
   - Metal acceleration on by default, CPU fallback toggle
   - Profiles: Performance vs Stability

2. **Model Management**
   - Storage: `~/Library/Application Support/LocalCoder/models/`
   - Format: GGUF
   - Catalog JSON for default models
   - Download/import with checksum verification
   - Version pinning

3. **Default Models**
   - Qwen2.5-Coder-7B Q4
   - Llama 3-8B Q4
   - 13–14B Q4 as heavy option

4. **Settings & Profiles**
   - Backend: embedded
   - Selectable models for code/general
   - Profiles: Fast (7B Q4), Balanced (8B Q4), Heavy (13B Q4)
   - Context window, sampling params, residency toggle

5. **Core AI Commands**
   - Improve Selection
   - Explain Selection
   - Model Switch
   - Model Manager UI

6. **Failure Handling**
   - OOM auto-fallback to smaller model
   - Crash restart without IDE restart
   - Slow performance → suggest lighter profile

7. **Privacy & Licensing**
   - No network calls in air-gapped mode
   - Only redistribute open-weight models
   - Show license before download

## Step-Based Implementation
1. Backend integration with IPC and Metal acceleration
2. Model Manager with download/import
3. Settings surface for backend, models, profiles
4. AI commands wired to backend
5. First-run onboarding flow
6. Resilience features for OOM, crashes, idle unload

## Non-Functional
- Warm token: ≤ 250ms
- Cold token: ≤ 800ms
- ≥ 30 tok/s throughput for 7B Q4
- No IDE crashes from backend failure
- ≥ 60% acceptance rate in tests

## Risks
- RAM limits for 13B — gate behind heavy profile
- Metal bugs — offer CPU fallback
- Licensing — curated catalog only
- First load delay — pre-warm option

## Appendix: IPC Surface (initial)
- ping: health check
- loadModel(modelRef, profile): load and warm
- unloadModel(): free resources
- generate(prompt, params, ctrl): stream tokens and usage
- cancel(requestId): interrupt generation

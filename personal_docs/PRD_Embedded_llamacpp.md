# Product Requirements Document (PRD)

## Product
LocalCoder IDE — Embedded llama.cpp Backend

## Owner
Aaryan Anand

## Status
Draft — Ready for implementation

---

## 1. Objective
Enable LocalCoder IDE to run open‑weight LLMs entirely locally via an embedded llama.cpp backend, optimized for 18 GB RAM systems (M‑series Pro baseline), providing offline AI‑assisted coding with zero external dependencies.

---

## 2. Scope
This PRD covers:

- Integration of `llama.cpp` as a worker process inside the VS Code fork
- Model management (download/import/delete)
- Default performance profiles and context settings
- Core AI commands (Improve, Explain)
- Failure handling and fallback mechanisms
- Privacy and licensing safeguards

Out of scope (initially): cloud backends, remote inference, marketplace publishing.

---

## 3. Target Hardware
- macOS (Apple Silicon) — M‑series Pro, 18 GB RAM baseline
- 7–8B Q4 models as defaults
- 13–14B Q4 as optional “Heavy” mode with reduced context

---

## 4. Key Requirements

### 4.1 Backend & Process Architecture
- Worker Process: `llama.cpp` runs in a dedicated helper process (IPC to IDE)
- Crash Isolation: Worker restartable without restarting IDE
- Metal Acceleration: Enabled by default; toggle for “Compatibility mode” (CPU fallback)
- Performance vs. Stability Mode:
  - Performance: aggressive GPU offload, keep model resident
  - Stability: conservative offload, unload on idle

### 4.2 Model Management
- Storage Path: `~/Library/Application Support/LocalCoder/models/`
- Format: GGUF only
- Catalog: JSON mapping model names to URL, size, quantization, license
- Download Manager:
  - Resumable downloads
  - SHA‑256 verification
  - Disk usage and license displayed before install
- Import: User can manually add GGUF
- Versioning: Models pinned by digest

### 4.3 Default Models
- Code Model: `Qwen2.5-Coder-7B` (Q4)
- General Model: `Llama 3‑8B` (Q4)
- Heavy Option: 13–14B (Q4) with reduced context

### 4.4 Settings & Profiles
- Backend: `embedded`
- Models: `codeModel`, `generalModel`
- Performance Profiles:
  - Fast (7B Q4, ctx=4k)
  - Balanced (8B Q4, ctx=4k–8k)
  - Heavy (13B Q4, ctx=4k)
- Advanced:
  - Context window up to 8k
  - Sampling params (temp, top_p, repeat_penalty)
  - Residency toggle (keep model loaded on idle)

### 4.5 Core AI Commands
- Improve Selection — Diff preview → Apply
- Explain Selection — Sidebar summary
- Model Switch — Warm‑load new model
- Model Manager — Download, import, delete

### 4.6 Failure Handling
- OOM: Auto‑fallback to smaller quant or lower `num_ctx`; offer retry
- Crash: Restart worker, suggest lighter profile if repeated
- Slow Model: Suggest “Fast” profile
- Checksum Fail: Abort and prompt re‑download

### 4.7 Privacy & Licensing
- No network calls in “Air‑gapped mode”
- Only list redistributable open‑weight models in catalog
- Show license terms before download
- Allow manual import for custom models

---

## 5. Step‑Based Implementation Plan

### Step 1 — Backend Integration
- Implement worker process for `llama.cpp` with IPC
- Wire start/stop, load/unload, generate, and health‑check commands
- Enable Metal acceleration with toggle

### Step 2 — Model Manager
- Build local model store
- Implement download/import with checksum verification
- Load models into worker on demand

### Step 3 — Settings Surface
- Add backend, model selection, and performance profiles to settings
- Implement context and sampling parameter controls
- Residency and performance/stability toggles

### Step 4 — AI Command Wiring
- Hook “Improve Selection” and “Explain Selection” to backend
- Add streaming output with cancel
- Implement diff preview

### Step 5 — Onboarding Flow
- First‑run wizard:
  - Select models
  - Download/import
  - Self‑test
  - Air‑gapped mode switch

### Step 6 — Resilience Features
- OOM auto‑fallback
- Crash restart sequence
- Idle unload + pre‑warm

---

## 6. Non‑Functional Requirements
- Performance:
  - Warm first token ≤ 250 ms
  - Cold first token ≤ 800 ms
  - Throughput ≥ 30 tok/s on 7B Q4
- Stability: No propagated IDE crashes
- Usability: Edit acceptance ≥ 60% in internal testing

---

## 7. Risks & Mitigations
- RAM Limit on 13B: Gate behind heavy profile, reduce context
- Metal Bugs: Offer CPU fallback
- Model Licensing: Curate catalog; require consent
- First Load Delay: Pre‑warm models and allow idle residency

---

## Appendix: IPC Surface (initial sketch)
- `ping`: health check
- `loadModel(modelRef, profile)`: load and warm
- `unloadModel()`: free resources
- `generate(prompt, params, ctrl)`: stream tokens and usage
- `cancel(requestId)`: interrupt generation



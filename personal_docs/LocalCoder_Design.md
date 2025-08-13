# LocalCoder: Fully-Local AI-Enhanced VS Code

## Goal
A VS Code fork with an integrated local AI assistant for code improvement, explanation, and refactoring, running entirely on-device using Apple Silicon’s Metal acceleration.

---

## Motivation
- No cloud calls, no data leakage
- Leverage Apple Silicon M-series for local LLM inference
- Target devs who need privacy + fast context-aware code help
- Extendable to NVIDIA GPU builds later

---

## Core Features (MVP)
1. Local-only inference
   - Powered by Ollama or llama.cpp
   - Uses quantized GGUF models for low-latency
2. Inline code actions
   - “Improve Selection”
   - “Explain Selection”
3. Persistent local memory
   - Stores project-wide context
4. Switchable models
   - Separate “code” and “general” assistants
5. No telemetry

---

## Architecture Overview

**Frontend (VS Code fork)**
- Adds context menu items for inline actions
- Calls into a new `localcoder` extension host

**LocalCoder Extension**
- Manages prompts, collects context
- Sends request to local HTTP endpoint (Ollama server)
- Streams back and applies edits

**Backend (Local Model Server)**
- Ollama running models pulled at install/setup
- Supports `/v1/completions` or `/v1/chat/completions`

See also: `personal_docs/arch_diagram.md` for a high-level diagram.

---

## Data/Call Flow (MVP)
1. User triggers a command (e.g., Improve Selection).
2. Extension collects context (selection, file path, repo map, recent edits).
3. Extension calls local server (`http://localhost:11434`) with a templated prompt.
4. Model streams tokens back; UI renders partial output and tooltips.
5. Optional: persist interaction to local memory (SQLite/JSONL).

---

## Model Guidance for Apple Silicon (M3 Pro)

Practical targets for an M3 Pro (18–36 GB unified memory) using Metal-accelerated backends (Ollama/llama.cpp) with GGUF quantization.

| Tier | Use-case | Examples | Quant | Context | tok/s | Notes |
|---|---|---|---|---|---|---|
| A (Fast) | Chat, small refactors | Phi-3-mini, Mistral 7B, Llama 3 8B | Q4_K_M | 4k–8k | 40–80 | Great latency |
| B (Coder 7–8B) | Inline code help | DeepSeek-Coder 6.7B, Qwen2.5-Coder 7B, CodeLlama 7B | Q4/Q5_K_M | 4k–8k | 30–60 | Balanced |
| C (Coder 13–14B) | Large refactors | CodeLlama 13B, Qwen2.5-Coder 14B | Q4_K_M | 8k–16k | 15–30 | Better planning |
| D (General 32B) | Heavy reasoning | Llama 3 32B | Q4_K_M | 8k–16k | 6–12 | Slow, high mem |

Memory needs (Q4_K_M): 7–8B ≈ 4–6 GB, 13–14B ≈ 8–12 GB, 32B ≈ 18–24 GB. Keep 3–6 GB free for OS/Electron.

---

## Implementation Notes (initial)
- Extension commands: `localcoder.improveSelection`, `localcoder.explainSelection`.
- Transport: HTTP(S) to `localhost` with streaming (Server-Sent Events or chunked JSON).
- Apply edits using VS Code `WorkspaceEdit` with proper undo grouping; show preview diff.
- Settings: server URL, active model, max tokens, temperature, per-language toggles.
- Persistence: lightweight local store (SQLite/JSONL) scoped to workspace; opt-in.
- Privacy: no telemetry, block network egress except `localhost` (configurable hard gate).

---

## Open Questions / Clarifications
1. Backend choice & protocol
   - Ollama vs llama.cpp (API compatibility, streaming format)?
   - SSE, WebSocket, or chunked JSON? Error and cancellation semantics?
2. Context collection
   - Max context window, file size limits, repo indexing strategy, embeddings (now or later)?
3. Prompting
   - Templates per language? How to ensure deterministic edits vs free-form text?
4. Persistence
   - SQLite vs JSONL; schema and retention; encryption-at-rest needed?
5. Model management
   - Who installs/pulls models? First-run flow to fetch recommended models?
6. UX
   - Where to show streaming output? Inline ghost text vs panel vs diff view?
7. Performance budgets
   - Min target tok/s, max latency for inline actions; fallback behaviors.
8. Hard privacy gate
   - Should we enforce localhost-only calls at the extension level?

---

## Milestones (proposed)
- M0: Skeleton extension, settings, command wiring, hello-world request to local server.
- M1: Implement Improve Selection end-to-end with streamed apply and preview.
- M2: Explain Selection + basic persistence of interactions.
- M3: Model switcher UI + recommended models installer flow.
- M4: Repository-wide context collection + quality iterations.

---

## Dev Environment
Follow `personal_docs/DEV-SETUP.md` for macOS self-hosting steps. Run:
```bash
npm run watch
./scripts/code.sh
```

---

## One-line Summary
LocalCoder is a privacy-first VS Code fork plus extension that performs useful, context-aware code transforms and explanations entirely on-device by talking to a local LLM server.



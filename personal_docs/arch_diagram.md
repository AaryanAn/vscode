## Architecture (Local-First)

```mermaid
flowchart LR
  subgraph IDE["VS Code Fork (Electron + TS)"]
    A[Editor + UI]
    B[Local Agent Sidebar]
    C[Commands/Keybindings]
    D[Extension Host]
    E[Settings & Model Picker]
  end

  subgraph Runtime["Local AI Runtime"]
    F[HTTP Client\n(fetch, streaming)]
    G[(Local Memory\nSQLite/JSONL)]
  end

  subgraph Inference["Model Server (local)"]
    H[Ollama / llama.cpp\nMetal (apple-silicon)]
    I[GGUF Models\n(quantized)]
  end

  A <--> D
  B --> F
  C --> F
  E --> F
  F <--> H
  H <--> I
  B <--> G
  C <--> G

  classDef ide fill:#1f2937,stroke:#fff,stroke-width:1px,color:#fff;
  classDef runtime fill:#0f766e,stroke:#fff,stroke-width:1px,color:#fff;
  classDef infer fill:#7c3aed,stroke:#fff,stroke-width:1px,color:#fff;

  class A,B,C,D,E ide
  class F,G runtime
  class H,I infer

# Ingestion Flow: Staff Update to Redis/App

Main: [ADR-015_SSG_ENGINE_SELECTION](./ADR-015_SSG_ENGINE_SELECTION.md)
Related: [ADR-016_AI_SEMANTIC_LAYER](./ADR-016_AI_SEMANTIC_LAYER.md)

This diagram tracks how a Markdown edit becomes a searchable AI vector (ADR-016) and an offline JSON asset.

```mermaid
sequenceDiagram
    participant Staff as Advocacy Staff
    participant Git as GitHub/Local FS
    participant Script as generate-guide.js
    participant JSON as bundled-assets.json
    participant Redis as Redis (Vector DB)
    participant OAI as OpenAI (Embeddings)

    Staff->>Git: Edit Markdown (Front-matter + Content)
    Git->>Script: Trigger Build Pipeline
    
    Note over Script,JSON: ADR-015 (Offline Layer)
    Script->>Script: Parse Front-matter & Content
    Script->>JSON: Append to Bundled JSON
    
    Note over Script,Redis: ADR-016 (AI Semantic Layer)
    Script->>OAI: Send Text Chunks for Embedding
    OAI-->>Script: Return Vector Arrays (Numbers)
    Script->>Redis: Store Text + Metadata + Vectors
    
    Note over Script: Pipeline Complete
```


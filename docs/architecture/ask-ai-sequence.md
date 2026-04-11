# Ask AI Sequence

Main: [ADR-016_AI_SEMANTIC_LAYER](./ADR-016_AI_SEMANTIC_LAYER.md)

This sequence describes the high-speed "Vertical Slice" that allows the native app to query the digital guide via the **Astro Actions** layer defined in [ADR-018](./ADR-018_WEB_MGMT_LAYER_API.md).

```mermaid
sequenceDiagram
    autonumber
    actor User as User (Mobile App)
    participant RN as React Native (Paper UI)
    participant AA as Astro Action (ADR-018)
    participant R as Redis (ADR-016)
    participant AI as OpenAI (GPT-4o)

    Note over User, RN: User initiates natural language query
    User->>RN: "How do I get CalFresh?"
    
    RN->>AA: actions.askAI({ question })
    
    Note over AA, R: Semantic Retrieval (The "Pull")
    AA->>R: similaritySearch(question)
    R-->>AA: Page 37 Snippets (ADR-015 Content)
    
    Note over AA, AI: RAG Grounding (The "Handshake")
    AA->>AI: Prompt: Use [Page 37] to answer [Question]
    AI-->>AA: "Call (866) 613-3777 to apply..."
    
    AA-->>RN: { answer: "Call (866) 613-3777..." }
    
    RN->>User: Displays Answer in Card


# ADR-002: The "Air-Gapped" Integrity Logic (Bronze Tier)

## Status: Proposed (Tier 1)

## Context
Stakeholders (HALA Board) expressed concern regarding AI "hallucinations" or inaccuracies within a guide used for life-essential food resources. We must ensure the AI assistant cannot alter or misrepresent the "Single Source of Truth."

## Decision
We will implement a Physical Logic Air-Gap within the application architecture:

   1. Verified Layer (ReadOnly): The primary guide text is served from a static, read-only Firestore collection (ingested from the PDF). It is immutable by the AI.
   2. AI Sandbox (Semantic Search): The "Ask HALA" module operates in a separate sandbox. It can read the verified layer to provide answers but has zero "write" or "edit" permissions.
   3. UI Enforcement: The "Toggle of Truth" (Pilot Toggle) explicitly switches the user's context between these two isolated data streams.

## Logic
This architecture protects HALA’s 20-year reputation for accuracy while allowing for modern, natural-language discovery.

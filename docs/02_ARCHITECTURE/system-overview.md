---
tags:
  - architecture
cssclasses:
  - architecture-doc
---

# System Overview

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER LAYER                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Web App (Next.js)                          │   │
│  │  - Chat UI          - Deep Think Toggle                 │   │
│  │  - Session History  - File Upload                       │   │
│  │  - Auth via Supabase                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS + JWT
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CONTROL PLANE                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Gateway (FastAPI)                          │   │
│  │  - JWT Validation      - Session Management             │   │
│  │  - Request Routing     - RAG Retrieval                  │   │
│  │  - Tool Execution      - Logging & Audit                │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                    │                    │
         ┌──────────┘                    └──────────┐
         │                                          │
         ▼                                          ▼
┌─────────────────────┐                ┌─────────────────────┐
│   INSTANT INFERENCE │                │  THINKING INFERENCE │
│  ┌───────────────┐  │                │  ┌───────────────┐  │
│  │   GPU Pod     │  │                │  │ Serverless GPU│  │
│  │  Always-On    │  │                │  │ Scale-to-Zero │  │
│  │  Fast Model   │  │                │  │ Strong Model  │  │
│  └───────────────┘  │                │  └───────────────┘  │
└─────────────────────┘                └─────────────────────┘
                              │
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Supabase                                   │   │
│  │  - Auth (Users, JWT)     - Storage (Documents)          │   │
│  │  - Postgres (Data)       - pgvector (Embeddings)        │   │
│  │  - RLS (Multi-tenant)                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Responsibilities

### Web App (Next.js)
- Presents [[chat-ux|chat interface]] to user
- Handles [[auth-ux|authentication flow]] via [[Supabase]] Auth
- Sends requests to [[gateway-responsibilities|Gateway]] with [[auth-and-jwt|JWT]]
- Displays responses, mode used, and [[rag-flow|citations]]

See [[ui-pages-and-components]] for full details.

### Gateway (FastAPI)
- Central brain of the system — see [[openclaw-style-gateway-notes]]
- Validates every request ([[auth-and-jwt|JWT verification]])
- Stores [[schema|chat sessions and messages]]
- Performs [[rag-flow|RAG retrieval]] when relevant
- Routes to appropriate [[routing-logic|inference tier]]
- Executes [[tools-framework|tools]] securely on backend
- Returns structured responses to UI

See [[gateway-responsibilities]] for full details.

### Instant Inference (GPU Pod)
- Runs fast instruct model — see [[inference-instant]]
- Always-on for responsiveness — see [[adr-0001-two-tier-inference]]
- [[networking-and-private-access|Private endpoint]] (only Gateway can reach)
- [[adr-0003-openai-compatible-inference-interface|OpenAI-compatible API]] interface

### Thinking Inference (Serverless GPU)
- Runs stronger reasoning model — see [[inference-thinking]]
- Scales to zero when idle — see [[cost-controls]]
- Cold start tolerated for cost savings
- Private endpoint, same API interface

### Supabase
- **Auth**: User management, session tokens, [[auth-and-jwt|JWT issuance]]
- **Postgres**: [[schema|Structured data]] (sessions, messages, profiles)
- **Storage**: [[storage-and-files|File uploads]] (documents, images)
- **pgvector**: [[embeddings-and-pgvector|Embedding storage and vector search]]
- **RLS**: [[rls-policies|Row-level security]] for [[adr-0004-rls-multi-tenant-from-day-1|multi-tenant isolation]]

See [[supabase-local-dev-plan]] for local setup, [[adr-0002-supabase-local-then-cloud]] for strategy.

---

## Data Flow Summary

1. User logs in → [[Supabase]] issues [[auth-and-jwt|JWT]]
2. User sends message → [[ui-pages-and-components|Web App]] calls [[gateway-responsibilities|Gateway]] with JWT
3. Gateway validates JWT → stores message → retrieves [[rag-flow|RAG context]] (optional)
4. Gateway [[routing-logic|routes]] to [[inference-instant|Instant]] or [[inference-thinking|Thinking]] inference
5. Model returns response → Gateway stores it → returns to UI
6. UI displays response with metadata (mode used, [[rag-flow|citations]])

For detailed flow diagrams, see [[data-flow]].

---

## Key Design Decisions

| Decision | Rationale | Reference |
|----------|-----------|------------|
| [[gateway-responsibilities|Gateway]] owns routing | Models stay "dumb"; policy lives centrally | [[openclaw-style-gateway-notes]] |
| [[adr-0001-two-tier-inference|Two inference tiers]] | Balance cost vs capability | [[inference-instant]], [[inference-thinking]] |
| [[Supabase]] for everything | Auth + DB + Storage in one platform | [[adr-0002-supabase-local-then-cloud]] |
| RLS from day one | Multi-tenant safety without refactoring later |
| OpenAI-compatible API | Easy model swapping, standard tooling |


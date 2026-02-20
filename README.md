# Local AI Assistant

An open source AI assistant system designed with multi-user architecture from day one. This project aims to build a personal AI that matches the experience of ChatGPT while maintaining full control over infrastructure, data, and costs.

## Overview

This system provides a chat interface with two inference modes:

| Mode | Purpose | Infrastructure | Latency |
|------|---------|----------------|---------|
| Instant Mode | Daily tasks (coding, writing, planning) | Always-on GPU pod | Fast |
| Thinking Mode | Complex reasoning, deep analysis | Serverless GPU (scale-to-zero) | Cold start OK |

## Key Features

- **Two-Tier Inference**: Balance between cost and capability with instant and thinking modes
- **Full Data Ownership**: All data stored in self-controlled Supabase instance
- **No Third-Party LLM API Costs**: Pay for GPU compute, not per-token API fees
- **Multi-User Ready Architecture**: Multi-tenant schema with Row-Level Security from day one
- **Private Infrastructure**: Inference endpoints are private and only accessible via Gateway

## Architecture

```
User Layer (Next.js Web App)
         |
         | HTTPS + JWT
         v
Control Plane (FastAPI Gateway)
    |                    |
    v                    v
Instant Inference    Thinking Inference
(GPU Pod)            (Serverless GPU)
         |
         v
Data Layer (Supabase)
- Auth (Users, JWT)
- Postgres (Data)
- Storage (Documents)
- pgvector (Embeddings)
- RLS (Multi-tenant)
```

## Tech Stack

- **Frontend**: Next.js
- **Backend/Gateway**: FastAPI (Python)
- **Database**: Supabase (PostgreSQL + pgvector)
- **Authentication**: Supabase Auth with JWT
- **Inference**: OpenAI-compatible API endpoints on GPU pods

## Project Structure

```
docs/
  00_README/          # Project overview and glossary
  01_PRODUCT/         # Vision, goals, MVP scope
  02_ARCHITECTURE/    # System design and data flow
  03_INFRA/           # Deployment and infrastructure
  04_DATABASE_SUPABASE/  # Schema, RLS, migrations
  05_GATEWAY_API/     # API endpoints and routing
  06_WEB_APP/         # UI/UX documentation
  07_SECURITY/        # Security and threat model
  08_ROADMAP/         # Development phases
  09_DECISIONS/       # Architecture Decision Records
```

---

## Progress Tracker

### Phase 1: Chat MVP
Build end-to-end chat functionality with message persistence.

| Task | Status |
|------|--------|
| Set up Supabase locally (Auth, DB) | Not Started |
| Create database schema (profiles, sessions, messages) | Not Started |
| Enable RLS policies | Not Started |
| Build Gateway foundation (FastAPI) | Not Started |
| Implement JWT validation | Not Started |
| Create /chat endpoint | Not Started |
| Connect Instant inference (GPU pod) | Not Started |
| Build basic Web UI (login, chat, sessions) | Not Started |

### Phase 2: Two Modes
Add Thinking Mode with user toggle between Instant and Think.

| Task | Status |
|------|--------|
| Deploy serverless thinking endpoint | Not Started |
| Configure scale-to-zero | Not Started |
| Implement routing logic in Gateway | Not Started |
| Add Deep Think toggle in UI | Not Started |
| Display mode indicator on responses | Not Started |
| Handle cold start UX gracefully | Not Started |

### Phase 3: RAG (Retrieval-Augmented Generation)
Add document upload and context-aware responses.

| Task | Status |
|------|--------|
| Build file upload endpoint | Not Started |
| Implement text extraction (PDF, TXT, MD) | Not Started |
| Create chunking logic | Not Started |
| Generate and store embeddings (pgvector) | Not Started |
| Implement vector search for context retrieval | Not Started |
| Display citations in UI | Not Started |

### Phase 4: Tools
Add tool execution capability for extended functionality.

| Task | Status |
|------|--------|
| Build tool execution framework in Gateway | Not Started |
| Implement tool detection from model responses | Not Started |
| Create web_search tool | Not Started |
| Create save_note and search_notes tools | Not Started |
| Create task management tools | Not Started |
| Add tool logging and audit | Not Started |

### Phase 5: Production Launch
Transition from private use to production-ready deployment.

| Task | Status |
|------|--------|
| Deploy to production infrastructure | Not Started |
| Set up Supabase Cloud | Not Started |
| Complete security hardening | Not Started |
| Configure monitoring and alerting | Not Started |
| Set up backup and recovery | Not Started |
| Implement user onboarding flow | Not Started |

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Instant Mode latency | Under 2 seconds first token |
| Thinking Mode cold start | Under 30 seconds |
| Data isolation | RLS passes security audit |
| Local-to-prod migration | Under 1 day |
| Monthly infrastructure cost (private use) | Under $150 |

## Documentation

For detailed documentation, see the `docs/` folder:

- [Overview](docs/00_README/overview.md) - Project overview
- [Step-by-Step Guide](docs/00_README/step-by-step-guide.md) - Build instructions
- [MVP Scope](docs/01_PRODUCT/mvp-scope.md) - What is in/out of scope
- [System Overview](docs/02_ARCHITECTURE/system-overview.md) - Technical architecture
- [Schema](docs/04_DATABASE_SUPABASE/schema.md) - Database structure

## License

Private project - All rights reserved.

---
tags:
  - product
cssclasses:
  - product-doc
---

# MVP Scope

## Definition of MVP

The Minimum Viable Product is: **"Chat works end-to-end with two inference modes."**

For build steps, see [[step-by-step-guide]] and [[phase-1-chat-mvp]].
For metrics, see [[success-metrics]].

---

## In Scope for MVP

### Core Chat Experience
- User can log in via [[auth-and-jwt|Supabase Auth]] — see [[auth-ux]]
- User can send messages and receive responses — see [[chat-ux]]
- Messages are stored persistently — see [[schema]]
- [[schema|Session history]] is accessible

### Two-Mode Inference
- Default: [[inference-instant|Instant Mode]] (fast, always-on)
- Toggle: [[inference-thinking|Thinking Mode]] (stronger, cold start OK) — see [[adr-0001-two-tier-inference]]
- UI shows which mode was used — see [[chat-ux]]

### Basic Infrastructure
- [[supabase-local-dev-plan|Local Supabase]] running (Auth + DB + Storage)
- [[gateway-responsibilities|Gateway API]] handling [[routing-logic|routing]]
- One GPU pod for [[inference-instant|Instant inference]]
- One serverless endpoint for [[inference-thinking|Thinking inference]]

### Security Baseline
- [[auth-and-jwt|JWT validation]] on all requests
- [[rls-policies|RLS enabled]] on all user tables — see [[adr-0004-rls-multi-tenant-from-day-1]]
- [[networking-and-private-access|Private inference endpoints]] (not public)

---

## Out of Scope for MVP

### RAG / Document Memory → [[phase-3-rag]]
- [[file-upload-ux|File uploads]]
- Text extraction and chunking — see [[rag-flow]]
- [[embeddings-and-pgvector|Embedding storage]]
- Context retrieval

### Tools Framework → [[phase-4-tools]]
- Web search — see [[tools-framework]]
- Note saving
- Task management
- Any external integrations

### Advanced UX
- Streaming responses
- [[rag-flow|Citations display]]
- Usage analytics
- Mobile optimization

### Production Deployment → [[phase-5-production-launch]]
- [[adr-0002-supabase-local-then-cloud|Hosted Supabase]]
- Public web app
- [[rate-limiting-and-abuse|Rate limiting]]
- [[logging-and-auditing|Monitoring and alerting]]

---

## MVP User Stories

1. **As a user**, I can log in securely and see my chat history.
2. **As a user**, I can send a message and receive a response quickly.
3. **As a user**, I can toggle "Deep Think" for complex questions.
4. **As a user**, I can see which mode was used for each response.
5. **As a user**, I can start new sessions and switch between them.

---

## MVP Technical Milestones

| # | Milestone | Acceptance |
|---|-----------|------------|
| 1 | Local Supabase running | Auth works, tables exist |
| 2 | Gateway /chat endpoint | Stores message, calls inference |
| 3 | Instant inference connected | Response returns in under 3s |
| 4 | Thinking inference connected | Response returns (cold start OK) |
| 5 | Web UI functional | Login, chat, sessions, toggle |

---

## Post-MVP Priorities

1. RAG memory system
2. Basic tools (web search, notes)
3. Streaming responses
4. Production deployment prep


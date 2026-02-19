---
tags:
  - roadmap
cssclasses:
  - roadmap-doc
---

# Phase 1: Chat MVP

## Goal

Get end-to-end chat working: user sends message, receives response, history persists.

For build steps, see [[step-by-step-guide]].
For MVP scope, see [[mvp-scope]].

---

## Success Criteria

- [ ] User can log in and see their account
- [ ] User can send a message and receive a response
- [ ] Messages are stored and persist across sessions
- [ ] Basic chat UI is functional
- [ ] Instant inference connected and working

---

## Scope

### In Scope
- [[supabase-local-dev-plan|Supabase local setup]] (Auth, DB)
- [[gateway-responsibilities|Gateway]] with [[endpoints-contracts|/chat endpoint]]
- [[inference-instant|Instant inference]] integration
- Basic [[ui-pages-and-components|web UI]] (login, chat, sessions)
- [[schema|Message storage]]

### Out of Scope
- [[inference-thinking|Thinking mode]] → [[phase-2-two-modes]]
- [[rag-flow|RAG / document context]] → [[phase-3-rag]]
- [[tools-framework|Tools]] → [[phase-4-tools]]
- [[file-upload-ux|File uploads]] → [[phase-3-rag]]
- Advanced UX

---

## Milestones

### M1.1: Local Infrastructure
**Goal:** Development environment ready

- Set up [[supabase-local-dev-plan|Supabase locally]]
- Create [[schema|database schema]] (profiles, sessions, messages)
- Enable [[rls-policies|RLS policies]] — see [[adr-0004-rls-multi-tenant-from-day-1]]
- Verify [[auth-and-jwt|Auth]] working

**Acceptance:** Can create user, login, see profile

---

### M1.2: Gateway Foundation
**Goal:** Basic [[gateway-responsibilities|Gateway API]] running

- FastAPI project structure
- [[auth-and-jwt|JWT validation]] working
- [[endpoints-contracts|/health endpoint]]
- Basic [[logging-and-auditing|logging]]

**Acceptance:** Gateway starts, health check passes, invalid JWT rejected

---

### M1.3: Chat Endpoint
**Goal:** /chat endpoint stores and retrieves messages

- POST /chat accepts message
- Creates session if needed
- Stores user message
- Calls inference (mock initially)
- Stores response
- Returns to client

**Acceptance:** Full round-trip with mock response

---

### M1.4: Instant Inference
**Goal:** Real model responding

- GPU pod running with model
- OpenAI-compatible endpoint
- Gateway can reach endpoint
- Real responses coming back

**Acceptance:** Ask "what is 2+2" and get sensible response

---

### M1.5: Web UI Basics
**Goal:** Functional chat interface

- Login page
- Chat page with input and messages
- Send message, see response
- Sessions list (basic)

**Acceptance:** Can complete full chat flow in browser

---

### M1.6: Session Management
**Goal:** Sessions work correctly

- Create new session
- Load existing session
- Switch between sessions
- Delete session

**Acceptance:** Multiple sessions work independently

---

## Technical Notes

### Database Schema (Phase 1)
```
profiles
- id (uuid, PK, FK to auth.users)
- email
- created_at

chat_sessions
- id (uuid, PK)
- user_id (FK to profiles)
- title
- created_at
- updated_at

chat_messages
- id (uuid, PK)
- session_id (FK to chat_sessions)
- user_id (FK to profiles)
- role (user/assistant)
- content
- created_at
```

### API Contracts (Phase 1)
```
POST /chat
- Input: { session_id?, message }
- Output: { session_id, message_id, content }

GET /sessions
- Output: { sessions: [...] }

GET /sessions/{id}/messages
- Output: { messages: [...] }
```

---

## Dependencies

- Supabase CLI installed and working
- Docker for Supabase local
- GPU instance access
- Node.js for web app
- Python for Gateway

---

## Risks

| Risk | Mitigation |
|------|------------|
| GPU availability | Have backup provider ready |
| Supabase local issues | Document manual setup alternative |
| Model quality | Start with known good model |

---

## Timeline

Estimated: 2-3 weeks

| Week | Focus |
|------|-------|
| 1 | M1.1 + M1.2 (Infrastructure + Gateway) |
| 2 | M1.3 + M1.4 (Chat endpoint + Inference) |
| 3 | M1.5 + M1.6 (UI + Sessions) |

---

## Exit Criteria

Phase 1 complete when:
- Full chat flow works end-to-end
- Data persists correctly
- Basic UI is usable
- Ready to add Thinking mode


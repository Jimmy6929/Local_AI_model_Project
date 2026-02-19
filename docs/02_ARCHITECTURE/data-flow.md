---
tags:
  - architecture
cssclasses:
  - architecture-doc
---

# Data Flow

## Overview

This document traces data flow through the system for key operations.

For component details, see:
- [[system-overview]] — High-level architecture
- [[gateway-responsibilities]] — Control plane details
- [[routing-logic]] — How inference routing works

---

## Flow 1: User Authentication

```
User                    Web App                 Supabase Auth
  │                        │                         │
  │─── Enter credentials ──▶                         │
  │                        │─── Authenticate ───────▶│
  │                        │◀── JWT + User Info ─────│
  │                        │                         │
  │◀── Store JWT locally ──│                         │
  │                        │                         │
```

**Key Points:**
- [[auth-and-jwt|JWT]] stored in browser (httpOnly cookie or secure storage)
- JWT contains user_id and claims
- All subsequent requests include JWT in Authorization header

For authentication implementation, see [[auth-and-jwt]] and [[auth-ux]].

---

## Flow 2: Send Chat Message (Instant Mode)

```
User        Web App        Gateway           Supabase        Instant GPU
  │            │              │                  │                │
  │── Send ───▶│              │                  │                │
  │            │── POST /chat ─▶                 │                │
  │            │   (JWT + msg)│                  │                │
  │            │              │── Validate JWT ──▶                │
  │            │              │◀── user_id ──────│                │
  │            │              │                  │                │
  │            │              │── Store message ─▶                │
  │            │              │◀── OK ───────────│                │
  │            │              │                  │                │
  │            │              │── Inference ─────────────────────▶│
  │            │              │◀── Response ─────────────────────│
  │            │              │                  │                │
  │            │              │── Store response ▶                │
  │            │              │◀── OK ───────────│                │
  │            │              │                  │                │
  │            │◀── Response ─│                  │                │
  │◀── Display │              │                  │                │
```

**Key Points:**
- [[gateway-responsibilities|Gateway]] is the only component calling inference
- Messages stored before and after inference — see [[schema]]
- Response includes metadata (mode used, latency) — see [[logging-and-auditing]]

For Instant Mode details, see [[inference-instant]] and [[adr-0001-two-tier-inference]].

---

## Flow 3: Send Chat Message (Thinking Mode)

Same as Flow 2, except:
- User toggles "Deep Think" in UI — see [[chat-ux]]
- Request includes `mode: "think"` — see [[endpoints-contracts]]
- [[gateway-responsibilities|Gateway]] routes to [[inference-thinking|Thinking GPU]] instead
- Cold start may add 10-30 seconds — see [[cost-controls]]
- UI indicates thinking mode active

For Thinking Mode details, see [[inference-thinking]] and [[adr-0001-two-tier-inference]].

---

## Flow 4: RAG-Enhanced Chat

```
User        Web App        Gateway           Supabase        Inference
  │            │              │                  │                │
  │── Send ───▶│              │                  │                │
  │            │── POST /chat ─▶                 │                │
  │            │              │                  │                │
  │            │              │── Embed query ───▶                │
  │            │              │◀── Query vector ─│                │
  │            │              │                  │                │
  │            │              │── Vector search ─▶                │
  │            │              │◀── Top K chunks ─│                │
  │            │              │                  │                │
  │            │              │── Build prompt with context ──▶   │
  │            │              │◀── Response with grounding ─────│
  │            │              │                  │                │
  │            │◀── Response + citations ─       │                │
  │◀── Display │              │                  │                │
```

**Key Points:**
- Query is embedded using same model as document chunks — see [[embeddings-and-pgvector]]
- Vector search uses [[embeddings-and-pgvector|pgvector]] in [[Supabase]]
- Only top K most relevant chunks injected (minimal context) — see [[rag-flow]]
- [[rag-flow|Citations]] track which chunks were used

For RAG implementation details, see [[rag-flow]] and [[embeddings-and-pgvector]].

---

## Flow 5: File Upload and Processing

```
User        Web App        Gateway           Supabase Storage    Supabase DB
  │            │              │                    │                  │
  │── Upload ─▶│              │                    │                  │
  │            │── POST /files ▶                   │                  │
  │            │              │── Store file ─────▶│                  │
  │            │              │◀── File URL ───────│                  │
  │            │              │                    │                  │
  │            │              │── Extract text ────│                  │
  │            │              │── Chunk text ──────│                  │
  │            │              │── Embed chunks ────│                  │
  │            │              │                    │                  │
  │            │              │── Store chunks + embeddings ─────────▶│
  │            │              │◀── OK ───────────────────────────────│
  │            │              │                    │                  │
  │            │◀── Success ──│                    │                  │
  │◀── Confirm │              │                    │                  │
```

**Key Points:**
- Original file stored in [[storage-and-files|Supabase Storage]]
- Text extraction happens in [[gateway-responsibilities|Gateway]] (or worker)
- Chunks stored with embeddings in [[schema|document_chunks table]] — see [[embeddings-and-pgvector]]
- Processing is asynchronous for large files

For file handling details, see [[storage-and-files]] and [[file-upload-ux]].

---

## Flow 6: Tool Execution

```
User        Web App        Gateway           Tool Backend      Supabase
  │            │              │                   │                │
  │── Query ──▶│              │                   │                │
  │            │── POST /chat ─▶                  │                │
  │            │              │── Inference ──────│                │
  │            │              │◀── Tool call suggestion ─          │
  │            │              │                   │                │
  │            │              │── Execute tool ──▶│                │
  │            │              │◀── Tool result ───│                │
  │            │              │                   │                │
  │            │              │── Log tool call ──────────────────▶│
  │            │              │                   │                │
  │            │              │── Continue inference with result ─│
  │            │              │◀── Final response │                │
  │            │              │                   │                │
  │            │◀── Response ─│                   │                │
  │◀── Display │              │                   │                │
```

**Key Points:**
- Model suggests tool calls, does NOT execute — see [[tools-framework]]
- [[gateway-responsibilities|Gateway]] validates against [[rate-limiting-and-abuse|allowlist]]
- Gateway executes tool securely — see [[threat-model]]

For tool implementation details, see [[tools-framework]] and [[phase-4-tools]].
- Every tool call logged for audit
- Results fed back to model for final response


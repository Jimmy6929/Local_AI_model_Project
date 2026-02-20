---
tags:
  - readme
cssclasses:
  - readme-doc
---

# Step-by-Step Build Guide

## Overview

This guide walks through building the Local AI Assistant from start to finish, phase by phase.

For project context, see:
- [[overview]] — Project vision and architecture
- [[vision-and-goals]] — Why we're building this
- [[mvp-scope]] — What's in and out of scope
- [[system-overview]] — How components connect

---

## Before You Start

### Prerequisites
- macOS/Linux development machine
- Docker Desktop installed (for [[supabase-local-dev-plan|local Supabase]])
- Node.js 18+ installed (for [[ui-pages-and-components|Web App]])
- Python 3.10+ installed (for [[gateway-responsibilities|Gateway]])
- VS Code with Copilot
- Obsidian for documentation

### Accounts Needed
- Supabase (free tier for staging) — see [[adr-0002-supabase-local-then-cloud]]
- Cloud GPU provider (RunPod, Lambda Labs, etc.) — see [[inference-instant]], [[inference-thinking]]
- Vercel (for web app deployment) — see [[deployment-plan]]
- GitHub (for version control)

---

## Phase 1: Chat MVP

For phase context, see [[phase-1-chat-mvp]].

### Step 1.1: Set Up Local Supabase

**Actions:**
1. Install Supabase CLI
2. Navigate to project directory
3. Run `supabase init`
4. Run `supabase start`
5. Note the local credentials printed

**Verification:**
- Studio accessible at localhost:54323
- Can login to dashboard

**Reference:** [[04_DATABASE_SUPABASE/supabase-local-dev-plan]]

---

### Step 1.2: Create Database Schema

**Actions:**
1. Create migration: `supabase migration new init_schema`
2. Add profiles, chat_sessions, chat_messages tables — see [[schema]]
3. Enable RLS on all tables — see [[adr-0004-rls-multi-tenant-from-day-1]]
4. Add RLS policies — see [[rls-policies]]
5. Apply: `supabase db reset`

**Verification:**
- Tables visible in Studio
- RLS enabled (lock icon)

**Reference:** [[04_DATABASE_SUPABASE/schema]]

---

### Step 1.3: Set Up Gateway Project

**Actions:**
1. Create FastAPI project structure — see [[gateway-responsibilities]]
2. Add JWT validation middleware — see [[auth-and-jwt]]
3. Create health endpoint — see [[endpoints-contracts]]
4. Configure [[secrets-management|environment variables]]
5. Run locally

**Verification:**
- /health returns 200
- Invalid JWT returns 401

**Reference:** [[05_GATEWAY_API/gateway-responsibilities]]

---

### Step 1.4: Implement Chat Endpoint

**Actions:**
1. Create POST /chat endpoint — see [[endpoints-contracts]]
2. Validate JWT, extract user_id — see [[auth-and-jwt]]
3. Create/load session — see [[schema|sessions table]]
4. Store user message — see [[schema|messages table]]
5. Call inference (mock first) — see [[routing-logic]]
6. Store response
7. Return to client

**Verification:**
- Round-trip works with mock
- Messages appear in database

**Reference:** [[05_GATEWAY_API/endpoints-contracts]]

---

### Step 1.5: Connect Instant Inference

**Actions:**
1. Provision GPU pod — see [[inference-instant]]
2. Install model server (vLLM, TGI) — see [[adr-0003-openai-compatible-inference-interface]]
3. Load fast instruct model
4. Configure firewall (Tailscale or private) — see [[networking-and-private-access]]
5. Update [[gateway-responsibilities|Gateway]] to call real endpoint

**Verification:**
- Ask "What is 2+2?"
- Get sensible response

**Reference:** [[03_INFRA/inference-instant]]

---

### Step 1.6: Build Web App

**Actions:**
1. Create Next.js project — see [[ui-pages-and-components]]
2. Install Supabase client — see [[supabase-local-dev-plan]]
3. Build login page — see [[auth-ux]]
4. Build chat page — see [[chat-ux]]
5. Implement session sidebar
6. Connect to [[gateway-responsibilities|Gateway]] — see [[data-flow]]

**Verification:**
- Can login
- Can send message and see response
- Messages persist

**Reference:** [[06_WEB_APP/ui-pages-and-components]]

---

### Step 1.7: Complete Session Management

**Actions:**
1. Implement new session creation
2. Implement session switching
3. Implement session deletion
4. Add session titles

**Verification:**
- Multiple sessions work independently

---

## Phase 2: Two Modes

For phase context, see [[phase-2-two-modes]] and [[adr-0001-two-tier-inference]].

### Step 2.1: Deploy Thinking Endpoint

**Actions:**
1. Choose serverless GPU platform — see [[inference-thinking]]
2. Deploy stronger model
3. Configure scale-to-zero — see [[cost-controls]]
4. Set token limits

**Verification:**
- Can get response (cold start OK)

**Reference:** [[03_INFRA/inference-thinking]]

---

### Step 2.2: Add Gateway Routing

**Actions:**
1. Accept mode parameter
2. Route to appropriate endpoint
3. Record mode_used in message
4. Handle errors/timeouts

**Verification:**
- Both modes work through /chat

**Reference:** [[05_GATEWAY_API/routing-logic]]

---

### Step 2.3: Add UI Toggle

**Actions:**
1. Add Deep Think toggle to input — see [[chat-ux]]
2. Send mode in request — see [[endpoints-contracts]]
3. Show mode indicator on responses
4. Add thinking state animation

**Verification:**
- Toggle changes mode
- UI reflects which mode was used

---

## Phase 3: RAG

For phase context, see [[phase-3-rag]] and [[rag-flow]].

### Step 3.1: Implement File Upload

**Actions:**
1. Add POST /files endpoint — see [[endpoints-contracts]]
2. Configure storage bucket — see [[storage-and-files]]
3. Store file metadata in database — see [[schema]]
4. Build upload UI — see [[file-upload-ux]]

**Verification:**
- Files upload and appear in list

**Reference:** [[04_DATABASE_SUPABASE/storage-and-files]]

---

### Step 3.2: Build Processing Pipeline

**Actions:**
1. Extract text from files — see [[storage-and-files]]
2. Implement chunking — see [[rag-flow]]
3. Generate embeddings — see [[embeddings-and-pgvector]]
4. Store in document_chunks — see [[schema]]

**Verification:**
- Uploaded file becomes searchable chunks

**Reference:** [[04_DATABASE_SUPABASE/embeddings-and-pgvector]]

---

### Step 3.3: Add Context Retrieval

**Actions:**
1. Embed user query — see [[embeddings-and-pgvector]]
2. Vector search chunks — see [[rag-flow]]
3. Inject context into prompt — see [[gateway-responsibilities]]
4. Return citations — see [[chat-ux]]

**Verification:**
- Chat uses document context
- Citations shown in UI

**Reference:** [[05_GATEWAY_API/rag-flow]]

---

## Phase 4: Tools

For phase context, see [[phase-4-tools]] and [[tools-framework]].

### Step 4.1: Build Tool Framework

**Actions:**
1. Define tool schema
2. Implement tool registry
3. Add tool parsing
4. Build execution dispatcher

**Verification:**
- Framework accepts tool definitions

**Reference:** [[05_GATEWAY_API/tools-framework]]

---

### Step 4.2: Implement MVP Tools

**Actions:**
1. Implement web_search — see [[tools-framework]]
2. Implement save_note — see [[rag-flow|integrate with RAG]]
3. Implement list_tasks, add_task
4. Add tool logging — see [[logging-and-auditing]]

**Verification:**
- Each tool works via chat

---

### Step 4.3: Add Tool UI

**Actions:**
1. Show tool indicator on messages
2. Add expandable tool details
3. Display tool inputs/outputs

**Verification:**
- User can see tool usage

---

## Phase 5: Production

For phase context, see [[phase-5-saas-launch]] and [[environments-local-staging-prod]].

### Step 5.1: Set Up Production Infrastructure

**Actions:**
1. Create Supabase Cloud project — see [[adr-0002-supabase-local-then-cloud]]
2. Deploy web app to Vercel — see [[deployment-plan]]
3. Deploy [[gateway-responsibilities|Gateway]] to production
4. Configure DNS and SSL — see [[networking-and-private-access]]

**Verification:**
- All services accessible

**Reference:** [[03_INFRA/deployment-plan]]

---

### Step 5.2: Apply Security Hardening

**Actions:**
1. Complete security checklist — see [[threat-model]]
2. Configure rate limiting — see [[rate-limiting-and-abuse]]
3. Set up monitoring — see [[logging-and-auditing]]
4. Enable alerting — see [[cost-controls]]

**Verification:**
- All hardening items checked

**Reference:** [[07_SECURITY/production-hardening]]

---

### Step 5.3: Launch

**Actions:**
1. Final testing pass
2. Enable signups
3. Monitor first 24 hours
4. Respond to issues

**Verification:**
- 🚀 Users can sign up and chat

---

## Quick Reference

### Daily Development
```
# Start local Supabase
supabase start

# Start Gateway
cd gateway && uvicorn main:app --reload

# Start Web App
cd webapp && npm run dev
```

### Reset Database
```
supabase db reset
```

### Create Migration
```
supabase migration new <name>
```

### Check Logs
```
supabase logs
```

---

## Troubleshooting

### Auth Not Working
- Check SUPABASE_URL and keys match — see [[secrets-management]]
- Verify JWT secret is correct — see [[auth-and-jwt]]
- Check token expiration

### Inference Not Connecting
- Verify endpoint URL — see [[inference-instant]], [[inference-thinking]]
- Check firewall/Tailscale — see [[networking-and-private-access]]
- Test with curl directly

### RLS Blocking Everything
- Verify policies exist — see [[rls-policies]]
- Check user_id matches auth.uid() — see [[adr-0004-rls-multi-tenant-from-day-1]]
- Test in Studio SQL editor

---

## Next Steps After Launch

1. Collect user feedback — see [[success-metrics]]
2. Monitor costs and performance — see [[cost-controls]]
3. Prioritize improvements
4. Plan billing integration — see [[phase-5-saas-launch]]
5. Consider mobile app


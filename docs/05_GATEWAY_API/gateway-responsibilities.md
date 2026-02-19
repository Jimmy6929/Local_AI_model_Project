---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# Gateway Responsibilities

## Overview

The Gateway is the central brain of the system. It owns all business logic, security, and coordination.

For architectural context, see [[openclaw-style-gateway-notes]] and [[system-overview]].

---

## Core Philosophy

**Models are dumb. Gateway is smart.** — see [[openclaw-style-gateway-notes]]

- Models only receive prompts and return completions — see [[adr-0003-openai-compatible-inference-interface]]
- Gateway handles everything else:
  - [[auth-and-jwt|Authentication]]
  - [[rls-policies|Authorization]]
  - [[schema|Session management]]
  - [[rag-flow|Memory and RAG]]
  - [[tools-framework|Tool execution]]
  - [[routing-logic|Routing decisions]]
  - [[logging-and-auditing|Logging and audit]]

---

## Responsibility Matrix

| Responsibility | Owner | Description | Reference |
|----------------|-------|-------------|------------|
| [[auth-and-jwt|JWT Validation]] | Gateway | Verify every request | [[auth-and-jwt]] |
| User Identity | Gateway | Extract user_id from token | [[auth-and-jwt]] |
| [[schema|Session Management]] | Gateway | Create, load, update sessions | [[endpoints-contracts]] |
| [[schema|Message Storage]] | Gateway | Store user and assistant messages | [[schema]] |
| [[rag-flow|RAG Retrieval]] | Gateway | Embed query, search, inject context | [[embeddings-and-pgvector]] |
| [[routing-logic|Routing Decision]] | Gateway | Choose [[inference-instant|instant]] vs [[inference-thinking|thinking]] | [[adr-0001-two-tier-inference]] |
| Prompt Construction | Gateway | Build system + context + history | [[rag-flow]] |
| Inference Call | Gateway | Send to model, receive response | [[adr-0003-openai-compatible-inference-interface]] |
| [[tools-framework|Tool Parsing]] | Gateway | Detect tool calls in response | [[tools-framework]] |
| [[tools-framework|Tool Execution]] | Gateway | Run tools safely on backend | [[threat-model]] |
| [[schema|Response Storage]] | Gateway | Store assistant response | [[schema]] |
| [[logging-and-auditing|Audit Logging]] | Gateway | Log all actions for debugging | [[logging-and-auditing]] |
| [[rate-limiting-and-abuse|Rate Limiting]] | Gateway | Prevent abuse | [[rate-limiting-and-abuse]] |
| Error Handling | Gateway | Graceful degradation | [[production-hardening]] |

---

## Request Lifecycle

For detailed data flow, see [[data-flow]].

### Step-by-Step

1. **Receive Request**
   - POST /chat with message and session_id — see [[endpoints-contracts]]
   - Extract [[auth-and-jwt|JWT]] from Authorization header

2. **Authenticate**
   - Verify JWT signature — see [[auth-and-jwt]]
   - Check expiration
   - Extract user_id

3. **Load/Create Session**
   - If session_id provided, load [[schema|session]]
   - Verify user owns session ([[rls-policies|RLS]])
   - If no session_id, create new session

4. **Store User Message**
   - Insert into [[schema|chat_messages]]
   - user_id, session_id, role='user', content

5. **Retrieve Context (if [[rag-flow|RAG]] enabled)**
   - [[embeddings-and-pgvector|Embed]] user message
   - Vector search user's documents
   - Get top K relevant chunks

6. **Build Prompt**
   - System prompt (personality, instructions)
   - Context from [[rag-flow|RAG]] (if any)
   - Recent message history
   - Current user message

7. **Route to Inference**
   - Check mode ([[inference-instant|instant]] or [[inference-thinking|thinking]]) — see [[routing-logic]]
   - Select appropriate endpoint
   - Send prompt

8. **Receive Response**
   - Get model completion
   - Check for tool calls

9. **Execute Tools (if any)**
   - Parse tool call from response
   - Validate against allowlist
   - Execute tool
   - Log tool call
   - Feed result back to model (loop)

10. **Store Assistant Response**
    - Insert into chat_messages
    - role='assistant', mode_used, tokens_used

11. **Return to Client**
    - Response content
    - Metadata (mode_used, latency, citations)

---

## Error Handling

### Authentication Errors
| Error | Response | Action |
|-------|----------|--------|
| No token | 401 | Return immediately |
| Invalid token | 401 | Return immediately |
| Expired token | 401 | Client should refresh |

### Business Logic Errors
| Error | Response | Action |
|-------|----------|--------|
| Session not found | 404 | Create new or error |
| Session not owned | 403 | Return forbidden |
| Rate limited | 429 | Include retry-after |

### Inference Errors
| Error | Response | Action |
|-------|----------|--------|
| Model timeout | 504 | Suggest retry |
| Model error | 502 | Log, return generic error |
| Thinking cold start | (wait) | Client shows "thinking..." |

### Tool Errors
| Error | Response | Action |
|-------|----------|--------|
| Tool not allowed | 400 | Reject, log attempt |
| Tool execution failed | 500 | Return partial response |

---

## Security Responsibilities

### Input Validation
- Sanitize all user input
- Limit message length
- Validate session_id format
- Check file types for uploads

### Prompt Injection Defense
- System prompt is not user-modifiable
- Clear delimiter between system and user content
- Monitor for injection patterns (future)

### Tool Safety
- Explicit allowlist of tools
- No arbitrary code execution
- Backend-only execution
- Log every tool call

### Data Isolation
- Always filter by user_id
- Never trust client-provided user_id
- RLS as second layer of defense

---

## Observability

### Logging
- Log every request (anonymized)
- Log routing decisions
- Log tool executions
- Log errors with context

### Metrics
- Request count
- Latency (total, inference, tools)
- Error rate
- Mode distribution (instant vs thinking)

### Tracing (Future)
- Trace ID for each request
- Correlate logs across services
- Debug complex failures

---

## Configuration

### Environment Variables
| Variable | Purpose |
|----------|---------|
| SUPABASE_URL | Database connection |
| SUPABASE_SERVICE_KEY | Privileged operations |
| SUPABASE_JWT_SECRET | JWT verification |
| INSTANT_URL | Instant inference endpoint |
| THINKING_URL | Thinking inference endpoint |
| LOG_LEVEL | Logging verbosity |
| RATE_LIMIT_RPM | Requests per minute limit |

### Feature Flags (Future)
- ENABLE_RAG
- ENABLE_TOOLS
- ENABLE_STREAMING
- ENABLE_CITATIONS

---

## Non-Responsibilities

Things Gateway does NOT do:
- Serve static files (Web App handles)
- Manage user passwords (Supabase Auth handles)
- Run model inference (Inference tier handles)
- Make routing decisions based on model output (model is dumb)

---

## Checklist: Gateway Implementation

- [ ] JWT validation working
- [ ] Session CRUD operations
- [ ] Message storage working
- [ ] Routing logic implemented
- [ ] Instant inference connected
- [ ] Thinking inference connected
- [ ] Error handling comprehensive
- [ ] Logging in place
- [ ] Rate limiting configured
- [ ] Tool framework ready (Phase 4)
- [ ] RAG retrieval ready (Phase 3)


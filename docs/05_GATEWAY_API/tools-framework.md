---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# Tools Framework

## Overview

Allow the AI to take actions beyond text generation through a controlled tool system.

For roadmap context, see [[phase-4-tools]].
For security context, see [[threat-model]] and [[prompt-injection-notes]].

---

## Design Principles

### 1. Backend-Only Execution
- Model suggests tool calls — see [[openclaw-style-gateway-notes]]
- [[gateway-responsibilities|Gateway]] executes tools
- Model NEVER executes directly

### 2. Explicit Allowlist
- Only approved tools can run — see [[rate-limiting-and-abuse]]
- Unknown tools rejected
- [[logging-and-auditing|Logged]] for security audit

### 3. Minimal Scope
- Start with few essential tools — see [[mvp-scope]]
- Add carefully over time
- Each tool has clear purpose

### 4. Auditable
- Every tool call [[logging-and-auditing|logged]]
- Input and output recorded
- User can review history — see [[chat-ux]]

---

## MVP Tool Set

### web_search
**Purpose:** Search the web for current information

**Input:**
```
{
  "query": "string"
}
```

**Output:**
```
{
  "results": [
    {
      "title": "string",
      "url": "string",
      "snippet": "string"
    }
  ]
}
```

**Implementation Notes:**
- Use search API (SerpAPI, Brave, etc.)
- Rate limit to prevent abuse
- Cache results briefly

---

### save_note
**Purpose:** Save information for later retrieval

**Input:**
```
{
  "title": "string",
  "content": "string",
  "tags": ["string"]
}
```

**Output:**
```
{
  "note_id": "uuid",
  "saved": true
}
```

**Implementation Notes:**
- Store in Supabase (new notes table)
- User_id for RLS
- Searchable via RAG

---

### list_tasks
**Purpose:** Show user's task list

**Input:**
```
{
  "status": "all | pending | completed",
  "limit": 10
}
```

**Output:**
```
{
  "tasks": [
    {
      "id": "uuid",
      "title": "string",
      "due_date": "ISO8601 | null",
      "status": "pending | completed"
    }
  ]
}
```

---

### add_task
**Purpose:** Add a new task

**Input:**
```
{
  "title": "string",
  "due_date": "ISO8601 | null",
  "priority": "low | medium | high"
}
```

**Output:**
```
{
  "task_id": "uuid",
  "created": true
}
```

---

## Tool Execution Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    TOOL EXECUTION FLOW                      │
│                                                             │
│  1. Model returns response with tool_calls                  │
│                                                             │
│  2. Gateway parses tool_calls                               │
│     └─ Extract: tool_name, arguments                        │
│                                                             │
│  3. Gateway validates                                       │
│     └─ Tool in allowlist?                                   │
│     └─ Arguments valid?                                     │
│     └─ User has permission?                                 │
│                                                             │
│  4. Gateway executes tool                                   │
│     └─ Call appropriate backend function                    │
│     └─ Capture output or error                              │
│                                                             │
│  5. Gateway logs execution                                  │
│     └─ Store in tool_calls table                            │
│                                                             │
│  6. Gateway continues inference                             │
│     └─ Feed tool result back to model                       │
│     └─ Model generates final response                       │
│                                                             │
│  7. Return to user                                          │
│     └─ Include tool_calls in response metadata              │
└─────────────────────────────────────────────────────────────┘
```

---

## Tool Definition Format

### OpenAI Function Calling Style
```
{
  "type": "function",
  "function": {
    "name": "web_search",
    "description": "Search the web for current information",
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "description": "The search query"
        }
      },
      "required": ["query"]
    }
  }
}
```

### Tool Registry
- Store tool definitions centrally
- Include in system prompt
- Gateway knows how to execute each

---

## Safety Rules

### Allowlist Enforcement
```
ALLOWED_TOOLS = ["web_search", "save_note", "list_tasks", "add_task"]

if tool_name not in ALLOWED_TOOLS:
    log_blocked_tool(tool_name, user_id)
    return error("Tool not allowed")
```

### Input Validation
- Validate against JSON schema
- Reject malformed inputs
- Sanitize strings

### Rate Limiting
| Tool | Limit |
|------|-------|
| web_search | 10/minute |
| save_note | 30/minute |
| list_tasks | 60/minute |
| add_task | 30/minute |

### No Dangerous Tools
Never implement:
- Arbitrary code execution
- File system access (except controlled uploads)
- Network requests to arbitrary URLs
- Database queries (except through defined tools)

---

## Logging Schema

### Table: tool_calls
| Column | Type | Description |
|--------|------|-------------|
| id | uuid | Primary key |
| message_id | uuid | Associated chat message |
| user_id | uuid | Owner |
| tool_name | text | Which tool |
| input | jsonb | Tool input |
| output | jsonb | Tool output |
| status | text | success / error |
| execution_time_ms | integer | How long it took |
| created_at | timestamptz | When executed |

---

## User Visibility

### In Chat Response
```
{
  "content": "I found some results for...",
  "tool_calls": [
    {
      "tool": "web_search",
      "input": {"query": "..."},
      "status": "success"
    }
  ]
}
```

### In UI
- Show tool icon when tools were used
- Expandable to see tool details
- Link to full tool history (future)

---

## Future Tools

### Phase 4 Candidates
- complete_task
- update_task
- search_notes
- create_reminder

### Phase 5+ Candidates
- send_email (requires careful permissions)
- calendar_create
- external_api_call (with strict controls)

### Integration Tools (Future Multi-User)
- slack_post
- github_create_issue
- notion_create_page

---

## Checklist: Tools Implementation

- [ ] Tool allowlist defined
- [ ] Tool definitions for model
- [ ] web_search implemented
- [ ] save_note implemented
- [ ] list_tasks implemented
- [ ] add_task implemented
- [ ] Validation for each tool
- [ ] Logging to tool_calls table
- [ ] Rate limiting per tool
- [ ] Error handling graceful
- [ ] Tool results in response metadata


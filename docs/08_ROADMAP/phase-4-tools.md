---
tags:
  - roadmap
cssclasses:
  - roadmap-doc
---

# Phase 4: Tools

## Goal

Add [[tools-framework|tool execution]] capability for web search, note saving, and task management.

For implementation details, see [[tools-framework]].
For security context, see [[threat-model]] and [[prompt-injection-notes]].
Previous: [[phase-3-rag]] | Next: [[phase-5-production-launch]]

---

## Success Criteria

- [ ] Tools framework in Gateway working
- [ ] web_search tool functional
- [ ] save_note tool functional
- [ ] list_tasks and add_task tools functional
- [ ] Tool execution logged
- [ ] Tool calls visible in UI

---

## Scope

### In Scope
- [[tools-framework|Tool definition format]]
- Tool execution framework — see [[gateway-responsibilities]]
- MVP tools (web_search, notes, tasks)
- Tool [[logging-and-auditing|logging and audit]]
- [[chat-ux|UI display]] of tool usage

### Out of Scope
- Complex tools (email, calendar)
- User confirmation for tools
- Custom tool creation

---

## Milestones

### M4.1: Tool Framework
**Goal:** Gateway can execute tools

- Tool definition schema
- Tool registry
- Execution dispatcher
- Error handling

**Acceptance:** Framework ready for tool implementations

---

### M4.2: Tool Detection
**Goal:** Parse tool calls from model responses

- Define tool calling format
- Parse model output
- Validate against allowlist
- Queue for execution

**Acceptance:** Tool calls detected and validated

---

### M4.3: web_search Tool
**Goal:** Search the web for information

- Integrate search API
- Format results
- Return to model for response
- Rate limiting

**Acceptance:** "Search for AI news" returns results

---

### M4.4: Notes Tools
**Goal:** Save and retrieve notes

- Create notes table
- save_note implementation
- search_notes (simple)
- RLS on notes table

**Acceptance:** Can save and find notes via chat

---

### M4.5: Tasks Tools
**Goal:** Manage simple task list

- Create tasks table
- add_task implementation
- list_tasks implementation
- complete_task (optional)

**Acceptance:** Can manage tasks via chat

---

### M4.6: Tool Logging
**Goal:** Audit trail for all tool executions

- Create tool_calls table
- Log input, output, status
- Include execution time
- RLS for user isolation

**Acceptance:** All tool calls recorded

---

### M4.7: UI Integration
**Goal:** Show tool usage to user

- Display tool indicator on messages
- Expandable tool details
- Show tool input/output
- Error display

**Acceptance:** User sees when tools were used

---

## Technical Notes

### Tool Definition
```
{
  "name": "web_search",
  "description": "Search the web",
  "parameters": {
    "query": { "type": "string", "required": true }
  }
}
```

### Tool Execution Flow
```
Model Response → Parse Tool Calls → Validate → Execute → Log → Continue
```

### Safety Rules
- Backend execution only
- Explicit allowlist
- Input validation
- No arbitrary code

---

## Schema Additions

### notes
```
- id (uuid)
- user_id (FK)
- title
- content
- tags (array)
- created_at
```

### tasks
```
- id (uuid)
- user_id (FK)
- title
- due_date
- status (pending/completed)
- created_at
```

### tool_calls
```
- id (uuid)
- message_id (FK)
- user_id (FK)
- tool_name
- input (jsonb)
- output (jsonb)
- status
- execution_time_ms
- created_at
```

---

## Dependencies

- Phases 1-3 complete
- Search API access (SerpAPI, Brave, etc.)
- Tool definitions shared with model

---

## Risks

| Risk | Mitigation |
|------|------------|
| Tool abuse | Rate limiting, monitoring |
| Search API costs | Daily limits, caching |
| Model hallucinating tools | Strict parsing, allowlist |

---

## Timeline

Estimated: 2 weeks

| Week | Focus |
|------|-------|
| 1 | M4.1 + M4.2 + M4.3 (Framework + Detection + Search) |
| 2 | M4.4 + M4.5 + M4.6 + M4.7 (Notes + Tasks + Logging + UI) |

---

## Exit Criteria

Phase 4 complete when:
- All MVP tools working
- Tool calls logged
- UI shows tool usage
- Ready for production hardening


---
tags:
  - roadmap
cssclasses:
  - roadmap-doc
---

# Phase 2: Two Modes

## Goal

Add [[inference-thinking|Thinking Mode]] with stronger model, user can toggle between [[inference-instant|Instant]] and Think.

For architecture decision, see [[adr-0001-two-tier-inference]].
Previous: [[phase-1-chat-mvp]] | Next: [[phase-3-rag]]

---

## Success Criteria

- [ ] Thinking inference endpoint deployed
- [ ] User can toggle Deep Think mode
- [ ] Routing works correctly (instant vs think)
- [ ] UI shows which mode was used
- [ ] Cold start handled gracefully

---

## Scope

### In Scope
- [[inference-thinking|Serverless thinking endpoint]] setup
- [[routing-logic|Routing logic]] in [[gateway-responsibilities|Gateway]]
- [[chat-ux|Deep Think toggle]] in UI
- Mode indicator on responses
- Cold start UX — see [[cost-controls]]

### Out of Scope
- Automatic mode detection → future enhancement
- [[rag-flow|RAG]] → [[phase-3-rag]]
- [[tools-framework|Tools]] → [[phase-4-tools]]

---

## Milestones

### M2.1: Thinking Endpoint
**Goal:** Serverless GPU endpoint running

- Deploy stronger model on serverless platform
- Configure scale-to-zero
- Set token limits and timeouts
- Verify endpoint accessible

**Acceptance:** Can send request and get response (cold start OK)

---

### M2.2: Gateway Routing
**Goal:** Gateway routes to correct endpoint

- Accept mode parameter in /chat
- Route to instant or thinking based on mode
- Record mode_used in message
- Handle thinking mode errors/timeouts

**Acceptance:** Both modes working through same endpoint

---

### M2.3: UI Toggle
**Goal:** User can choose mode

- Deep Think toggle in chat input area
- Visual indicator when toggle is on
- Persist preference for session
- Clear when starting new session

**Acceptance:** Toggle changes mode, sends correct request

---

### M2.4: Response Metadata
**Goal:** Show mode information to user

- Display mode_used on each message
- Different visual treatment for thinking responses
- Show latency information (optional)

**Acceptance:** User can see which mode answered each message

---

### M2.5: Cold Start UX
**Goal:** Handle waiting gracefully

- "Thinking..." indicator while waiting
- Show expected wait time
- Allow cancel
- Graceful timeout handling

**Acceptance:** User understands why they're waiting

---

## Technical Notes

### Routing Logic
```
if request.mode == "think":
    endpoint = THINKING_URL
else:
    endpoint = INSTANT_URL

response = call_inference(endpoint, prompt)
response.mode_used = request.mode
```

### Message Schema Addition
```
chat_messages
+ mode_used (instant | think | null)
```

### Cost Controls
```
THINK_MAX_TOKENS = 4096
THINK_TIMEOUT = 300  # 5 minutes
THINK_DAILY_LIMIT = 20  # requests per user
```

---

## UX Details

### Toggle States
| State | Display |
|-------|---------|
| Off | Simple toggle, muted color |
| On | Brain icon, highlighted |
| Waiting | Animated, "Thinking..." |

### Response Display
```
[User] How do I design a database schema?

[Assistant - Thinking Mode 🧠]
Let me think through this carefully...
[detailed response]
⏱️ Took 45 seconds
```

---

## Dependencies

- Phase 1 complete
- Serverless GPU platform access
- Stronger model selected and tested

---

## Risks

| Risk | Mitigation |
|------|------------|
| Cold start too slow | Test, document, set expectations |
| Thinking mode too expensive | Strict limits, monitoring |
| Users confused by modes | Clear UX, documentation |

---

## Timeline

Estimated: 1-2 weeks

| Week | Focus |
|------|-------|
| 1 | M2.1 + M2.2 (Endpoint + Routing) |
| 2 | M2.3 + M2.4 + M2.5 (UI + Polish) |

---

## Exit Criteria

Phase 2 complete when:
- Both modes work reliably
- User can choose mode easily
- Cold start experience acceptable
- Cost controls in place
- Ready to add RAG


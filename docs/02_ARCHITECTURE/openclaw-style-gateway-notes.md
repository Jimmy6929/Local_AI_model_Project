---
tags:
  - architecture
cssclasses:
  - architecture-doc
---

# OpenClaw-Style Gateway Architecture

## Concept

Inspired by the "OpenClaw" pattern: a central control plane that coordinates everything, while keeping model servers as simple, stateless inference endpoints.

This is the architectural foundation for the [[gateway-responsibilities|Gateway]] component.

---

## Core Principle

**The Gateway owns the brain. Models are just muscles.**

- Models do not know about users — see [[auth-and-jwt]]
- Models do not store state — see [[schema]]
- Models do not execute tools — see [[tools-framework]]
- Models only receive prompts and return completions — see [[adr-0003-openai-compatible-inference-interface]]

All intelligence about [[routing-logic|routing]], [[threat-model|safety]], [[rag-flow|memory]], and [[tools-framework|tools]] lives in the Gateway.

---

## Why This Pattern?

### Flexibility
- Swap models without changing policies — see [[adr-0001-two-tier-inference]]
- Add new [[routing-logic|inference tiers]] without rewriting logic
- A/B test models transparently

### Security
- Single point for [[auth-and-jwt|authentication]]
- [[tools-framework|Tool execution]] happens in trusted backend — see [[threat-model]]
- [[prompt-injection-notes|Prompt injection defenses]] centralized

### Observability
- All requests flow through one point
- [[logging-and-auditing|Logging and audit]] centralized
- [[cost-controls|Cost tracking]] straightforward

### Multi-Channel Ready
- Future: add Telegram, Discord, Slack adapters — see [[phase-5-production-launch]]
- All adapters route into same [[gateway-responsibilities|Gateway]]
- Consistent behavior across channels

---

## Gateway Responsibilities (Full List)

See [[gateway-responsibilities]] for detailed implementation notes.

1. **Authenticate** — Validate [[auth-and-jwt|JWT]], extract user identity
2. **Authorize** — Check user permissions for requested action — see [[rls-policies]]
3. **Store Input** — Save user message to [[schema|database]]
4. **Retrieve Context** — [[rag-flow|RAG]]: embed query, search, fetch relevant chunks
5. **Build Prompt** — Construct system prompt + context + history
6. **Route Request** — Choose [[routing-logic|instant vs thinking tier]]
7. **Call Inference** — Send prompt to [[inference-instant|chosen]] [[inference-thinking|endpoint]]
8. **Parse Response** — Handle [[tools-framework|tool call suggestions]]
9. **Execute Tools** — Run approved [[tools-framework|tools]] on backend
10. **Continue Inference** — Feed tool results back if needed
11. **Store Output** — Save assistant response to [[schema|database]]
12. **Log Everything** — [[logging-and-auditing|Audit trail]] for debugging and compliance
13. **Return Response** — Send structured response to client — see [[endpoints-contracts]]

---

## Model Server Responsibilities (Minimal)

See [[inference-instant]] and [[inference-thinking]] for infrastructure details.

1. Receive prompt via [[adr-0003-openai-compatible-inference-interface|OpenAI-compatible API]]
2. Run inference
3. Return completion (possibly with [[tools-framework|tool call suggestions]])
4. That's it. No state, no memory, no decisions.

---

## Diagram: Control Flow

```
┌────────────────────────────────────────────────────────────┐
│                        GATEWAY                             │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ 1. Auth        │ 2. Session     │ 3. RAG Retrieve   │ │
│  ├──────────────────────────────────────────────────────┤ │
│  │ 4. Route       │ 5. Call Model  │ 6. Parse Response │ │
│  ├──────────────────────────────────────────────────────┤ │
│  │ 7. Tool Exec   │ 8. Store       │ 9. Log & Return   │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
            │                               │
            ▼                               ▼
     ┌─────────────┐                 ┌─────────────┐
     │ Instant GPU │                 │ Thinking GPU│
     │  (Dumb)     │                 │  (Dumb)     │
     └─────────────┘                 └─────────────┘
```

---

## Future: Channel Adapters

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│ Telegram │   │ Discord  │   │  Slack   │
│ Adapter  │   │ Adapter  │   │ Adapter  │
└────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
                    ▼
              ┌───────────┐
              │  GATEWAY  │
              └───────────┘
```

Each adapter:
- Receives messages from platform API
- Authenticates user (maps platform user to internal user)
- Calls Gateway with standardized request
- Formats Gateway response for platform

---

## Implications for Development

### Gateway is the "hard" part
- Most business logic lives here
- Security-critical code
- Must be well-tested

### Inference is "easy" to operate
- Standard deployment pattern
- Commodity infrastructure
- Easy to scale or swap

### Tools are "trusted" backend code
- Never exposed to model directly
- Allowlist enforced
- Audit logged

---

## Comparison: Alternative Patterns

| Pattern | Pros | Cons |
|---------|------|------|
| **Gateway Central** (this project) | Flexible, secure, observable | Single point of failure |
| **Smart Model** (model owns logic) | Simpler infra | Model lock-in, security risks |
| **Agent Framework** (LangChain, etc.) | Rapid prototyping | Harder to audit, vendor dependency |

This project chooses **Gateway Central** for maximum control and production-readiness.


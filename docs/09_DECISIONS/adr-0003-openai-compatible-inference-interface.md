---
tags:
  - decision
cssclasses:
  - decision-doc
---

# ADR-0003: OpenAI-Compatible Inference Interface

## Status
Accepted

## Context

We need to interface with multiple AI models ([[inference-instant|Instant]] and [[inference-thinking|Thinking]] tiers). Options:
1. Native APIs per model provider
2. Custom unified API
3. OpenAI-compatible API

The chosen interface affects:
- Model swappability — see [[adr-0001-two-tier-inference]]
- Development complexity
- Tooling compatibility
- Future flexibility

Related documents:
- [[inference-instant]] — Instant tier setup
- [[inference-thinking]] — Thinking tier setup
- [[gateway-responsibilities]] — Gateway interface

## Decision

Use **OpenAI-compatible API** (`/v1/chat/completions`) for all [[inference-instant|inference]] [[inference-thinking|endpoints]].

Both [[inference-instant|Instant]] and [[inference-thinking|Thinking]] tiers expose the same API shape:
- Standard request format
- Standard response format
- [[tools-framework|Tool calling]] support

## Rationale

### Why OpenAI-Compatible

**Industry Standard**
- Most model servers support it (vLLM, TGI, Ollama, etc.)
- Well-documented specification
- Wide tooling support

**Model Swappability**
- Change models without changing Gateway code
- A/B testing easier
- Upgrade path clear

**Tooling Ecosystem**
- Existing libraries work
- Debugging tools available
- Community knowledge

### Why Not Native APIs

- Each model has different API
- Gateway must handle multiple formats
- Testing becomes complex
- Harder to swap models

### Why Not Custom API

- Must design and document
- Must maintain
- No community support
- Reinventing the wheel

## Consequences

### Positive
- Easy model swapping
- Standard request/response handling
- Existing tooling works
- Community documentation

### Negative
- Limited to what OpenAI API supports
- Some features may require extensions
- Version compatibility considerations

### Neutral
- Must run OpenAI-compatible server
- Must map model names
- Must handle slight variations

## API Shape

### Request
```
POST /v1/chat/completions
{
  "model": "instant-model",
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."}
  ],
  "max_tokens": 2048,
  "temperature": 0.7,
  "tools": [...]  // optional
}
```

### Response
```
{
  "id": "...",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "...",
      "tool_calls": [...]  // optional
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 100,
    "completion_tokens": 50
  }
}
```

## Implementation Notes

- Use vLLM, TGI, or similar for serving
- Configure model name mapping
- Handle streaming if needed
- Monitor for API compatibility updates

## Related Decisions

- ADR-0001: Two-tier inference architecture


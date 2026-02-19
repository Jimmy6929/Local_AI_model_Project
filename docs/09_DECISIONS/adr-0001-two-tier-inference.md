---
tags:
  - decision
cssclasses:
  - decision-doc
---

# ADR-0001: Two-Tier Inference Architecture

## Status
Accepted

## Context

We need to run AI inference for the personal assistant. Options include:
1. Single model for everything
2. Two-tier approach (fast + strong)
3. Multiple specialized models

The system must balance:
- Response latency for daily use — see [[success-metrics]]
- Capability for complex tasks
- Infrastructure cost — see [[cost-controls]]
- Operational complexity

Related documents:
- [[inference-instant]] — Tier 1 implementation
- [[inference-thinking]] — Tier 2 implementation
- [[routing-logic]] — How routing works
- [[phase-2-two-modes]] — Roadmap phase

## Decision

Implement a **two-tier inference architecture**:

**Tier 1: [[inference-instant|Instant Mode]]**
- Fast, smaller model
- Always-on GPU pod
- Used for most requests
- Low latency priority

**Tier 2: [[inference-thinking|Thinking Mode]]**
- Stronger, larger model
- Serverless (scale-to-zero)
- Used for complex tasks
- Cold start acceptable

## Rationale

### Why Two Tiers

**Cost Efficiency**
- Instant tier handles 90%+ of requests affordably
- Thinking tier costs only when used
- Scale-to-zero prevents idle spending

**User Experience**
- Daily tasks feel responsive
- Complex tasks get better answers
- User controls the tradeoff

**Operational Simplicity**
- Two models is manageable
- Clear routing logic
- Easier to optimize each tier

### Why Not Single Model

- Strong models are expensive to run always-on
- Weak models frustrate users on complex tasks
- No flexibility in cost/capability tradeoff

### Why Not Multiple Models

- Routing complexity increases
- Operational burden increases
- Diminishing returns for personal use

## Consequences

### Positive
- Responsive experience for daily use — see [[success-metrics]]
- Strong capability available when needed
- [[cost-controls|Cost]] scales with actual usage
- Clear mental model for users — see [[chat-ux]]

### Negative
- Cold start latency for [[inference-thinking|thinking mode]]
- Two models to maintain/update
- [[routing-logic|Routing logic]] to implement
- User must understand modes — see [[chat-ux]]

### Neutral
- Need to select appropriate models for each tier
- Need to define routing rules
- Usage tracking becomes more important

## Implementation Notes

- Gateway handles routing, models are stateless
- Use OpenAI-compatible API for both tiers
- Track mode_used for each message
- Monitor costs per tier separately

## Related Decisions

- ADR-0003: OpenAI-compatible inference interface


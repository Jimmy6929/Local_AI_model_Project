---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# Routing Logic

## Overview

How [[gateway-responsibilities|Gateway]] decides which inference tier to use for each request.

For tier details, see:
- [[inference-instant]] — Fast, always-on tier
- [[inference-thinking]] — Stronger, scale-to-zero tier
- [[adr-0001-two-tier-inference]] — Architecture decision

---

## Routing Decision Tree

```
Request received
       │
       ▼
┌──────────────────┐
│ Mode explicitly  │
│ specified?       │
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
   Yes        No
    │         │
    ▼         ▼
┌────────┐  ┌──────────────────┐
│ Use    │  │ Apply automatic  │
│ that   │  │ routing rules    │
│ mode   │  └────────┬─────────┘
└────────┘           │
                     ▼
              ┌──────────────────┐
              │ Default: Instant │
              │ Unless triggers  │
              │ hit (Phase 2+)   │
              └──────────────────┘
```

---

## MVP Routing (Phase 1)

See [[phase-1-chat-mvp]] and [[mvp-scope]] for MVP context.

### Simple Rule
- If `mode: "think"` → [[inference-thinking|Thinking tier]]
- Otherwise → [[inference-instant|Instant tier]]

### No Automatic Detection
- User explicitly chooses via [[chat-ux|toggle]]
- Simple and predictable
- No surprises

---

## Advanced Routing (Phase 2+)

### Automatic Triggers

Consider thinking mode when:

| Trigger | Example | Confidence |
|---------|---------|------------|
| Explicit keywords | "think deeply", "analyze carefully" | High |
| Complex math | Multi-step calculations | Medium |
| Long-form writing | "Write a 2000 word essay" | Medium |
| Code architecture | "Design a system for..." | Medium |
| Comparison/analysis | "Compare these three approaches" | Medium |

### Implementation Options

**Option A: Keyword Detection**
- Simple string matching
- Low overhead
- Limited accuracy

**Option B: Classifier Model**
- Small model classifies query
- Better accuracy
- Adds latency

**Option C: User Learns**
- Suggest thinking mode for certain queries
- User confirms or rejects
- Build personalized patterns

### Recommendation for Phase 2
Start with keyword detection. Simple and transparent.

---

## Routing Configuration

### Environment Variables
```
ROUTING_DEFAULT_MODE=instant
ROUTING_AUTO_DETECT=false  # Enable in Phase 2
ROUTING_KEYWORDS_THINK=["think deeply", "analyze", "compare"]
```

### Per-Request Override
```
{
  "mode": "think"  // Explicit override
}
```

---

## Mode Selection UI

### User Experience
- Default: Instant (no indicator needed)
- Toggle: "Deep Think" switch
- Visual feedback: mode indicator on response

### UI States
| State | Display |
|-------|---------|
| Instant (default) | Normal send button |
| Thinking (toggled) | Brain icon, "Thinking..." |
| Auto-detected | "Suggested: Deep Think" |

---

## Fallback Behavior

### Thinking Mode Unavailable
- If thinking endpoint is down
- Fallback to instant with warning
- Log the fallback

### High Latency Warning
- If thinking mode cold start > 30s
- Show progress indicator
- Allow cancel

---

## Monitoring Routing

### Metrics to Track
| Metric | Purpose |
|--------|---------|
| Mode distribution | Usage patterns |
| Auto-detect accuracy | Improve triggers |
| Mode switch rate | User satisfaction |
| Fallback frequency | Reliability |

### Logging
```
{
  "request_id": "...",
  "requested_mode": "instant",
  "auto_detected": false,
  "final_mode": "instant",
  "routing_reason": "explicit"
}
```

---

## Cost Implications

### Thinking Mode Costs More
- Larger model = more tokens = more cost
- Track per-user thinking usage
- Consider limits for free tier (future SaaS)

### Cost Controls
| Control | Implementation |
|---------|----------------|
| Daily think limit | Max 20 thinking requests/day |
| Token cap | Max 4096 tokens per think request |
| Spending alert | Notify at 80% of daily budget |

---

## Future: Smart Routing

### Learn from User
- Track when user overrides auto-detect
- Build user-specific patterns
- Personalized routing

### Learn from Outcomes
- Track response quality (thumbs up/down)
- Correlate with mode used
- Improve trigger rules

### Hybrid Mode
- Start with instant for speed
- If response quality low, retry with thinking
- Show both options to user

---

## Checklist: Routing Implementation

### Phase 1 (MVP)
- [ ] Honor explicit mode parameter
- [ ] Default to instant when not specified
- [ ] Log mode used in response

### Phase 2
- [ ] Add keyword detection
- [ ] UI suggests thinking mode
- [ ] User can accept/reject suggestion
- [ ] Track suggestion accuracy

### Future
- [ ] Classifier model for detection
- [ ] User preference learning
- [ ] Hybrid mode option


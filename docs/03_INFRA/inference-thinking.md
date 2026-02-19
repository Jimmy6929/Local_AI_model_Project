---
tags:
  - infrastructure
cssclasses:
  - infra-doc
---

# Thinking Inference

## Overview

The serverless GPU endpoint for complex reasoning, activated only when needed.

Part of the [[adr-0001-two-tier-inference|two-tier inference architecture]].

---

## Purpose

- Handle difficult problems that require stronger reasoning
- Used when user toggles "[[chat-ux|Deep Think]]" mode
- Cold start acceptable (user expects to wait)
- Cost-optimized via scale-to-zero — see [[cost-controls]]

---

## Infrastructure Requirements

### Serverless GPU Platform
- Endpoint scales to zero when idle
- Spins up on request (cold start 10-30 seconds)
- Supports larger models (30B-70B parameter range)
- Examples: RunPod Serverless, Modal, Replicate, Banana

### Model Requirements
- Stronger reasoning capabilities
- Can handle complex coding, analysis, planning
- Fits within serverless memory limits
- Acceptable inference time (2-5 minutes for complex requests)

---

## Operational Design

### Scale-to-Zero
- No cost when idle
- Cold start on first request
- Configurable minimum/maximum workers

### Cost Controls
See [[cost-controls]] for overall budget strategy.
| Control | Setting |
|---------|---------|
| Max tokens per request | 4096 |
| Timeout per request | 5 minutes |
| Max concurrent requests | 2 |
| Daily spending cap | $10 (example) |

### Cold Start Mitigation Options
1. **Accept cold start** — User understands "thinking" takes time — see [[chat-ux]]
2. **Keep 1 warm worker** — Small idle cost, instant response — see [[cost-controls]]
3. **Predictive warm-up** — Warm worker when user opens app (complex)

Recommended for MVP: Accept cold start, show clear "thinking..." indicator — see [[mvp-scope]].

---

## Model Selection Criteria

| Factor | Requirement |
|--------|-------------|
| Tasks | Complex reasoning, difficult coding, deep analysis |
| Size | 30B-70B range (stronger than Instant tier) |
| License | Permissive for commercial use |
| Quality | Significantly better than Instant model on hard tasks |
| Speed | Acceptable (2-5 min for complex requests) |

### Candidate Model Classes
- Llama 70B / Qwen 72B (if VRAM allows)
- DeepSeek Coder (for coding-heavy use)
- Mixtral 8x22B (MoE for efficiency)

---

## API Interface

### Endpoint Contract
- Same as [[inference-instant|Instant]]: [[adr-0003-openai-compatible-inference-interface|OpenAI-compatible]] `/v1/chat/completions`
- [[gateway-responsibilities|Gateway]] treats both tiers identically
- Only difference: [[routing-logic|routing decision]] at Gateway

### Configuration Defaults
| Parameter | Default Value |
|-----------|---------------|
| max_tokens | 4096 |
| temperature | 0.5 (more focused) |
| top_p | 0.9 |
| stream | false (batch response OK) |

---

## User Experience

See [[chat-ux]] for detailed UI specifications.

### UI Indicators
- "Deep Think" toggle clearly visible — see [[chat-ux]]
- When active: show "Thinking deeply..." with spinner
- Show estimated wait time (if cold start)
- Response shows "Thinking Mode" badge

### Expectation Setting
- User knows this mode is for complex tasks
- User expects to wait 30 seconds to several minutes
- UI explains the tradeoff (stronger but slower)

---

## Security Considerations

- Endpoint must NOT be public
- Only Gateway can reach this endpoint
- Stricter token limits (expensive model)
- Log usage for cost attribution

---

## Monitoring Plan

### Metrics to Track
- Request count (track usage growth)
- Cold start frequency and duration
- Inference latency (p50, p95)
- Token usage per request
- Cost per request

### Alerting
- Alert if daily spending approaches cap
- Alert if cold start > 60 seconds
- Alert if error rate > 10%

---

## Acceptance Criteria

- [ ] Endpoint scales to zero when idle
- [ ] Cold start under 30 seconds
- [ ] Complex reasoning quality noticeably better than Instant
- [ ] Token limits enforced
- [ ] Timeout kills runaway requests
- [ ] Cost tracking in place
- [ ] Only Gateway can reach endpoint


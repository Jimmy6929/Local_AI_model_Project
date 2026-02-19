---
tags:
  - infrastructure
cssclasses:
  - infra-doc
---

# Instant Inference

## Overview

The always-on GPU pod that handles most daily tasks with low latency.

Part of the [[adr-0001-two-tier-inference|two-tier inference architecture]].

---

## Purpose

- Fast response for coding, writing, planning, general Q&A
- Default inference tier (used unless user toggles [[chat-ux|Thinking Mode]])
- Must feel responsive (first token under 2 seconds) — see [[success-metrics]]

---

## Infrastructure Requirements

### GPU Selection Criteria
- Sufficient VRAM for chosen model (24GB+ recommended)
- Good price-to-performance ratio
- Availability in preferred region

### Runtime Environment
- Container or VM with:
  - GPU drivers installed
  - Model serving framework
  - OpenAI-compatible API server
  - Process manager for auto-restart

### Networking
- [[networking-and-private-access|Private endpoint]] (not public internet)
- Only [[gateway-responsibilities|Gateway]] can reach this endpoint
- Firewall rules: allow only VPN subnet (dev) or internal network (prod) — see [[environments-local-staging-prod]]

---

## Operational Design

### Always-On Strategy
- Pod runs continuously during active development hours
- Manual or scheduled stop when idle (overnight, weekends)
- Health check endpoint for monitoring

### Cost Management
- Monitor GPU-hours weekly — see [[cost-controls]]
- Set spending alerts at 80% of budget — see [[cost-controls]]
- Consider spot/preemptible instances if interruption acceptable

### Reliability
- Process manager (systemd, supervisord, or similar)
- Auto-restart on crash
- Log rotation to prevent disk fill — see [[logging-and-auditing]]
- Health endpoint: `/health` returns 200 if ready

---

## Model Selection Criteria

| Factor | Requirement |
|--------|-------------|
| Tasks | Coding, writing, planning, summarization |
| Size | Must fit in available VRAM |
| License | Permissive for commercial use (future SaaS) |
| Format | GGUF, AWQ, or native weights |
| Speed | Optimized for inference (quantized OK) |

### Candidate Model Classes
- 7B-13B instruct models (fast, good quality)
- Code-specialized models (if coding is primary use)
- Mistral, Llama, Qwen families (check license)

---

## API Interface

### Endpoint Contract
- [[adr-0003-openai-compatible-inference-interface|OpenAI-compatible]] `/v1/chat/completions`
- Supports system, user, assistant messages
- Supports [[tools-framework|tool definitions]] (if used)
- Returns streamed or complete responses — see [[chat-ux]]

### Configuration Defaults
| Parameter | Default Value |
|-----------|---------------|
| max_tokens | 2048 |
| temperature | 0.7 |
| top_p | 0.9 |
| stream | true (for UI) |

---

## Security Considerations

- Endpoint must NOT be public — see [[networking-and-private-access]]
- No authentication at model layer ([[gateway-responsibilities|Gateway]] handles [[auth-and-jwt|auth]])
- [[rate-limiting-and-abuse|Rate limiting]] at Gateway level, not here
- Logs should not contain sensitive user data — see [[threat-model]]

See also: [[inference-thinking]] for the Thinking tier.

---

## Monitoring Plan

### Metrics to Track
- Request count (daily, hourly)
- Latency (p50, p95, p99)
- GPU utilization
- Memory usage
- Error rate

### Alerting
- Alert if error rate > 5% for 5 minutes
- Alert if latency p95 > 5 seconds
- Alert if GPU pod unreachable

---

## Acceptance Criteria

- [ ] Model loads and responds correctly
- [ ] First token latency < 2 seconds
- [ ] Auto-restart on crash works
- [ ] Only Gateway can reach endpoint
- [ ] Logs rotate and don't fill disk
- [ ] Cost tracking in place


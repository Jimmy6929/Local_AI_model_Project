---
tags:
  - infrastructure
cssclasses:
  - infra-doc
---

# Cost Controls

## Overview

Strategies to manage infrastructure costs and prevent runaway spending.

For budget targets, see [[success-metrics]].

---

## Cost Categories

| Category | Driver | Control Strategy |
|----------|--------|------------------|
| [[inference-instant|Instant GPU]] | Hourly rental | Schedule on/off, monitor utilization |
| [[inference-thinking|Thinking GPU]] | Per-request billing | Token caps, concurrency limits |
| [[Supabase]] | Storage + compute | Monitor growth, clean unused data |
| Bandwidth | Data transfer | CDN caching, minimize payloads |
| [[embeddings-and-pgvector|Embeddings]] | API calls or compute | Batch processing, cache results |

---

## Instant Inference Costs

See [[inference-instant]] for infrastructure details.

### Cost Model
- Hourly rental for GPU pod
- Paying whether used or idle
- Example: $0.30-0.80/hour depending on GPU

### Control Strategies

**Manual Shutdown**
- Stop pod when done for the day
- Weekend shutdown if not using
- Simple but requires discipline

**Scheduled Shutdown**
- Cron job to stop at midnight, start at 9am
- Automated, but may miss off-hours use

**Idle Detection**
- Monitor request rate
- Auto-stop after 2 hours of no requests
- Requires additional tooling

### Budget Guardrails
| Metric | Target | Alert At |
|--------|--------|----------|
| Daily GPU hours | 12 hours | 10 hours |
| Weekly GPU spend | $50 | $40 |
| Monthly GPU spend | $150 | $120 |

---

## Thinking Inference Costs

See [[inference-thinking]] for infrastructure details.

### Cost Model
- Pay per request (token-based or time-based)
- Scale-to-zero means no idle cost
- Cold start overhead included in request cost

### Control Strategies

**Token Limits**
- Max 4096 output tokens per request
- Reject requests exceeding limit
- Return error with guidance

**Timeout Limits**
- Max 5 minutes per request
- Kill runaway inference
- Return partial or error

**Concurrency Limits**
- Max 2 concurrent requests
- Queue additional requests
- Reject if queue too long

**Daily Spending Cap**
- Hard limit: $10/day
- Disable [[inference-thinking|thinking mode]] if cap reached
- Alert user in UI — see [[chat-ux]]

### Budget Guardrails
| Metric | Target | Alert At |
|--------|--------|----------|
| Daily thinking requests | 20 | 15 |
| Daily thinking spend | $10 | $8 |
| Per-request cost | $0.50 avg | $1.00 |

---

## Supabase Costs

### Cost Model (Free Tier Limits)
- 500MB database
- 1GB storage
- 2GB bandwidth
- 50K monthly active users

### Control Strategies

**Monitor Database Size**
- Alert at 80% of limit
- Archive old chat sessions
- Delete test data regularly

**Monitor Storage**
- Alert at 80% of limit
- Compress documents before storage
- Delete unused files

**Upgrade Strategy**
- Move to Pro plan when approaching limits
- Budget $25/month for Supabase Pro

---

## Embedding Costs

### If Using External API
- OpenAI embeddings: ~$0.0001 per 1K tokens
- Budget per document uploaded
- Batch embed on upload, not on query

### If Self-Hosted
- Same GPU as Instant can run embeddings
- No additional cost, but adds latency
- Async processing recommended

---

## Total Monthly Budget (Private Use)

| Component | Target | Max |
|-----------|--------|-----|
| Instant GPU | $100 | $150 |
| Thinking GPU | $30 | $50 |
| Supabase | $0-25 | $25 |
| Bandwidth | $0-10 | $10 |
| Embeddings | $0-5 | $10 |
| **Total** | **$130-170** | **$245** |

---

## Alerting Setup

### Alert Channels
- Email for non-urgent
- Slack/Discord for urgent
- SMS for critical (optional)

### Alert Triggers
| Trigger | Severity | Action |
|---------|----------|--------|
| 80% daily GPU budget | Warning | Review usage |
| 100% daily GPU budget | Critical | Investigate immediately |
| Thinking mode disabled | Info | Notify user |
| Database 80% full | Warning | Plan cleanup or upgrade |
| Unknown charges | Critical | Audit immediately |

---

## Cost Tracking Dashboard (Future)

MVP can skip this, but for SaaS:
- Per-user usage tracking
- Daily/weekly/monthly summaries
- Projected end-of-month cost
- Breakdown by feature (chat, thinking, storage)

---

## Checklist: Cost Controls

- [ ] Instant GPU shutdown schedule configured
- [ ] Thinking mode token limits set
- [ ] Thinking mode spending cap set
- [ ] Supabase usage monitored
- [ ] Alerting configured for budget thresholds
- [ ] Monthly budget reviewed weekly


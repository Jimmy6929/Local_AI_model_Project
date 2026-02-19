---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# Rate Limiting and Abuse Prevention

## Overview

Protect the system from abuse, ensure fair usage, and control [[cost-controls|costs]].

For security context, see [[threat-model]] and [[production-hardening]].

---

## Why Rate Limiting?

### Prevent Abuse
- Block automated attacks
- Stop credential stuffing
- Prevent resource exhaustion

### Control Costs
- Limit expensive operations — see [[cost-controls]]
- Prevent runaway spending — see [[inference-thinking]]
- Enable usage-based billing (future) — see [[phase-5-saas-launch]]

### Fair Usage
- One user can't monopolize resources
- Ensure availability for all users
- Predictable performance

---

## Rate Limit Strategy

### Per-User Limits
- Based on authenticated user_id
- Fair allocation per user
- Scales with user count

### Per-Endpoint Limits
- Different limits for different operations
- Expensive operations have lower limits
- Read operations have higher limits

### Tiered Limits (Future SaaS)

See [[phase-5-saas-launch]] and [[adr-0004-rls-multi-tenant-from-day-1]] for multi-tenant context.

| Tier | Chat ([[inference-instant|instant]]) | Chat ([[inference-thinking|think]]) | [[storage-and-files|Files]] |
|------|----------------|--------------|-------|
| Free | 30/hour | 5/hour | 10/day |
| Pro | 300/hour | 50/hour | 100/day |
| Team | 1000/hour | 200/hour | 500/day |

---

## Current Limits (MVP)

See [[mvp-scope]] for MVP context.

### Per-User Limits
| Endpoint | Limit | Window |
|----------|-------|--------|
| POST /chat ([[inference-instant|instant]]) | 60 | per minute |
| POST /chat ([[inference-thinking|think]]) | 10 | per minute |
| POST /files | 10 | per minute — see [[file-upload-ux]] |
| GET endpoints | 120 | per minute |

### Global Limits
| Resource | Limit |
|----------|-------|
| Concurrent inference | 10 |
| File upload size | 50MB |
| Message length | 10,000 chars |

---

## Implementation

### Token Bucket Algorithm
- Each user has a bucket of tokens
- Tokens replenish at constant rate
- Request consumes tokens
- Rejected when bucket empty

### Redis-Based Tracking
- Store user token counts in Redis
- Fast reads and writes
- Automatic expiration

### Alternative: Sliding Window
- Count requests in rolling time window
- More accurate than fixed windows
- Slightly more complex

---

## Response Headers

### Include in Every Response
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1700000060
```

### When Rate Limited
```
HTTP 429 Too Many Requests

{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests",
    "retry_after_seconds": 45
  }
}
```

---

## Abuse Detection

### Patterns to Watch
| Pattern | Indicator | Action |
|---------|-----------|--------|
| Rapid fire | >10 req/sec | Rate limit |
| Large payloads | >1MB messages | Reject |
| Invalid auth spam | >10 failures/min | Temp block |
| Prompt injection | Known patterns | Log, monitor |
| Scraping | Sequential resource access | Rate limit |

### Automated Blocking
- Temporary blocks (1 hour)
- Based on behavior patterns
- Alert for review

### Manual Review
- Flagged accounts
- Suspicious activity
- Appeal process

---

## IP-Based Limits

### When to Use
- Pre-authentication (login attempts)
- Anonymous endpoints
- Global protection

### Caution
- Shared IPs (corporate, VPN)
- Mobile networks
- May affect legitimate users

### Implementation
- Higher limits than per-user
- Primarily for auth endpoints
- Example: 100 login attempts/IP/hour

---

## Graceful Degradation

### When Overloaded
1. Return 429 with clear retry time
2. Serve cached responses if possible
3. Queue requests (if appropriate)
4. Prioritize paid users (future)

### Messaging
- Clear error messages
- Suggested wait time
- Link to documentation

---

## Monitoring

### Metrics
| Metric | Purpose |
|--------|---------|
| Rate limit hits | Usage patterns |
| 429 response rate | User impact |
| Blocked IPs | Attack detection |
| Per-user usage | Identify heavy users |

### Alerts
| Trigger | Severity |
|---------|----------|
| >10% requests rate limited | Warning |
| Sudden spike in requests | Warning |
| Single user >50% of traffic | Warning |
| Potential DDoS pattern | Critical |

---

## User Communication

### In UI
- Show remaining quota
- Warn when approaching limit
- Clear message when limited

### Documentation
- Published rate limits
- Best practices for clients
- Contact for limit increases

---

## Future: Usage-Based Pricing

### Track Per-User
- Requests count
- Tokens used
- Storage used
- Thinking mode usage

### Billing Integration
- Report usage to billing system
- Alert at tier thresholds
- Enable/disable based on payment

---

## Checklist: Rate Limiting

- [ ] Per-user limits implemented
- [ ] Per-endpoint limits configured
- [ ] Rate limit headers in responses
- [ ] 429 response handling
- [ ] Redis or similar for tracking
- [ ] IP-based limits for auth
- [ ] Monitoring dashboard
- [ ] Alerting for anomalies
- [ ] Documentation published
- [ ] Graceful error messages


---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# Logging and Auditing

## Overview

Comprehensive logging for debugging, monitoring, and compliance.

For production requirements, see [[production-hardening]].
For environment differences, see [[environments-local-staging-prod]].

---

## Logging Principles

### 1. Log Everything Important
- All requests and responses — see [[endpoints-contracts]]
- All errors and exceptions
- All [[tools-framework|tool executions]]
- All [[auth-and-jwt|auth events]]

### 2. Don't Log Secrets
- Never log passwords — see [[secrets-management]]
- Never log full [[auth-and-jwt|JWTs]]
- Sanitize PII where possible — see [[threat-model]]

### 3. Structured Logging
- JSON format for machine parsing
- Consistent field names
- Include trace IDs

### 4. Appropriate Levels
| Level | Use For |
|-------|---------|
| DEBUG | Detailed troubleshooting ([[environments-local-staging-prod|local]] only) |
| INFO | Normal operations |
| WARN | Potential issues |
| ERROR | Failures |

---

## Log Structure

### Standard Fields
```
{
  "timestamp": "ISO8601",
  "level": "INFO",
  "service": "gateway",
  "trace_id": "uuid",
  "user_id": "uuid | null",
  "event": "chat_request",
  "data": { ... }
}
```

### Request Log
```
{
  "event": "request_received",
  "method": "POST",
  "path": "/chat",
  "user_id": "a1b2c3...",
  "trace_id": "x1y2z3...",
  "session_id": "s1s2s3...",
  "mode_requested": "instant"
}
```

### Response Log
```
{
  "event": "response_sent",
  "trace_id": "x1y2z3...",
  "status_code": 200,
  "mode_used": "instant",
  "latency_ms": 1234,
  "tokens_used": 150
}
```

### Error Log
```
{
  "event": "error",
  "trace_id": "x1y2z3...",
  "error_code": "INFERENCE_TIMEOUT",
  "error_message": "Model did not respond in time",
  "stack_trace": "..."
}
```

---

## What to Log

### Request/Response
- Timestamp
- User ID (anonymized if needed)
- Session ID
- Request type
- Response status
- Latency

### Inference
- Mode requested vs used
- Model endpoint called
- Tokens in/out
- Inference latency
- Errors

### RAG
- Chunks retrieved
- Similarity scores
- Context size
- Citations generated

### Tools
- Tool name
- Input (sanitized)
- Output (summarized)
- Execution time
- Success/failure

### Auth
- Login attempts
- Token refresh
- Invalid tokens
- Permission denials

---

## What NOT to Log

### Never Log
- Full message content (unless opt-in)
- User passwords
- Full JWT tokens
- Credit card numbers
- Other sensitive PII

### Sanitization Rules
- Truncate long strings
- Hash identifiers where appropriate
- Redact known PII patterns

---

## Audit Trail

### Purpose
- Compliance requirements
- Security investigations
- User support

### Auditable Events
| Event | Data Captured |
|-------|---------------|
| User login | User ID, timestamp, IP |
| Session created | Session ID, user ID, timestamp |
| Message sent | Message ID, session ID, timestamp |
| Tool executed | Tool name, input/output, timestamp |
| File uploaded | Document ID, filename, timestamp |
| File deleted | Document ID, timestamp |

### Retention
- Operational logs: 30 days
- Audit logs: 1 year
- Compliance logs: per regulation (GDPR, etc.)

---

## Log Storage

### Development
- Console output
- Local log files
- Debug level

### Staging
- Structured log files
- Info level
- Retention: 7 days

### Production
- Log aggregation service
- Searchable dashboard
- Alerting integration
- Retention: 30+ days

### Options for Production
- Cloud provider logging (AWS CloudWatch, GCP Logging)
- Self-hosted (ELK stack, Loki)
- Third-party (Datadog, Papertrail)

---

## Monitoring Integration

### Metrics from Logs
- Request rate
- Error rate
- Latency percentiles
- Mode distribution

### Alerting Triggers
| Trigger | Threshold | Alert |
|---------|-----------|-------|
| Error rate | > 5% for 5 min | Critical |
| Latency p95 | > 5 sec | Warning |
| Auth failures | > 20/min | Warning |
| Tool failures | > 10% | Warning |

---

## Trace IDs

### Purpose
- Correlate logs across services
- Debug complex requests
- Track user journey

### Implementation
- Generate UUID at request start
- Include in all logs for that request
- Return in response headers
- Include in error messages

### Example Flow
```
trace_id: abc123
  → request_received
  → auth_validated
  → session_loaded
  → rag_retrieval_started
  → rag_retrieval_completed (chunks: 3)
  → inference_started (mode: instant)
  → inference_completed (latency: 1200ms)
  → response_sent (status: 200)
```

---

## Privacy Considerations

### GDPR Compliance
- Log only what's necessary
- Document retention periods
- Enable user data deletion
- Anonymize where possible

### User Request
- User can request log export
- User can request log deletion
- Document process for handling

---

## Checklist: Logging Setup

- [ ] Structured logging format defined
- [ ] Log levels configured per environment
- [ ] Request/response logging implemented
- [ ] Error logging with stack traces
- [ ] Tool execution logging
- [ ] Auth event logging
- [ ] Trace IDs implemented
- [ ] Sensitive data sanitization
- [ ] Log aggregation configured (prod)
- [ ] Alerting rules defined
- [ ] Retention policy documented


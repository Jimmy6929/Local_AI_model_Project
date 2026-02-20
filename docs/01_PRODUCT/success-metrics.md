---
tags:
  - product
cssclasses:
  - product-doc
---

# Success Metrics

## Overview

Metrics to evaluate whether the system is meeting its goals at each phase.

For goals context, see [[vision-and-goals]].
For phase details, see [[phase-1-chat-mvp]], [[phase-2-two-modes]], [[phase-3-rag]], [[phase-4-tools]], [[phase-5-production-launch]].

---

## Phase 1: MVP Metrics

See [[phase-1-chat-mvp]] for build details.

### Functional Success
- [ ] User can complete [[auth-ux|login flow]]
- [ ] Chat round-trip works end-to-end — see [[data-flow]]
- [ ] Both [[inference-instant|inference]] [[inference-thinking|modes]] respond correctly
- [ ] [[schema|Session persistence]] works across browser refresh

### Performance Targets
| Metric | Target | Measurement |
|--------|--------|-------------|
| [[inference-instant|Instant Mode]] first token | < 2 seconds | Stopwatch from send to first response |
| [[inference-thinking|Thinking Mode]] cold start | < 30 seconds | Stopwatch from toggle to response |
| [[gateway-responsibilities|Gateway]] response time | < 500ms overhead | Excluding inference time |

### Data Integrity
- [ ] [[schema|Messages stored correctly]] in database
- [ ] Session history loads accurately
- [ ] No data loss on browser close

---

## Phase 2: Two-Mode Metrics

See [[phase-2-two-modes]] for build details.

### Routing Accuracy
- [[inference-instant|Instant Mode]] used for simple queries
- [[inference-thinking|Thinking Mode]] used only when toggled or [[routing-logic|heuristic triggers]]
- Mode indicator in [[chat-ux|UI]] matches actual backend routing

### User Experience
- [[chat-ux|Toggle]] is obvious and responsive
- User understands why they're waiting (if cold start) — see [[inference-thinking]]
- Mode selection feels intentional, not random

---

## Phase 3: RAG Metrics

See [[phase-3-rag]] for build details.

### Retrieval Quality
| Metric | Target | Reference |
|--------|--------|------------|
| Relevant chunks in top 5 | > 80% relevance | [[rag-flow]] |
| Context size | Minimal (< 2000 tokens injected) | [[rag-flow]] |
| Citation accuracy | Sources match retrieved chunks | [[chat-ux]] |

### Pipeline Reliability
- [ ] [[file-upload-ux|File upload]] completes without error
- [ ] Text extraction works for pdf, txt, md — see [[storage-and-files]]
- [ ] [[embeddings-and-pgvector|Embeddings]] stored correctly
- [ ] Vector search returns results

---

## Phase 4: Tools Metrics

See [[phase-4-tools]] for build details.

### Tool Execution
- [ ] Tools execute only on backend — see [[openclaw-style-gateway-notes]]
- [ ] Tool calls are [[logging-and-auditing|logged]]
- [ ] Invalid tool calls rejected gracefully — see [[tools-framework]]

### Safety
- [ ] No arbitrary code execution possible — see [[threat-model]]
- [ ] Allowlist enforced — see [[tools-framework]]
- [ ] User sees what tools were called — see [[chat-ux]]

---

## Phase 5: Production Readiness Metrics

See [[phase-5-production-launch]] for build details.

### Security Audit
- [ ] [[rls-policies|RLS policies]] pass manual review — see [[adr-0004-rls-multi-tenant-from-day-1]]
- [ ] No exposed secrets in client code — see [[secrets-management]]
- [ ] [[auth-and-jwt|JWT validation]] cannot be bypassed
- [ ] [[storage-and-files|Storage bucket]] permissions correct

### Operational Readiness
- [ ] [[migrations-and-release-process|Migrations]] apply cleanly to [[deployment-plan|production]]
- [ ] [[logging-and-auditing|Monitoring]] catches errors
- [ ] Backup and restore tested
- [ ] [[rate-limiting-and-abuse|Rate limiting]] prevents abuse

### Cost Control
| Metric | Target | Reference |
|--------|--------|-----------|
| Monthly GPU cost | < $150 private use | [[cost-controls]] |
| [[inference-thinking|Thinking mode]] cost per request | Tracked and bounded | [[cost-controls]] |
| No runaway spending | Alerts at 80% budget | [[cost-controls]] |

---

## How to Measure

- **Latency**: Browser DevTools network tab, or Gateway logging
- **Reliability**: Manual testing during development
- **Security**: Checklist review before each phase promotion
- **Cost**: Cloud provider billing dashboard, daily review


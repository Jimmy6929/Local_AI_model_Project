---
tags:
  - product
cssclasses:
  - product-doc
---

# Vision and Goals

## Vision Statement

Build a private AI assistant that matches the experience of ChatGPT while maintaining full control over infrastructure, data, and costs — architected for eventual SaaS deployment.

For project overview, see [[overview]].
For build guide, see [[step-by-step-guide]].

---

## Primary Goals

### 1. Responsive Daily Assistant
- [[inference-instant|Instant Mode]] handles most tasks with low latency
- Feels like a premium [[chat-ux|chat experience]]
- No waiting for cold starts during normal use

### 2. Deep Thinking When Needed
- Toggle to activate stronger reasoning — see [[inference-thinking]]
- Accept cold start tradeoff for capability — see [[adr-0001-two-tier-inference]]
- Used for complex coding, analysis, planning

### 3. Full Data Ownership
- All chats stored in owned [[Supabase]] instance — see [[schema]]
- Documents and files under personal control — see [[storage-and-files]]
- No data sent to third-party LLM providers

### 4. Cost Control
- Pay for GPU compute, not per-token API fees — see [[cost-controls]]
- Scale-to-zero for expensive [[inference-thinking|thinking model]]
- [[cost-controls|Budget guardrails]] and usage monitoring

### 5. SaaS-Ready Architecture
- [[adr-0004-rls-multi-tenant-from-day-1|Multi-tenant schema from day one]]
- [[rls-policies|RLS]] enforced at database level
- Easy transition from private to public product — see [[phase-5-saas-launch]]

---

## Non-Goals (For Now)

- Public deployment (staying private during dev)
- Mobile native apps (web-first)
- Voice interface (text chat only)
- Real-time collaboration (single-user focus)
- Enterprise features (audit trails, SSO — future)

---

## Success Criteria

See [[success-metrics]] for detailed measurement.

| Metric | Target | Reference |
|--------|--------|------------|
| [[inference-instant|Instant Mode]] latency | Under 2 seconds first token | [[success-metrics]] |
| [[inference-thinking|Thinking Mode]] availability | Cold start under 30 seconds | [[cost-controls]] |
| Data isolation | [[rls-policies|RLS]] passes security audit | [[adr-0004-rls-multi-tenant-from-day-1]] |
| Local-to-prod transition | Under 1 day migration | [[deployment-plan]] |
| Monthly infrastructure cost | Under $150 for private use | [[cost-controls]] |

---

## Long-Term Vision

See [[phase-5-saas-launch]] for SaaS roadmap.

Phase into a SaaS product where:
- Multiple users subscribe to the service — see [[adr-0004-rls-multi-tenant-from-day-1]]
- Each user has isolated data and sessions — see [[rls-policies]]
- Pricing tiers based on [[inference-thinking|thinking mode]] usage — see [[rate-limiting-and-abuse]]
- Self-serve onboarding via [[ui-pages-and-components|web app]]


---
tags:
  - architecture
cssclasses:
  - architecture-doc
---

# Environments: Local → Staging → Production

## Overview

Three environments with increasing stability and security requirements.

Related documents:
- [[adr-0002-supabase-local-then-cloud]] — Local-first strategy
- [[deployment-plan]] — Production deployment steps
- [[production-hardening]] — Security requirements
- [[secrets-management]] — Managing environment variables

---

## Environment Comparison

| Aspect | Local | Staging | Production |
|--------|-------|---------|------------|
| Purpose | Development | Integration testing | Real users |
| Supabase | [[supabase-local-dev-plan|Local CLI stack]] | Hosted project or always-on home | Supabase Cloud |
| Inference | Local or cloud dev | Cloud GPU | [[inference-instant]], [[inference-thinking|Cloud GPU]] |
| Data | Test data, can reset | Test data, persistent | Real user data |
| Auth | [[auth-and-jwt|Supabase local]] | Supabase hosted | Supabase Cloud |
| Networking | [[networking-and-private-access|Tailscale mesh]] | Tailscale or private | VPC / private backend |
| RLS | [[rls-policies|Enabled]] | Enabled + tested | Enabled + [[adr-0004-rls-multi-tenant-from-day-1|audited]] |
| Monitoring | None | Basic | [[logging-and-auditing|Full alerting]] |
| Backups | None | Optional | Required |

---

## Local Environment

### Purpose
- Active development
- Rapid iteration
- Testing new features

### Setup
- [[supabase-local-dev-plan|Supabase CLI running locally]]
- [[gateway-responsibilities|Gateway]] running on localhost
- Inference endpoint (cloud dev or mock) — see [[inference-instant]]
- [[ui-pages-and-components|Web app]] on localhost

### Characteristics
- Can reset database frequently — see [[migrations-and-release-process]]
- Debug [[logging-and-auditing|logging]] enabled
- No [[production-hardening|security hardening]] needed
- MacBook can sleep (dev box only)

### Configuration
- [[secrets-management|Environment variables]] point to local services
- No secrets in version control
- `.env.local` file for overrides

---

## Staging Environment

### Purpose
- Integration testing
- Pre-production validation
- Performance testing

### Setup Options

**Option A: Hosted Supabase Project**
- Free tier Supabase project — see [[adr-0002-supabase-local-then-cloud]]
- Full [[auth-and-jwt|Auth]] + [[schema|DB]] + [[storage-and-files|Storage]]
- Mirrors production capabilities

**Option B: Always-On Home Stack**
- Spare MacBook as staging server
- [[supabase-local-dev-plan|Local Supabase]] running 24/7
- Requires: disable sleep, UPS, monitoring

### Characteristics
- Persistent data (not reset frequently)
- [[rls-policies|RLS policies]] actively tested
- Inference endpoints same as production — see [[inference-instant]], [[inference-thinking]]
- Simulates real user flows

### Configuration
- Separate [[secrets-management|environment variables]]
- Production-like settings
- Test users only (no real data)

---

## Production Environment

### Purpose
- Real users
- Business-critical
- Must be secure and reliable

### Setup
- [[adr-0002-supabase-local-then-cloud|Supabase Cloud]] (paid plan)
- Production [[inference-instant|GPU infrastructure]] — see [[deployment-plan]]
- CDN for [[ui-pages-and-components|web app]]
- Full [[logging-and-auditing|monitoring and alerting]]

### Characteristics
- Zero-tolerance for data loss
- [[rls-policies|RLS audited]] and verified — see [[adr-0004-rls-multi-tenant-from-day-1]]
- [[rate-limiting-and-abuse|Rate limiting]] enabled
- Backups scheduled
- Incident response plan — see [[production-hardening]]

### Configuration
- [[secrets-management|Secrets in secure vault]]
- No debug [[logging-and-auditing|logging]]
- Performance monitoring
- [[cost-controls|Cost alerting]]

---

## Migration Process

See [[migrations-and-release-process]] for detailed workflow.

### Local → Staging
1. Run [[migrations-and-release-process|migrations]] locally first
2. Test all features
3. Apply migrations to staging
4. Smoke test in staging
5. Fix any issues before proceeding

### Staging → Production
1. All staging tests pass
2. [[production-hardening|Security checklist]] complete
3. Apply migrations to production
4. Gradual rollout (if applicable)
5. Monitor for errors — see [[logging-and-auditing]]

---

## Environment Variable Strategy

See [[secrets-management]] for security best practices.

| Variable | Local | Staging | Production |
|----------|-------|---------|------------|
| SUPABASE_URL | localhost:54321 | staging project URL | production URL |
| SUPABASE_KEY | local anon key | staging anon key | production anon key |
| SERVICE_KEY | local service key | staging service key | [[secrets-management|vault-managed]] |
| INSTANT_URL | dev endpoint | [[inference-instant|prod endpoint]] | prod endpoint |
| THINK_URL | dev endpoint | [[inference-thinking|prod endpoint]] | prod endpoint |
| LOG_LEVEL | debug | info | warn |
| RATE_LIMIT | disabled | [[rate-limiting-and-abuse|enabled]] | strict |

---

## Checklist: Before Production

See [[production-hardening]] for complete security requirements.

- [ ] All [[rls-policies|RLS policies]] reviewed and tested
- [ ] [[storage-and-files|Storage bucket]] permissions verified
- [ ] [[secrets-management|Service role key]] not exposed anywhere
- [ ] [[auth-and-jwt|JWT validation]] cannot be bypassed
- [ ] [[rate-limiting-and-abuse|Rate limiting]] configured
- [ ] [[logging-and-auditing|Monitoring and alerting]] set up
- [ ] Backup schedule configured
- [ ] Incident response plan documented
- [ ] Cost alerts configured
- [ ] Security audit completed


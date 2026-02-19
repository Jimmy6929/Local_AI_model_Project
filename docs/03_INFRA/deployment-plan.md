---
tags:
  - infrastructure
cssclasses:
  - infra-doc
---

# Deployment Plan

## Overview

How to deploy each component from development to production.

For environment details, see [[environments-local-staging-prod]].
For security requirements, see [[production-hardening]].

---

## Component Deployment Summary

| Component | Dev Deployment | Staging | Production |
|-----------|----------------|---------|------------|
| [[ui-pages-and-components|Web App]] | localhost:3000 | Vercel preview | Vercel production |
| [[gateway-responsibilities|Gateway]] | localhost:8000 | Cloud VM or home server | Cloud VM + LB |
| [[inference-instant|Instant GPU]] | Cloud dev instance | Cloud GPU pod | Cloud GPU pod |
| [[inference-thinking|Thinking GPU]] | Same or mock | Serverless endpoint | Serverless endpoint |
| [[Supabase]] | [[supabase-local-dev-plan|Local CLI stack]] | Hosted project | [[adr-0002-supabase-local-then-cloud|Supabase Cloud]] |

---

## Web App (Next.js)

See [[ui-pages-and-components]] for component details.

### Development
- Run locally with `npm run dev`
- Points to local [[gateway-responsibilities|Gateway]] and [[supabase-local-dev-plan|Supabase]]

### Staging
- Vercel preview deployments
- [[secrets-management|Environment variables]] for staging Supabase/Gateway
- Automatic deploy on PR

### Production
- Vercel production deployment
- Custom domain
- Production [[secrets-management|environment variables]]
- CDN caching enabled

### Deployment Checklist
- [ ] [[secrets-management|Environment variables]] set correctly
- [ ] [[auth-ux|Auth callback URLs]] configured in [[Supabase]]
- [ ] API routes point to correct [[gateway-responsibilities|Gateway]]
- [ ] Error tracking enabled (Sentry, etc.) — see [[logging-and-auditing]]

---

## Gateway (FastAPI)

See [[gateway-responsibilities]] for detailed responsibilities.

### Development
- Run locally with uvicorn
- Hot reload enabled
- Debug [[logging-and-auditing|logging]]

### Staging
- Container deployment (Docker)
- Cloud VM or home server
- [[networking-and-private-access|Tailscale]] for private access
- Systemd or similar for process management

### Production
- Container deployment
- Cloud VM with reserved resources
- Load balancer for HTTPS termination — see [[networking-and-private-access]]
- Auto-scaling (optional for private use)

### Deployment Checklist
- [ ] Docker image built and tested
- [ ] [[secrets-management|Environment variables]] for production
- [ ] Health check endpoint working — see [[endpoints-contracts]]
- [ ] [[logging-and-auditing|Logging]] configured (structured JSON)
- [ ] [[secrets-management|Secrets]] not baked into image

---

## Instant GPU Pod

See [[inference-instant]] for infrastructure requirements.

### Development
- Cloud GPU instance (RunPod, Lambda Labs, etc.)
- Start manually when developing
- Stop when done — see [[cost-controls]]

### Staging
- Same infrastructure as production
- Separate endpoint URL
- Test model and configuration

### Production
- Dedicated GPU pod
- Auto-restart on failure
- [[logging-and-auditing|Monitoring]] and alerting
- [[networking-and-private-access|Private networking]] only

### Deployment Checklist
- [ ] GPU type and VRAM verified
- [ ] Model downloaded and loaded
- [ ] [[adr-0003-openai-compatible-inference-interface|API server]] running and healthy
- [ ] Firewall allows only [[gateway-responsibilities|Gateway]] — see [[networking-and-private-access]]
- [ ] Process manager configured

---

## Thinking GPU (Serverless)

### Development
- Serverless endpoint on dev account
- May share with staging
- Low usage during dev

### Staging/Production
- Serverless GPU platform (RunPod, Modal, etc.)
- Configure scale-to-zero
- Set concurrency and timeout limits
- Spending caps enabled

### Deployment Checklist
- [ ] Endpoint URL configured
- [ ] Model selected and tested
- [ ] Token limits set
- [ ] Spending caps enabled
- [ ] Cold start time acceptable

---

## Supabase

### Development
- Local stack via Supabase CLI
- Run `supabase start`
- Migrations tracked in version control

### Staging
- Option A: Free-tier Supabase project
- Option B: Always-on local stack on staging server
- Migrations applied from main branch

### Production
- Supabase Cloud (Pro plan recommended)
- Migrations applied via CI/CD
- Backups enabled
- Point-in-time recovery configured

### Migration Workflow
1. Create migration locally: `supabase migration new <name>`
2. Apply locally: `supabase db reset`
3. Test thoroughly
4. Push to staging
5. Apply to staging Supabase
6. Test in staging
7. Apply to production

### Deployment Checklist
- [ ] All RLS policies verified
- [ ] Storage bucket permissions set
- [ ] Service role key secured
- [ ] Backups configured
- [ ] Monitoring enabled

---

## CI/CD Pipeline (Future)

### Triggers
- Push to main → deploy to staging
- Tag/release → deploy to production

### Stages
1. **Test** — Run unit and integration tests
2. **Build** — Build Docker images, Next.js build
3. **Deploy Staging** — Deploy all components
4. **Smoke Test** — Verify staging works
5. **Deploy Production** — Manual approval, then deploy

### Secrets Management
- Store in CI/CD platform (GitHub Secrets, etc.)
- Never commit to repository
- Rotate periodically

---

## Rollback Plan

### Web App
- Vercel instant rollback to previous deployment

### Gateway
- Keep previous Docker image tagged
- Rollback: redeploy previous tag

### Supabase
- Database: restore from backup
- Migrations: write reverse migration

### Inference
- Keep previous model version available
- Rollback: restart pod with previous config

---

## Deployment Environments Summary

```
┌──────────────────────────────────────────────────────────────┐
│                         LOCAL                                │
│  localhost:3000 → localhost:8000 → local Supabase           │
│                            ↓                                 │
│                     Cloud GPU (dev)                          │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                        STAGING                               │
│  Vercel Preview → Staging VM → Staging Supabase             │
│                            ↓                                 │
│                  Cloud GPU (staging)                         │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                       PRODUCTION                             │
│  Vercel Prod → Production VM/LB → Supabase Cloud            │
│                            ↓                                 │
│                  Cloud GPU (production)                      │
└──────────────────────────────────────────────────────────────┘
```


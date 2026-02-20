---
tags:
  - roadmap
cssclasses:
  - roadmap-doc
---

# Phase 5: Production Launch

## Goal

Transition from private use to production-ready deployment with multiple users.

For security requirements, see [[production-hardening]] and [[threat-model]].
For multi-tenant architecture, see [[adr-0004-rls-multi-tenant-from-day-1]].
Previous: [[phase-4-tools]]

---

## Success Criteria

- [ ] Production infrastructure deployed
- [ ] Security hardening complete
- [ ] Multi-tenant isolation verified
- [ ] Monitoring and alerting active
- [ ] Ready for external users

---

## Scope

### In Scope
- [[deployment-plan|Production deployment]]
- [[production-hardening|Security audit and fixes]]
- Performance optimization — see [[cost-controls]]
- [[logging-and-auditing|Operational tooling]]
- [[auth-ux|User onboarding flow]]

### Out of Scope
- Billing integration (future)
- Advanced admin features
- Mobile apps
- Enterprise features (SSO, audit)

---

## Milestones

### M5.1: Production Infrastructure
**Goal:** Deploy to [[environments-local-staging-prod|production environment]]

- [[adr-0002-supabase-local-then-cloud|Supabase Cloud]] project
- Production [[inference-instant|GPU pods]]
- [[ui-pages-and-components|Web app]] deployment (Vercel)
- [[gateway-responsibilities|Gateway]] deployment
- DNS and SSL setup — see [[networking-and-private-access]]

**Acceptance:** All services running in production

---

### M5.2: Migration
**Goal:** Move from local to production

- Apply all [[migrations-and-release-process|migrations]] to production DB
- Configure [[storage-and-files|storage buckets]]
- Set up [[auth-and-jwt|Auth settings]]
- Verify [[rls-policies|RLS policies]] work — see [[adr-0004-rls-multi-tenant-from-day-1]]

**Acceptance:** Clean production database ready

---

### M5.3: Security Hardening
**Goal:** Production security measures

- Complete production hardening checklist
- Security headers configured
- Rate limiting enabled
- Secrets in secret manager
- Firewall rules verified

**Acceptance:** Security checklist 100% complete

---

### M5.4: Monitoring Setup
**Goal:** Visibility into production

- Health checks configured
- Error alerting
- Performance metrics
- Usage dashboards
- Cost monitoring

**Acceptance:** Can detect issues within 5 minutes

---

### M5.5: Backup & Recovery
**Goal:** Data protection

- Automated backups enabled
- Point-in-time recovery configured
- Recovery procedure documented
- Test restore completed

**Acceptance:** Can restore from backup

---

### M5.6: User Onboarding
**Goal:** New users can sign up and use

- Public signup enabled
- Welcome email (optional)
- Getting started guide
- Support contact

**Acceptance:** New user can complete signup to first chat

---

### M5.7: Documentation
**Goal:** Support and operational docs

- User documentation
- API documentation (if public)
- Operational runbooks
- Incident response plan

**Acceptance:** Documentation reviewed and published

---

## Production Environment

### Services
| Service | Platform |
|---------|----------|
| Web App | Vercel |
| Gateway | Cloud VM / Container |
| Instant GPU | Cloud GPU provider |
| Thinking GPU | Serverless GPU |
| Database | Supabase Cloud |
| Storage | Supabase Storage |

### Configuration
- Separate secrets from staging
- Production-only API keys
- Higher rate limits where appropriate
- Stricter security settings

---

## Security Checklist (Recap)

- [ ] All RLS policies verified
- [ ] JWT validation secure
- [ ] HTTPS enforced
- [ ] Security headers set
- [ ] Secrets in vault
- [ ] No debug info exposed
- [ ] Rate limiting active
- [ ] Input validation complete
- [ ] Output sanitization complete
- [ ] Logging sanitized

---

## Monitoring Stack

### Minimum Viable Monitoring
- Health check endpoints
- Error rate alerts
- Response time alerts
- Cost alerts

### Recommended Additions
- Structured log aggregation
- Distributed tracing
- Custom metrics dashboard
- On-call rotation

---

## Launch Checklist

### Week Before
- [ ] Final security review
- [ ] Load testing (optional)
- [ ] Backup verified
- [ ] Monitoring tested
- [ ] Documentation complete
- [ ] Support ready

### Launch Day
- [ ] Deploy final version
- [ ] Smoke test all features
- [ ] Monitor closely (4 hours)
- [ ] Team on standby

### Post-Launch
- [ ] Review logs daily
- [ ] Address issues immediately
- [ ] Gather user feedback
- [ ] Plan next iteration

---

## Future Considerations

### After Launch
- User feedback collection
- Performance optimization
- Feature requests prioritization
- Scaling preparation

### Future Phases
- Billing and subscriptions
- Team/organization support
- Enterprise features
- Mobile apps
- API for integrations

---

## Dependencies

- Phases 1-4 complete
- Supabase Cloud account
- Production infrastructure access
- Domain and SSL
- Monitoring tools

---

## Risks

| Risk | Mitigation |
|------|------------|
| Performance under load | Start small, scale as needed |
| Security vulnerabilities | Audit, monitoring, quick response |
| High costs | Budget alerts, optimize usage |
| User issues | Clear docs, support process |

---

## Timeline

Estimated: 2-3 weeks

| Week | Focus |
|------|-------|
| 1 | M5.1 + M5.2 (Infrastructure + Migration) |
| 2 | M5.3 + M5.4 + M5.5 (Security + Monitoring + Backup) |
| 3 | M5.6 + M5.7 (Onboarding + Docs) + Launch |

---

## Exit Criteria

Phase 5 complete when:
- Production running stably
- Security hardened
- Monitoring active
- Ready for users
- 🚀 LAUNCHED


---
tags:
  - security
cssclasses:
  - security-doc
---

# Production Hardening

## Overview

Security measures required before deploying to production with real users.

For environment setup, see [[environments-local-staging-prod]].
For deployment steps, see [[deployment-plan]] and [[phase-5-production-launch]].

---

## Pre-Production Checklist

### Authentication & Authorization
See [[auth-and-jwt]] and [[rls-policies]] for implementation details.

- [ ] All [[endpoints-contracts|endpoints]] require authentication
- [ ] [[auth-and-jwt|JWT validation]] correct and complete
- [ ] Token expiration enforced
- [ ] Refresh token flow secure
- [ ] [[rls-policies|RLS policies]] on all user tables — see [[adr-0004-rls-multi-tenant-from-day-1]]
- [ ] RLS policies tested with multiple users
- [ ] [[secrets-management|Service role key]] not exposed to clients

### Network Security
See [[networking-and-private-access]] for architecture details.

- [ ] HTTPS enforced everywhere
- [ ] TLS 1.3 preferred
- [ ] HSTS headers enabled
- [ ] [[inference-instant|Inference endpoints]] private — see [[networking-and-private-access]]
- [ ] Database not publicly accessible
- [ ] Firewall rules reviewed

### Data Protection
- [ ] Encryption at rest enabled
- [ ] Backup encryption enabled
- [ ] PII minimized in logs
- [ ] Sensitive data masked
- [ ] Retention policies defined

### Application Security
See [[threat-model]] for attack categories.

- [ ] Input validation on all [[endpoints-contracts|endpoints]]
- [ ] Output sanitization (XSS prevention)
- [ ] SQL injection prevention (parameterized) — see [[threat-model|T2.2]]
- [ ] CSRF protection enabled
- [ ] [[rate-limiting-and-abuse|Rate limiting]] active
- [ ] Error messages don't leak info — see [[logging-and-auditing]]

---

## HTTPS Configuration

### Requirements
- Valid SSL certificate
- Automatic HTTP → HTTPS redirect
- TLS 1.2 minimum, 1.3 preferred
- Strong cipher suites

### Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Certificate Management
- Auto-renewal (Let's Encrypt, Cloudflare, etc.)
- Monitor expiration
- Alert before expiry

---

## Security Headers

### Required Headers
```
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=()
```

### CSP Configuration
- Restrict script sources
- Block inline scripts where possible
- Report violations (report-uri)

---

## Rate Limiting Production Config

### Limits
| Endpoint | Limit | Action |
|----------|-------|--------|
| POST /chat | 60/min | 429 + retry-after |
| POST /chat (think) | 10/min | 429 + retry-after |
| POST /files | 10/min | 429 + retry-after |
| GET endpoints | 120/min | 429 + retry-after |
| Auth endpoints | 10/min per IP | 429 + temp block |

### DDoS Protection
- Use CDN/WAF (Cloudflare, AWS WAF)
- Challenge suspicious traffic
- Geographic restrictions if appropriate

---

## Database Hardening

### Supabase Specific
- [ ] RLS enabled on all tables
- [ ] Policies use auth.uid()
- [ ] Public schema restricted
- [ ] Storage bucket policies set
- [ ] Connection limits configured

### General
- [ ] Strong password for database
- [ ] SSL required for connections
- [ ] No public IP (or restricted)
- [ ] Backup schedule verified
- [ ] Point-in-time recovery enabled

---

## Secrets in Production

### Storage
- Use secret manager (not env vars in plain text)
- Encrypt at rest
- Access logging enabled

### Rotation
- JWT secret: rotate quarterly
- API keys: rotate on schedule
- Database password: rotate on personnel change

### Access
- Principle of least privilege
- Audit log for secret access
- Alert on unusual access

---

## Logging & Monitoring

### Production Logging
- No debug logs
- No sensitive data
- Structured format
- Centralized aggregation

### Monitoring Requirements
- [ ] Health checks configured
- [ ] Error rate alerting
- [ ] Latency alerting
- [ ] Resource usage monitoring
- [ ] Security event alerting

### Security Monitoring
- Failed login rate
- Authorization failures
- Unusual query patterns
- Rate limit hits
- Tool execution anomalies

---

## Backup & Recovery

### Backup Requirements
- Daily database backups
- Point-in-time recovery (7 days minimum)
- Backup encryption
- Off-site backup storage

### Recovery Testing
- Quarterly restore test
- Document recovery procedure
- Test recovery time

### Data Retention
- Define retention periods
- Automated cleanup
- Comply with regulations

---

## Dependency Security

### Regular Updates
- Automated dependency scanning
- Weekly review of vulnerabilities
- Patch critical issues immediately

### Tools
- npm audit / pip-audit
- GitHub Dependabot
- Snyk or similar

### Container Security
- Minimal base images
- Regular image updates
- Scan images for vulnerabilities

---

## Incident Preparedness

### Documentation
- [ ] Incident response plan written
- [ ] Escalation contacts defined
- [ ] Runbooks for common issues

### Capabilities
- [ ] Can revoke all sessions immediately
- [ ] Can block specific users
- [ ] Can roll back deployments
- [ ] Can restore from backup

### Testing
- Annual incident drill
- Document lessons learned
- Update procedures

---

## Compliance Considerations

### GDPR (If EU Users)
- Data processing documentation
- Right to deletion capability
- Data export capability
- Privacy policy

### General
- Terms of service
- Acceptable use policy
- Cookie consent (if applicable)

---

## Launch Day Checklist

### Before Launch
- [ ] All hardening items complete
- [ ] Penetration test (optional but recommended)
- [ ] Security review by second person
- [ ] Monitoring verified working
- [ ] Alerting tested

### Launch
- [ ] Deploy with minimal fanfare initially
- [ ] Monitor closely first 24 hours
- [ ] Team on standby for issues

### Post-Launch
- [ ] Review logs for anomalies
- [ ] Address any issues immediately
- [ ] Document any changes needed

---

## Ongoing Security

### Regular Activities
| Activity | Frequency |
|----------|-----------|
| Dependency updates | Weekly |
| Security log review | Weekly |
| Backup restore test | Quarterly |
| Secret rotation | Quarterly |
| Penetration test | Annually |
| Security review | Annually |

### Stay Current
- Follow security advisories
- Monitor for new vulnerabilities
- Update defenses as threats evolve


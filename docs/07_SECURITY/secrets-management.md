---
tags:
  - security
cssclasses:
  - security-doc
---

# Secrets Management

## Overview

Secure handling of API keys, credentials, and sensitive configuration.

For environment setup, see [[environments-local-staging-prod]].
For production requirements, see [[production-hardening]].

---

## Types of Secrets

### Critical Secrets
| Secret | Risk if Exposed | Reference |
|--------|-----------------|------------|
| [[Supabase]] Service Role Key | Full database access, bypass [[rls-policies|RLS]] | [[threat-model]] |
| [[auth-and-jwt|JWT Secret]] | Forge any user session | [[auth-and-jwt]] |
| Database password | Direct database access | [[schema]] |

### Important Secrets
| Secret | Risk if Exposed |
|--------|-----------------|
| Inference API keys | Resource abuse |
| Embedding API key | Cost, rate limits |
| Third-party API keys | Service abuse |

### Sensitive Configuration
| Config | Risk if Exposed |
|--------|-----------------|
| Endpoint URLs | Attack surface mapping |
| Internal IPs | Network reconnaissance |

---

## Core Principles

### 1. Never Commit Secrets
- No secrets in version control
- No secrets in code
- No secrets in comments

### 2. Least Privilege
- Each component gets only needed secrets
- Rotate regularly
- Revoke immediately if compromised

### 3. Audit Trail
- Log secret access (not values)
- Monitor for anomalies
- Alert on suspicious access

---

## Environment Variables

### Structure
```
# .env.example (committed - no real values)
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_KEY=
SUPABASE_JWT_SECRET=
INSTANT_INFERENCE_URL=
THINKING_INFERENCE_URL=
```

```
# .env.local (not committed - real values)
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_KEY=eyJ...
...
```

### .gitignore
```
.env
.env.local
.env.*.local
*.pem
*.key
```

---

## By Environment

### Development
- Local .env files
- Supabase CLI generates local secrets
- Can be reset easily

### Staging
- Environment variables on host
- Access limited to staging systems
- Different keys than production

### Production
- Secret manager (recommended)
- Encrypted at rest
- Minimal access
- Rotation policy

---

## Secret Manager Options

### Cloud Provider Native
- AWS Secrets Manager
- GCP Secret Manager
- Azure Key Vault

### Self-Hosted
- HashiCorp Vault
- Doppler

### Platform-Specific
- Vercel Environment Variables
- Railway Variables
- Fly.io Secrets

### Recommendation
Start with platform environment variables. Move to dedicated manager as complexity grows.

---

## Access Patterns

### Service Access
```
Gateway → reads → SUPABASE_SERVICE_KEY
Gateway → reads → JWT_SECRET
Gateway → reads → INFERENCE_URLS
```

### Never Accessed By
- Web App frontend (anon key only)
- Inference servers (no database access)
- Logs (never log secrets)

---

## Secret Rotation

### When to Rotate
- Scheduled (quarterly for critical)
- After employee departure
- After suspected compromise
- After dependency update

### Rotation Process
1. Generate new secret
2. Deploy with both old and new valid
3. Verify new secret works
4. Remove old secret
5. Update documentation

### Automated Rotation (Future)
- Secret manager handles rotation
- Services refresh secrets periodically
- Zero-downtime updates

---

## Detecting Exposure

### Git History
- Use tools like git-secrets, trufflehog
- Scan before every push
- Scan repository history

### Public Exposure
- Monitor for leaked keys (GitHub alerts)
- Set up secret scanning
- Google alerts for key patterns

### Immediate Response
1. Revoke exposed secret immediately
2. Generate new secret
3. Deploy with new secret
4. Audit for unauthorized access
5. Document incident

---

## Developer Practices

### Do
- Use environment variables
- Use .env.example with placeholders
- Share secrets via secure channel (1Password, etc.)
- Rotate after sharing

### Don't
- Commit real secrets
- Share secrets via chat/email
- Use same secrets across environments
- Log secret values

---

## CI/CD Secrets

### GitHub Actions
- Use repository secrets
- Use environment protection rules
- Never echo secrets in logs

### Deployment
- Inject secrets at deploy time
- Don't bake into images
- Verify secrets are set before deploy

---

## Emergency Procedures

### If Secret is Exposed

**Immediate (within minutes)**
1. Revoke the secret
2. Generate replacement
3. Deploy new secret

**Short-term (within hour)**
4. Audit access logs
5. Check for unauthorized actions
6. Notify affected parties

**Follow-up (within day)**
7. Document incident
8. Review how it happened
9. Implement prevention measures

---

## Checklist: Secrets Security

- [ ] All secrets in environment variables
- [ ] .gitignore includes all secret files
- [ ] .env.example has no real values
- [ ] Service role key restricted to Gateway
- [ ] JWT secret not exposed anywhere
- [ ] Rotation schedule documented
- [ ] Secret scanning enabled
- [ ] Emergency procedure documented
- [ ] Secrets differ per environment
- [ ] Access audit trail exists


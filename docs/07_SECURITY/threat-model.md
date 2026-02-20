---
tags:
  - security
cssclasses:
  - security-doc
---

# Threat Model

## Overview

Security threats and mitigations for the Local AI Assistant system.

For specific defenses, see:
- [[prompt-injection-notes]] — AI-specific attacks
- [[secrets-management]] — Credential security
- [[production-hardening]] — Deployment security
- [[rls-policies]] — Data isolation

---

## Threat Actors

### External Attackers
- **Motivation:** Data theft, service disruption, resource abuse
- **Capabilities:** Network attacks, credential stuffing, injection attempts
- **Targets:** Public endpoints, user accounts, stored data

### Malicious Users
- **Motivation:** Free resource usage, data exfiltration, abuse
- **Capabilities:** Account creation, API access, prompt manipulation
- **Targets:** Inference resources, other users' data, system prompts

### Insider Threats
- **Motivation:** Data access, sabotage
- **Capabilities:** Privileged access, knowledge of systems
- **Targets:** Production data, secrets, infrastructure

---

## Asset Classification

### Critical Assets
| Asset | Impact if Compromised | Reference |
|-------|----------------------|------------|
| User credentials | Account takeover | [[auth-and-jwt]] |
| User data (chats, files) | Privacy breach | [[rls-policies]], [[schema]] |
| [[secrets-management|Service role key]] | Full database access | [[secrets-management]] |
| [[inference-instant|Inference endpoints]] | Resource abuse | [[networking-and-private-access]] |
| [[auth-and-jwt|JWT secret]] | Session forgery | [[secrets-management]] |

### Important Assets
| Asset | Impact if Compromised |
|-------|----------------------|
| System prompts | Behavior manipulation |
| Usage logs | Privacy leak |
| Model weights | IP theft |

---

## Threat Categories

### T1: Authentication Attacks

**T1.1: Credential Stuffing**
- Attacker uses leaked credentials
- Mitigation: Rate limiting, breach detection, MFA (future)

**T1.2: Brute Force**
- Attacker guesses passwords
- Mitigation: Rate limiting, account lockout, strong password requirements

**T1.3: Session Hijacking**
- Attacker steals JWT
- Mitigation: httpOnly cookies, short expiry, TLS only

---

### T2: Injection Attacks

**T2.1: Prompt Injection**
- User manipulates model behavior via input
- Mitigation: System prompt isolation, input filtering, monitoring

**T2.2: SQL Injection**
- Malicious input in database queries
- Mitigation: Parameterized queries, ORM usage, input validation

**T2.3: XSS (Cross-Site Scripting)**
- Malicious scripts in chat content
- Mitigation: Output sanitization, CSP headers, React auto-escaping

---

### T3: Data Exposure

**T3.1: Cross-User Data Access**
- User A sees User B's data
- Mitigation: RLS policies, user_id validation, testing

**T3.2: Log Exposure**
- Sensitive data in logs
- Mitigation: Log sanitization, access controls, retention limits

**T3.3: Backup Exposure**
- Unencrypted backups accessed
- Mitigation: Encrypted backups, access controls

---

### T4: Resource Abuse

**T4.1: Inference Abuse**
- Excessive requests drain GPU budget
- Mitigation: Rate limiting, spending caps, monitoring

**T4.2: Storage Abuse**
- Large file uploads fill storage
- Mitigation: File size limits, quotas, monitoring

**T4.3: DoS Attack**
- Overwhelming requests crash service
- Mitigation: Rate limiting, WAF, auto-scaling

---

### T5: Infrastructure Attacks

**T5.1: Exposed Inference Endpoints**
- Model servers accessible publicly
- Mitigation: Private networking, firewall rules, no public IPs

**T5.2: Secrets Exposure**
- API keys in code or logs
- Mitigation: Environment variables, secret managers, code scanning

**T5.3: Supply Chain Attack**
- Compromised dependency
- Mitigation: Dependency scanning, lock files, minimal dependencies

---

## Risk Matrix

| Threat | Likelihood | Impact | Risk Level | Priority |
|--------|------------|--------|------------|----------|
| T1.1 Credential stuffing | Medium | High | High | P1 |
| T1.2 Brute force | Low | High | Medium | P2 |
| T2.1 Prompt injection | High | Medium | High | P1 |
| T3.1 Cross-user access | Low | Critical | High | P1 |
| T4.1 Inference abuse | Medium | High | High | P1 |
| T5.1 Exposed endpoints | Low | Critical | Medium | P2 |
| T5.2 Secrets exposure | Low | Critical | Medium | P2 |

---

## Security Controls by Layer

### Network Layer
- [ ] TLS for all traffic
- [ ] Private subnets for inference
- [ ] Firewall rules enforced
- [ ] VPN for dev access

### Application Layer
- [ ] JWT validation on all endpoints
- [ ] Input validation
- [ ] Output sanitization
- [ ] Rate limiting

### Data Layer
- [ ] RLS on all tables
- [ ] Encryption at rest
- [ ] Encrypted backups
- [ ] Minimal data retention

### Operational Layer
- [ ] Secrets in vault
- [ ] Audit logging
- [ ] Monitoring and alerting
- [ ] Incident response plan

---

## Monitoring for Threats

### Indicators of Compromise
| Signal | Possible Threat |
|--------|-----------------|
| High failed login rate | Credential attack |
| Unusual query patterns | Prompt injection |
| Cross-user query attempts | Authorization bypass |
| Sudden usage spike | Resource abuse |
| New IP for sensitive actions | Account compromise |

### Alerting
- Real-time alerts for critical signals
- Daily review of security logs
- Weekly security metrics review

---

## Incident Response

### Severity Levels
| Level | Definition | Response Time |
|-------|------------|---------------|
| Critical | Active breach, data exposure | Immediate |
| High | Potential breach, service impact | 1 hour |
| Medium | Suspicious activity | 24 hours |
| Low | Minor policy violation | 1 week |

### Response Steps
1. Detect and alert
2. Assess scope
3. Contain threat
4. Eradicate cause
5. Recover service
6. Document lessons

---

## Checklist: Security Review

- [ ] Authentication hardened
- [ ] RLS policies tested
- [ ] Injection defenses in place
- [ ] Logging sanitized
- [ ] Secrets secured
- [ ] Network isolated
- [ ] Monitoring active
- [ ] Incident plan documented


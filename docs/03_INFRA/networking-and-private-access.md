---
tags:
  - infrastructure
cssclasses:
  - infra-doc
---

# Networking and Private Access

## Overview

Keep inference endpoints private while allowing [[gateway-responsibilities|Gateway]] to reach them securely.

For threat context, see [[threat-model]].

---

## Threat Model

See [[threat-model]] for comprehensive security analysis.

### What We're Protecting Against
- Public exposure of inference endpoints → abuse, [[cost-controls|cost attacks]]
- Unauthorized access to model servers → data leakage, [[prompt-injection-notes|prompt injection]]
- Man-in-the-middle attacks → credential theft — see [[auth-and-jwt]]
- Internal lateral movement → if one component is compromised

### Assets to Protect
- [[inference-instant|Inference endpoints]] ([[inference-instant|Instant]] and [[inference-thinking|Thinking]])
- [[Supabase]] database and [[storage-and-files|storage]]
- [[gateway-responsibilities|Gateway service]]
- User credentials and [[auth-and-jwt|tokens]]

---

## Development Phase: Tailscale Mesh

### Setup
- Install Tailscale on:
  - Development laptop
  - Gateway server
  - Instant GPU pod
  - Thinking GPU (if persistent)
- All devices join same Tailnet (private mesh)

### Access Model
```
┌─────────────────────────────────────────────┐
│              TAILSCALE MESH                 │
│                                             │
│  ┌────────┐    ┌─────────┐    ┌──────────┐ │
│  │ Laptop │◄──▶│ Gateway │◄──▶│ Instant  │ │
│  └────────┘    └────┬────┘    │   GPU    │ │
│                     │         └──────────┘ │
│                     │                      │
│                     ▼                      │
│              ┌──────────┐                  │
│              │ Thinking │                  │
│              │   GPU    │                  │
│              └──────────┘                  │
└─────────────────────────────────────────────┘
```

### Firewall Rules
- [[inference-instant|Inference]] endpoints: allow only from Tailscale IPs
- [[gateway-responsibilities|Gateway]]: allow HTTPS from anywhere (or just Tailscale for full private)
- [[Supabase]]: if [[supabase-local-dev-plan|local]], only localhost; if cloud, managed firewall

### Benefits
- Zero-config VPN
- No public IP exposure — see [[production-hardening]]
- Works across cloud providers
- MagicDNS for easy addressing

---

## Staging Phase: Hybrid

See [[environments-local-staging-prod]] for environment comparison.

### If Staging is Home-Based
- Spare MacBook on Tailscale mesh
- [[gateway-responsibilities|Gateway]] publicly accessible (HTTPS)
- [[inference-instant|Inference]] still private via Tailscale

### If Staging is Cloud-Based
- VPC with private subnets
- Gateway in public subnet with load balancer
- Inference in private subnet, no public IP
- Security groups restrict access

---

## Production Phase: Full Isolation

### Architecture
```
┌───────────────────────────────────────────────────┐
│                  PUBLIC INTERNET                  │
└───────────────────────────┬───────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Load Balancer │
                    │    (HTTPS)    │
                    └───────┬───────┘
                            │
┌───────────────────────────┼───────────────────────┐
│              PRIVATE VPC  │                       │
│                           ▼                       │
│                   ┌───────────────┐               │
│                   │    Gateway    │               │
│                   │  (Public SN)  │               │
│                   └───────┬───────┘               │
│                           │                       │
│           ┌───────────────┼───────────────┐       │
│           ▼               ▼               ▼       │
│     ┌──────────┐   ┌──────────┐   ┌───────────┐  │
│     │ Instant  │   │ Thinking │   │ Supabase  │  │
│     │   GPU    │   │   GPU    │   │   Cloud   │  │
│     │(Priv SN) │   │(Priv SN) │   │           │  │
│     └──────────┘   └──────────┘   └───────────┘  │
└───────────────────────────────────────────────────┘
```

### Security Groups / Firewall

| From | To | Port | Allow |
|------|----|------|-------|
| Internet | Load Balancer | 443 | Yes |
| Load Balancer | Gateway | 8000 | Yes |
| Gateway | Instant GPU | 8080 | Yes |
| Gateway | Thinking GPU | 8080 | Yes |
| Gateway | Supabase | 5432 | Yes |
| Anything else | * | * | Deny |

---

## DNS and Service Discovery

### Development
- Tailscale MagicDNS: `instant-gpu.tailnet`, `thinking-gpu.tailnet`
- Simple and automatic

### Production
- Internal DNS or service mesh
- Environment variables for endpoint URLs
- No hardcoded IPs

---

## TLS / Encryption

### In Transit
- All public traffic: HTTPS (TLS 1.3)
- Internal traffic: TLS or mTLS if required
- Tailscale: encrypted by default

### At Rest
- Supabase: encrypted by default
- GPU pods: ephemeral, no persistent storage recommended

---

## Incident Response

### If Inference Endpoint is Exposed
1. Immediately rotate any credentials
2. Block public access
3. Audit logs for unauthorized access
4. Reset model state (restart pod)

### If Gateway is Compromised
1. Revoke all JWTs (rotate Supabase JWT secret)
2. Block compromised IP
3. Audit database for tampering
4. Restore from backup if needed

---

## Checklist: Networking Security

- [ ] Inference endpoints have no public IP
- [ ] Firewall rules explicitly allow only Gateway
- [ ] Tailscale mesh configured for dev
- [ ] HTTPS enforced on all public endpoints
- [ ] DNS/service discovery documented
- [ ] Incident response plan written


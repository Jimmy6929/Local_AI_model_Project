---
tags:
  - readme
cssclasses:
  - readme-doc
---

# Glossary

## Core Concepts

### Gateway / Control Plane
The central backend service that owns:
- Authentication and authorization — see [[auth-and-jwt]]
- Request routing (instant vs thinking) — see [[routing-logic]]
- Session and chat management — see [[endpoints-contracts]]
- Tool execution — see [[tools-framework]]
- RAG retrieval — see [[rag-flow]]
- Safety policies and logging — see [[logging-and-auditing]]

For full details, see [[gateway-responsibilities]] and [[openclaw-style-gateway-notes]].

### Instant Mode
- Fast, cheap inference for most daily tasks — see [[inference-instant]]
- [[adr-0001-two-tier-inference|Always-on GPU pod]] infrastructure
- Low latency, responsive experience
- Handles coding, writing, planning, general Q&A

### Thinking Mode
- Stronger model for complex reasoning — see [[inference-thinking]]
- [[adr-0001-two-tier-inference|Serverless GPU]] (scale-to-zero)
- Cold start acceptable (user expects to wait)
- Used for deep analysis, difficult problems

See [[routing-logic]] for how the [[Gateway]] chooses between modes.

---

## Data and Storage

### RAG (Retrieval-Augmented Generation)
A pattern where:
1. User query is embedded — see [[embeddings-and-pgvector]]
2. Vector search finds relevant stored chunks
3. Only minimal, relevant context is injected into the prompt
4. Model generates response with grounded information

For implementation details, see [[rag-flow]] and [[storage-and-files]].

### RLS (Row Level Security)
Database-level access control:
- Policies enforce that users can only access their own rows
- Uses `auth.uid()` pattern in [[Supabase]]
- Critical for [[Multi-Tenant|multi-tenant]] safety

For implementation details, see [[rls-policies]] and [[adr-0004-rls-multi-tenant-from-day-1]].

### Multi-Tenant
A multi-user architecture where:
- Many users share one database
- Isolation is enforced via `user_id` column + [[rls-policies|RLS policies]]
- No data leakage between users

See [[schema]] for how tables are designed for multi-tenancy.

---

## Infrastructure

### GPU Pod
- Dedicated GPU instance running continuously
- Used for [[Instant Mode]] inference — see [[inference-instant]]
- Cost: hourly rental, must monitor utilization — see [[cost-controls]]

### Serverless GPU
- GPU endpoint that scales to zero when idle
- Used for [[Thinking Mode]] — see [[inference-thinking]]
- Cost: pay-per-request, includes cold start overhead

### Tailscale
- Zero-config VPN mesh for private networking
- Connects laptop, [[Gateway]], and model servers
- Keeps inference endpoints private during development

For networking details, see [[networking-and-private-access]].

---

## Authentication

### JWT (JSON Web Token)
- Token issued by [[Supabase]] Auth after login
- Contains user identity and claims
- [[Gateway]] validates JWT on every request

For details, see [[auth-and-jwt]].

### Service Role Key
- Admin-level [[Supabase]] key for backend operations
- NEVER exposed to client-side code — see [[secrets-management]]
- Used only by [[Gateway]] for privileged operations

---

## Development Terms

### Local Supabase
- Full [[Supabase]] stack running locally via CLI
- Mirrors production capabilities (Auth, DB, Storage)
- Used for development and testing

See [[supabase-local-dev-plan]] and [[adr-0002-supabase-local-then-cloud]].

### Migration
- Version-controlled database [[schema]] change
- Applied sequentially: local → staging → production
- Tracked in repository for reproducibility

For migration workflow, see [[migrations-and-release-process]].


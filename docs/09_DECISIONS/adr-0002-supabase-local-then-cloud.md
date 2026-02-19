---
tags:
  - decision
cssclasses:
  - decision-doc
---

# ADR-0002: Supabase Local Development, Cloud Production

## Status
Accepted

## Context

We need a database, authentication, and storage solution that:
- Works in [[environments-local-staging-prod|local development]]
- Scales to production
- Supports [[adr-0004-rls-multi-tenant-from-day-1|multi-tenant architecture]]
- Minimizes operational complexity

Options considered:
1. [[Supabase]] (local CLI → cloud)
2. Separate solutions (Postgres + custom auth + S3)
3. Firebase
4. PlanetScale + Auth0 + S3

Related documents:
- [[supabase-local-dev-plan]] — Local setup guide
- [[schema]] — Database schema
- [[rls-policies]] — Security policies
- [[auth-and-jwt]] — Authentication

## Decision

Use **[[Supabase]]** for all data services:
- **Local:** [[supabase-local-dev-plan|Supabase CLI]] for development
- **Production:** Supabase Cloud

This includes:
- [[schema|PostgreSQL database]]
- [[auth-and-jwt|Authentication]]
- [[storage-and-files|Storage]]
- [[embeddings-and-pgvector|pgvector]] for embeddings
- [[rls-policies|Row Level Security]]

## Rationale

### Why Supabase

**Integrated Platform**
- Auth + Database + Storage in one
- Consistent API across all services
- Single dashboard and documentation

**Developer Experience**
- Local CLI mirrors production
- Auto-generated TypeScript types
- Real-time subscriptions (future use)

**Row Level Security**
- Database-enforced multi-tenancy
- Policies written in SQL
- Cannot be bypassed from application bugs

**pgvector Built-in**
- Native vector support
- No external vector DB needed
- Same RLS policies apply

### Why Not Separate Solutions

- More integration work
- Inconsistent auth patterns
- Multiple dashboards/docs to learn
- RLS coordination is harder

### Why Not Firebase

- Document database less suitable
- Vector search requires workarounds
- Google ecosystem lock-in

## Consequences

### Positive
- Fast development with local parity
- RLS ensures data isolation
- Single platform to learn and manage
- Easy production migration

### Negative
- Supabase-specific patterns
- Vendor dependency
- Some limits on free tier

### Neutral
- Must use SQL migrations
- Must learn RLS syntax
- Storage has bucket-based policies

## Migration Strategy

### Local → Staging
```
1. Run supabase migration new <name>
2. Test locally with supabase db reset
3. Apply to staging with supabase db push
```

### Staging → Production
```
1. Verify migrations work in staging
2. Create backup of production
3. Apply migrations to production
```

## Implementation Notes

- Store migrations in version control
- Never modify applied migrations
- Use seed.sql for development data only
- Service role key only in backend

## Related Decisions

- ADR-0004: RLS multi-tenant from day 1


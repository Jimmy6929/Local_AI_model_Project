---
tags:
  - decision
cssclasses:
  - decision-doc
---

# ADR-0004: RLS Multi-Tenant from Day 1

## Status
Accepted

## Context

The system will eventually support multiple users ([[phase-5-production-launch|multi-user]]). Data isolation options:
1. Separate databases per user
2. Schema-per-user
3. Shared tables with application-level filtering
4. Shared tables with [[rls-policies|Row Level Security (RLS)]]

Early architectural decisions affect migration difficulty later.

Related documents:
- [[rls-policies]] — Policy definitions
- [[schema]] — Table structures
- [[threat-model]] — Security context
- [[phase-5-production-launch]] — When this pays off

## Decision

Implement **[[rls-policies|Row Level Security (RLS)]]** on all user-data tables from day one, even during single-user development.

Every table containing user data includes:
- `user_id` column (FK to [[schema|profiles]])
- RLS enabled
- Policies using `auth.uid()` — see [[auth-and-jwt]]

## Rationale

### Why RLS

**Database-Enforced Security**
- Cannot be bypassed by application bugs
- Enforced at query level
- Defense in depth

**Multi-User Ready Architecture**
- No migration needed for multi-user
- Patterns established early
- Testing happens throughout development

**Simpler Application Code**
- No manual filtering required
- Queries naturally scoped to user
- Fewer places for mistakes

### Why Day 1

**Avoiding Migration Pain**
- Adding RLS later requires:
  - Backfilling user_id
  - Writing all policies
  - Testing everything again
- Much harder than starting with it

**Building Good Habits**
- Developers think multi-tenant
- Tests verify isolation
- Reviews check policies

**Catching Issues Early**
- Policy problems found in dev
- Performance impacts measured early
- Patterns refined over time

## Consequences

### Positive
- Security built-in — see [[threat-model]]
- No multi-tenant migration needed → [[phase-5-production-launch]]
- Application code simpler — see [[gateway-responsibilities]]
- Database catches mistakes

### Negative
- Slightly more initial setup
- Must understand [[rls-policies|RLS syntax]]
- Policies must be maintained
- Performance requires indexes — see [[schema]]

### Neutral
- Every table needs user_id — see [[schema]]
- Every table needs [[rls-policies|policies]]
- Testing requires multiple users

## Implementation Pattern

### Table Creation
```sql
CREATE TABLE items (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES profiles(id),
  content TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE items ENABLE ROW LEVEL SECURITY;

CREATE POLICY select_own ON items
  FOR SELECT USING (user_id = auth.uid());

CREATE POLICY insert_own ON items
  FOR INSERT WITH CHECK (user_id = auth.uid());

CREATE INDEX idx_items_user_id ON items(user_id);
```

### Testing Pattern
```
1. Create User A
2. Insert data as User A
3. Create User B
4. Query as User B
5. Verify User A's data not visible
```

## Performance Notes

- Index on user_id is critical
- RLS adds implicit WHERE clause
- Use EXPLAIN to verify index usage
- Compound indexes for common queries

## Related Decisions

- ADR-0002: Supabase local then cloud


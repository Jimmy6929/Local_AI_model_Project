---
tags:
  - database
cssclasses:
  - database-doc
---

# Migrations and Release Process

## Overview

Version-controlled [[schema|database changes]] applied consistently across [[environments-local-staging-prod|environments]].

For local Supabase setup, see [[supabase-local-dev-plan]].

---

## Migration Principles

### 1. Version Control
- All migrations in repository
- Timestamped filenames
- Never modify applied migrations

### 2. Sequential Application
- Migrations apply in order
- Each builds on previous
- No skipping allowed

### 3. Idempotent When Possible
- Use `IF NOT EXISTS`, `IF EXISTS`
- Safe to run multiple times
- Reduces deployment errors

### 4. Reversible (Ideally)
- Write both up and down
- Not always possible
- Document when irreversible

---

## Migration Workflow

### Creating a Migration

1. **Make changes locally** (via Studio or SQL)
2. **Generate diff**: `supabase db diff -f <name>`
3. **Review generated SQL**
4. **Edit if needed** (clean up, add indexes)
5. **Test locally**: `supabase db reset`
6. **Commit migration file**

### Naming Convention
```
YYYYMMDDHHMMSS_description.sql
```

Examples:
- `20260219120000_create_profiles_table.sql`
- `20260220093000_add_chat_sessions.sql`
- `20260221150000_add_rls_policies.sql`

---

## Migration File Structure

### Folder
```
supabase/
└── migrations/
    ├── 20260219120000_create_profiles_table.sql
    ├── 20260220093000_add_chat_sessions.sql
    └── 20260221150000_add_rls_policies.sql
```

### File Content Example
```
-- Migration: Create profiles table
-- Author: Jimmy
-- Date: 2026-02-19

-- Create table
CREATE TABLE IF NOT EXISTS profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT NOT NULL UNIQUE,
  name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY select_own ON profiles
  FOR SELECT USING (id = auth.uid());

CREATE POLICY update_own ON profiles
  FOR UPDATE USING (id = auth.uid())
  WITH CHECK (id = auth.uid());

-- Create index
CREATE INDEX IF NOT EXISTS idx_profiles_email ON profiles(email);
```

---

## Applying Migrations

### Local Development
```
supabase db reset
```
- Drops database
- Applies all migrations
- Runs seed.sql

### Staging / Production

**Option A: Supabase CLI**
```
supabase db push --db-url <connection_string>
```

**Option B: Manual Application**
1. Connect to database
2. Run migration SQL
3. Verify success
4. Record in migrations table

**Option C: CI/CD Pipeline**
- Automated on merge to main
- Runs migrations automatically
- Requires secure credential handling

---

## Release Checklist

### Before Releasing to Staging

- [ ] All migrations tested locally
- [ ] `supabase db reset` succeeds
- [ ] RLS policies verified
- [ ] No breaking changes without plan
- [ ] Rollback plan documented (if needed)

### Before Releasing to Production

- [ ] Staging tests pass
- [ ] Security review complete
- [ ] Backup created
- [ ] Off-peak timing (if possible)
- [ ] Monitoring ready
- [ ] Team notified

---

## Handling Breaking Changes

### Schema Changes

| Change Type | Approach |
|-------------|----------|
| Add column (nullable) | Safe, no migration needed |
| Add column (not null) | Add with default, then remove default |
| Remove column | Deploy code first, then remove column |
| Rename column | Add new, copy data, deploy, remove old |
| Change type | Create new column, migrate, swap |

### Policy Changes

| Change Type | Approach |
|-------------|----------|
| Relax policy | Safe, apply directly |
| Restrict policy | Verify no breaking queries first |
| Remove policy | Audit code for dependencies |

---

## Rollback Strategies

### Simple Rollback (Reverse Migration)
- Write reverse SQL
- Apply to undo changes
- Works for additive changes

### Data Backup Rollback
- Restore from backup
- Use point-in-time recovery
- Last resort, data loss possible

### Feature Flag Approach
- Deploy behind flag
- Enable gradually
- Disable if issues
- Remove after stable

---

## Seed Data

### Purpose
- Consistent development data
- Test scenarios
- Demo data for staging

### seed.sql Location
```
supabase/seed.sql
```

### Content
- Test users (with known UUIDs)
- Sample sessions and messages
- Test documents and chunks
- NOT production data

### Applied
- Automatically after `supabase db reset`
- NOT applied to staging/production

---

## Environment-Specific Migrations

### When Needed
- Different settings per environment
- Feature flags for staged rollout
- Debug helpers in dev only

### Approach
- Use environment variables in application
- Keep migrations identical across environments
- Use seed.sql for dev-only data

---

## Common Migration Tasks

### Add New Table
1. Create table
2. Enable RLS
3. Add policies
4. Add indexes

### Add Column
1. ALTER TABLE ADD COLUMN
2. Set default if needed
3. Update policies if column affects access

### Add Index
1. CREATE INDEX (use CONCURRENTLY in production)
2. Verify query performance improved

### Modify RLS Policy
1. DROP old policy
2. CREATE new policy
3. Test with multiple users

---

## Checklist: Migration Process

- [ ] Migration naming follows convention
- [ ] Migration tested locally
- [ ] RLS implications considered
- [ ] Indexes added for new queries
- [ ] Rollback plan exists
- [ ] Applied to staging first
- [ ] Production backup exists
- [ ] Post-migration verification done


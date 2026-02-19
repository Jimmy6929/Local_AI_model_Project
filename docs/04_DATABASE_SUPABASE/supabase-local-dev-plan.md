---
tags:
  - database
cssclasses:
  - database-doc
---

# Supabase Local Development Plan

## Overview

Run a full [[Supabase]] stack locally for development parity with production.

For environment strategy, see [[adr-0002-supabase-local-then-cloud]] and [[environments-local-staging-prod]].

---

## Why Local Supabase?

### Benefits
- Full feature parity with production
- Fast iteration (no network latency)
- Test RLS policies safely
- Test migrations before production
- No risk to production data

### What You Get Locally
- PostgreSQL database with pgvector
- Auth server with JWT
- Storage server with S3-compatible API
- Edge Functions runtime
- Studio dashboard (web UI)
- Realtime server
- PostgREST (auto-generated API)

---

## Prerequisites

### Required Tools
- Docker Desktop running
- Supabase CLI installed
- Node.js (for CLI)

### Install Supabase CLI
- Install via npm, brew, or direct download
- Verify with `supabase --version`

---

## Initial Setup

### Project Initialization
1. Navigate to project directory
2. Initialize Supabase: `supabase init`
3. Creates `supabase/` folder with config

### Folder Structure
```
project/
├── supabase/
│   ├── config.toml        # Local config
│   ├── migrations/        # SQL migrations — see [[migrations-and-release-process]]
│   ├── seed.sql          # Optional seed data
│   └── functions/        # Edge functions
├── .env.local            # Local [[secrets-management|environment vars]]
└── ...
```

---

## Running Local Stack

### Start Services
- Command: `supabase start`
- First run downloads Docker images
- Subsequent starts are faster

### What Starts
| Service | Local URL |
|---------|-----------|
| Studio | http://localhost:54323 |
| API | http://localhost:54321 |
| Database | localhost:54322 |
| Auth | http://localhost:54321/auth/v1 |
| Storage | http://localhost:54321/storage/v1 |

### Stop Services
- Command: `supabase stop`
- Data persists between stops
- Use `supabase stop --no-backup` to reset

---

## Environment Variables for Local Dev

### Required Variables
| Variable | Value |
|----------|-------|
| SUPABASE_URL | http://localhost:54321 |
| SUPABASE_ANON_KEY | (shown after `supabase start`) |
| SUPABASE_SERVICE_KEY | (shown after `supabase start`) |
| DATABASE_URL | postgresql://postgres:postgres@localhost:54322/postgres |

### Example .env.local
```
NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...local_anon_key
SUPABASE_SERVICE_ROLE_KEY=eyJ...local_service_key
```

---

## Working with Migrations

### Create New Migration
- Command: `supabase migration new <name>`
- Creates timestamped SQL file in `migrations/`

### Apply Migrations
- Command: `supabase db reset`
- Drops database and reapplies all migrations
- Also runs seed.sql if present

### Generate Migration from Diff
- Make changes in Studio
- Command: `supabase db diff -f <name>`
- Creates migration from database diff

---

## Testing RLS Policies

### Why Test Locally
- Safe to test with fake users
- Fast feedback loop
- Verify policies before production

### Testing Approach
1. Create test users via Auth
2. Generate JWTs for each user
3. Make API calls with different user tokens
4. Verify data isolation

### Common Patterns to Test
- User can only see own rows
- User can only insert with own user_id
- User cannot update others' data
- User cannot delete others' data

---

## Using pgvector Locally

### Enable Extension
- Add to migration: `CREATE EXTENSION IF NOT EXISTS vector;`
- Apply migration

### Create Vector Column
- Example: `embedding VECTOR(1536)`
- Index: `CREATE INDEX ON table USING ivfflat (embedding vector_cosine_ops)`

### Test Vector Search
- Insert test embeddings
- Query with similarity search
- Verify results ordered correctly

---

## Studio Dashboard

### Access
- Open http://localhost:54323
- Full database admin interface
- Table editor, SQL editor, Auth management

### Useful Features
- View and edit data directly
- Test SQL queries
- Manage auth users
- View storage buckets
- Test RLS policies

---

## Seed Data

### Purpose
- Populate database with test data
- Reproducible development environment
- Reset with `supabase db reset`

### seed.sql Example
```
-- Create test user profile
INSERT INTO profiles (id, email, name)
VALUES ('test-uuid', 'test@example.com', 'Test User');

-- Create test chat session
INSERT INTO chat_sessions (id, user_id, title)
VALUES ('session-uuid', 'test-uuid', 'Test Session');
```

---

## Common Issues and Fixes

### Docker Not Running
- Error: Cannot connect to Docker
- Fix: Start Docker Desktop

### Port Conflict
- Error: Port already in use
- Fix: Stop conflicting service or change port in config.toml

### Migration Fails
- Error: SQL syntax error
- Fix: Check migration file, test in SQL editor first

### Auth Not Working
- Issue: JWT validation fails
- Check: Using correct ANON_KEY from `supabase status`

---

## Checklist: Local Setup Complete

- [ ] Supabase CLI installed
- [ ] `supabase init` run
- [ ] `supabase start` successful
- [ ] Studio accessible
- [ ] Environment variables configured
- [ ] Initial migrations created
- [ ] RLS policies added and tested
- [ ] pgvector extension enabled


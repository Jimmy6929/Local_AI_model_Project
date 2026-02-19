---
tags:
  - database
cssclasses:
  - database-doc
---

# RLS Policies

## Overview

Row Level Security ensures users can only access their own data. This is critical for [[adr-0004-rls-multi-tenant-from-day-1|multi-tenant safety]].

For schema details, see [[schema]].
For threat context, see [[threat-model]].

---

## Core Principle

Every policy follows this pattern:
```
user_id = auth.uid()
```

Where `auth.uid()` returns the authenticated user's ID from the [[auth-and-jwt|JWT]].

---

## Enabling RLS

RLS must be enabled on every table with user data — see [[schema]]:
- `profiles`
- `chat_sessions`
- `chat_messages`
- `documents` — see [[storage-and-files]]
- `document_chunks` — see [[embeddings-and-pgvector]]

**Important:** When RLS is enabled, all access is denied by default. Policies explicitly grant access.

---

## Policy Definitions

### Table: profiles

| Policy | Operation | Using | With Check |
|--------|-----------|-------|------------|
| select_own | SELECT | id = auth.uid() | - |
| update_own | UPDATE | id = auth.uid() | id = auth.uid() |

**Notes:**
- No INSERT policy needed (profile created by trigger on signup)
- No DELETE policy (profiles not deletable via API)

---

### Table: chat_sessions

| Policy | Operation | Using | With Check |
|--------|-----------|-------|------------|
| select_own | SELECT | user_id = auth.uid() | - |
| insert_own | INSERT | - | user_id = auth.uid() |
| update_own | UPDATE | user_id = auth.uid() | user_id = auth.uid() |
| delete_own | DELETE | user_id = auth.uid() | - |

**Notes:**
- User can CRUD only their own sessions
- WITH CHECK ensures new rows have correct user_id

---

### Table: chat_messages

| Policy | Operation | Using | With Check |
|--------|-----------|-------|------------|
| select_own | SELECT | user_id = auth.uid() | - |
| insert_own | INSERT | - | user_id = auth.uid() |

**Notes:**
- No UPDATE/DELETE for messages (immutable history)
- If needed, add update_own for editing messages

---

### Table: documents

| Policy | Operation | Using | With Check |
|--------|-----------|-------|------------|
| select_own | SELECT | user_id = auth.uid() | - |
| insert_own | INSERT | - | user_id = auth.uid() |
| delete_own | DELETE | user_id = auth.uid() | - |

**Notes:**
- No UPDATE for documents (status updates via service role)
- Deleting document should cascade to chunks

---

### Table: document_chunks

| Policy | Operation | Using | With Check |
|--------|-----------|-------|------------|
| select_own | SELECT | user_id = auth.uid() | - |
| insert_own | INSERT | - | user_id = auth.uid() |
| delete_own | DELETE | user_id = auth.uid() | - |

**Notes:**
- Chunks are created during processing (may need service role)
- Users can read for RAG, delete for cleanup

---

## Service Role Access

The service role key bypasses RLS. Used by Gateway for:
- Processing documents (updating status)
- Admin operations
- Background jobs

**Security:**
- Service role key NEVER exposed to client
- Only used in backend code
- Environment variable, not hardcoded

---

## Testing RLS

### Test Matrix

| Table | Operation | Own Data | Other User Data |
|-------|-----------|----------|-----------------|
| profiles | SELECT | ✅ Pass | ❌ Empty result |
| profiles | UPDATE | ✅ Pass | ❌ No rows affected |
| chat_sessions | SELECT | ✅ Pass | ❌ Empty result |
| chat_sessions | INSERT | ✅ Pass | ❌ RLS violation error |
| chat_messages | SELECT | ✅ Pass | ❌ Empty result |
| documents | SELECT | ✅ Pass | ❌ Empty result |
| document_chunks | SELECT | ✅ Pass | ❌ Empty result |

### Testing Method

1. Create two test users
2. Insert data for each user
3. Query as User A, verify only User A data returned
4. Attempt cross-user access, verify blocked
5. Attempt insert with wrong user_id, verify error

---

## Common RLS Pitfalls

### Pitfall 1: Forgetting to Enable RLS
- Table is public by default
- ALWAYS enable RLS after table creation

### Pitfall 2: Missing Index on user_id
- RLS checks run on every row
- Without index, full table scan
- Add index: `CREATE INDEX ON table (user_id)`

### Pitfall 3: Incorrect WITH CHECK
- Must use WITH CHECK for INSERT/UPDATE
- Prevents inserting rows user can't read

### Pitfall 4: JOIN Leakage
- Joining to another table may leak data
- Ensure all joined tables have RLS

### Pitfall 5: Function Security
- Database functions may bypass RLS
- Use SECURITY DEFINER carefully
- Prefer SECURITY INVOKER

---

## Performance Considerations

### Indexes Required
- Every table: index on `user_id`
- Compound indexes for common queries:
  - `chat_sessions`: `(user_id, created_at DESC)`
  - `chat_messages`: `(session_id, created_at ASC)`
  - `document_chunks`: `(user_id)` for RLS, `(document_id)` for retrieval

### Query Patterns
- Always filter by user_id first
- RLS adds implicit filter, but explicit is clearer
- Use EXPLAIN to verify index usage

---

## Audit Checklist

Before production:
- [ ] RLS enabled on all user tables
- [ ] Every table has SELECT policy
- [ ] INSERT policies use WITH CHECK
- [ ] No UPDATE/DELETE policies where not needed
- [ ] Indexes exist on user_id columns
- [ ] Tested with multiple users
- [ ] Service role key secured
- [ ] No public functions that bypass RLS


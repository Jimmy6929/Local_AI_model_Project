---
tags:
  - database
cssclasses:
  - database-doc
---

# Database Schema

## Overview

[[adr-0004-rls-multi-tenant-from-day-1|Multi-tenant schema]] designed for SaaS from day one. All user-owned tables include `user_id` for [[rls-policies|RLS]].

For migration workflow, see [[migrations-and-release-process]].

---

## Table Diagram

```
┌─────────────┐       ┌───────────────────┐       ┌─────────────────┐
│   profiles  │       │   chat_sessions   │       │  chat_messages  │
├─────────────┤       ├───────────────────┤       ├─────────────────┤
│ id (PK)     │◄──────│ user_id (FK)      │       │ id (PK)         │
│ email       │       │ id (PK)           │◄──────│ session_id (FK) │
│ name        │       │ title             │       │ user_id (FK)    │
│ avatar_url  │       │ created_at        │       │ role            │
│ created_at  │       │ updated_at        │       │ content         │
│ updated_at  │       │ is_archived       │       │ mode_used       │
└─────────────┘       └───────────────────┘       │ tokens_used     │
                                                  │ created_at      │
                                                  └─────────────────┘

┌─────────────────┐       ┌───────────────────────┐
│    documents    │       │    document_chunks    │
├─────────────────┤       ├───────────────────────┤
│ id (PK)         │◄──────│ document_id (FK)      │
│ user_id (FK)    │       │ id (PK)               │
│ filename        │       │ user_id (FK)          │
│ storage_path    │       │ content               │
│ file_type       │       │ embedding (vector)    │
│ file_size       │       │ chunk_index           │
│ status          │       │ created_at            │
│ created_at      │       └───────────────────────┘
│ processed_at    │
└─────────────────┘
```

---

## Table: profiles

Stores user profile information. Created automatically on signup via [[Supabase]] Auth.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PK, FK to auth.users | User ID from [[auth-and-jwt|Supabase Auth]] |
| email | text | not null, unique | User email |
| name | text | | Display name |
| avatar_url | text | | Profile picture URL |
| created_at | timestamptz | default now() | Account creation time |
| updated_at | timestamptz | default now() | Last profile update |

**Indexes:**
- Primary key on `id`

**RLS Policy:** see [[rls-policies]]
- Users can only read/update their own profile

---

## Table: chat_sessions

Stores chat conversation sessions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PK, default gen_random_uuid() | Session ID |
| user_id | uuid | FK to profiles, not null | Owner |
| title | text | default 'New Chat' | Session title |
| created_at | timestamptz | default now() | Creation time |
| updated_at | timestamptz | default now() | Last message time |
| is_archived | boolean | default false | Soft delete flag |

**Indexes:**
- Primary key on `id`
- Index on `user_id, created_at DESC` (list sessions)

**RLS Policy:**
- Users can only access their own sessions

---

## Table: chat_messages

Stores individual messages within sessions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PK, default gen_random_uuid() | Message ID |
| session_id | uuid | FK to chat_sessions, not null | Parent session |
| user_id | uuid | FK to profiles, not null | Owner (denormalized) |
| role | text | not null, check in ('user','assistant','system') | Message role |
| content | text | not null | Message content |
| mode_used | text | check in ('instant','thinking') | Which inference tier |
| tokens_used | integer | | Token count for billing/tracking |
| created_at | timestamptz | default now() | Message time |

**Indexes:**
- Primary key on `id`
- Index on `session_id, created_at ASC` (message history)
- Index on `user_id` (RLS performance)

**RLS Policy:**
- Users can only access their own messages

---

## Table: documents

Stores metadata about uploaded files.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PK, default gen_random_uuid() | Document ID |
| user_id | uuid | FK to profiles, not null | Owner |
| filename | text | not null | Original filename |
| storage_path | text | not null | Path in Supabase Storage |
| file_type | text | not null | MIME type |
| file_size | integer | not null | Size in bytes |
| status | text | default 'pending' | processing status |
| created_at | timestamptz | default now() | Upload time |
| processed_at | timestamptz | | When processing completed |

**Indexes:**
- Primary key on `id`
- Index on `user_id, created_at DESC`

**Status Values:**
- `pending` — uploaded, not yet processed
- `processing` — text extraction in progress
- `completed` — chunks created, ready for RAG
- `failed` — processing error

**RLS Policy:**
- Users can only access their own documents

---

## Table: document_chunks

Stores text chunks with embeddings for RAG.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | uuid | PK, default gen_random_uuid() | Chunk ID |
| document_id | uuid | FK to documents, not null | Parent document |
| user_id | uuid | FK to profiles, not null | Owner (denormalized) |
| content | text | not null | Chunk text |
| embedding | vector(1536) | | Embedding vector |
| chunk_index | integer | not null | Order within document |
| created_at | timestamptz | default now() | Creation time |

**Indexes:**
- Primary key on `id`
- Index on `document_id, chunk_index` (reconstruct order)
- Index on `user_id` (RLS performance)
- IVFFlat index on `embedding` (vector search)

**RLS Policy:**
- Users can only access their own chunks

---

## Design Decisions

### Why `user_id` on Every Table
- Enables simple RLS policies
- Avoids complex joins for permission checks
- Denormalization worth the consistency benefit

### Why Soft Delete for Sessions
- Users may want to restore
- Archive instead of delete
- Permanent delete via separate process

### Why Denormalized `user_id` on Messages and Chunks
- RLS performance: no join needed
- Simpler policies
- Storage cost minimal

### Why `mode_used` on Messages
- Track usage patterns
- Enable billing by mode (future)
- Help user understand their usage

---

## Future Additions

### For Tools
```
Table: tool_calls
- id, message_id, user_id, tool_name, input, output, status, created_at
```

### For Teams (Future SaaS)
```
Table: organizations
- id, name, created_at

Table: memberships
- user_id, org_id, role
```

---

## Migration Notes

- All tables should be created in order (profiles → sessions → messages)
- Foreign keys ensure referential integrity
- RLS must be enabled immediately after table creation
- Indexes critical for query performance


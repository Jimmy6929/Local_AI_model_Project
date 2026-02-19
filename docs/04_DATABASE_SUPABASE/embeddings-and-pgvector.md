---
tags:
  - database
cssclasses:
  - database-doc
---

# Embeddings and pgvector

## Overview

Store and search vector embeddings using pgvector extension in [[Supabase]] Postgres.

For RAG flow, see [[rag-flow]].
For schema details, see [[schema]].

---

## What Are Embeddings?

Embeddings are numerical representations of text:
- Convert text into a vector of numbers
- Similar text has similar vectors
- Enable semantic search (meaning, not just keywords)

Example:
- "How to cook pasta" → [0.12, -0.34, 0.56, ...]
- "Pasta cooking instructions" → [0.11, -0.33, 0.55, ...]
- These vectors are close together (cosine similarity ~0.98)

---

## pgvector Extension

### What It Provides
- Vector data type: `VECTOR(dimensions)`
- Distance functions: cosine, L2, inner product
- Index types: IVFFlat, HNSW

### Enable in Database
- Add to [[migrations-and-release-process|migration]]: `CREATE EXTENSION IF NOT EXISTS vector;`
- Available in [[Supabase]] Postgres by default

---

## Embedding Dimensions

| Model | Dimensions |
|-------|------------|
| OpenAI text-embedding-3-small | 1536 |
| OpenAI text-embedding-3-large | 3072 |
| Sentence Transformers | 384-768 |
| Cohere embed-v3 | 1024 |

Choose based on model used. **Recommendation:** 1536 for compatibility.

---

## Storage Schema

### document_chunks Table

See [[schema]] for full table details.

| Column | Type | Purpose |
|--------|------|--------|
| id | uuid | Primary key |
| document_id | uuid | Parent [[storage-and-files|document]] |
| user_id | uuid | Owner (for [[rls-policies|RLS]]) |
| content | text | Chunk text |
| embedding | vector(1536) | Embedding vector |
| chunk_index | integer | Order in document |

---

## Indexing Strategies

### IVFFlat Index
- Good for most use cases
- Faster queries, approximate results
- Requires training (specify lists)

```
CREATE INDEX ON document_chunks 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```

### HNSW Index
- Better recall, slightly slower
- No training required
- Good for smaller datasets

```
CREATE INDEX ON document_chunks 
USING hnsw (embedding vector_cosine_ops);
```

### Recommendation
- Start with IVFFlat (simpler)
- Lists = sqrt(row_count) roughly
- Re-index as data grows

---

## Query Pattern: Similarity Search

### Goal
Find chunks most similar to a query embedding.

### Steps
1. Embed the user query
2. Run similarity search
3. Return top K results

### Query Structure
```
SELECT id, content, 
       1 - (embedding <=> query_vector) as similarity
FROM document_chunks
WHERE user_id = $1
ORDER BY embedding <=> query_vector
LIMIT 5;
```

### Operators
- `<=>` — Cosine distance (1 - similarity)
- `<->` — L2 distance (Euclidean)
- `<#>` — Inner product (negative)

---

## Embedding Generation

### Options

**Option A: External API**
- OpenAI Embeddings API
- Cost: ~$0.0001 per 1K tokens
- Pros: High quality, easy
- Cons: API dependency, cost

**Option B: Self-Hosted**
- Run embedding model on GPU
- Same GPU as Instant inference
- Pros: No API cost, privacy
- Cons: Complexity, latency

### Recommendation for MVP
Use OpenAI embeddings for simplicity. Migrate to self-hosted later if cost is concern.

---

## Chunking Strategy

### Why Chunk?
- Embeddings have context window limits
- Smaller chunks are more precise
- Enables retrieving only relevant parts

### Chunk Size
| Strategy | Size | Use Case |
|----------|------|----------|
| Small | 200-500 tokens | Precise retrieval |
| Medium | 500-1000 tokens | Balanced |
| Large | 1000-2000 tokens | More context per chunk |

### Overlap
- Overlap chunks by 10-20%
- Prevents cutting sentences mid-thought
- Example: 500 token chunks with 50 token overlap

### MVP Approach
- Fixed size: 500 tokens
- Overlap: 50 tokens
- Split on sentence boundaries when possible

---

## RAG Flow (Retrieval-Augmented Generation)

### Full Pipeline

1. **User sends query**
2. **Gateway embeds query**
3. **Gateway searches chunks** (user's documents only)
4. **Top K chunks retrieved** (K=3-5)
5. **Chunks added to prompt** as context
6. **Model generates response** using context
7. **Response includes citations** (optional)

### Context Injection Format
```
Context from your documents:

[Source 1: report.pdf]
{chunk content}

[Source 2: notes.md]
{chunk content}

---

User question: {query}
```

---

## Performance Considerations

### Query Latency
- Index type affects speed
- IVFFlat: ~1-10ms for 100K rows
- Filter by user_id first (RLS)

### Storage Size
- 1536 dimensions × 4 bytes = ~6KB per vector
- 10,000 chunks = ~60MB vectors
- Plan for growth

### Batch Embedding
- Embed multiple chunks in one API call
- Reduces round trips
- Up to 2048 inputs per batch (OpenAI)

---

## Checklist: pgvector Setup

- [ ] pgvector extension enabled
- [ ] document_chunks table created
- [ ] Vector column with correct dimensions
- [ ] IVFFlat or HNSW index created
- [ ] RLS policy on document_chunks
- [ ] Embedding API configured
- [ ] Chunking logic implemented
- [ ] Similarity search query working
- [ ] RAG flow integrated with chat


---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# RAG Flow

## Overview

Retrieval-Augmented Generation: enhance model responses with relevant document context.

For implementation details, see:
- [[embeddings-and-pgvector]] — Vector storage
- [[storage-and-files]] — File uploads
- [[phase-3-rag]] — Roadmap phase

---

## Why RAG?

### Problem
- Models have knowledge cutoff
- Models don't know user's private data
- Long documents don't fit in context

### Solution
- Store documents as embedded chunks
- Retrieve only relevant chunks
- Inject minimal context into prompt
- Model generates grounded response

---

## RAG Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCUMENT INGESTION                           │
│                                                                 │
│  Upload → Extract → Chunk → Embed → Store                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    QUERY TIME                                   │
│                                                                 │
│  Query → Embed → Search → Retrieve → Inject → Generate          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: Document Ingestion

### Upload
- User uploads file via [[endpoints-contracts|/files endpoint]] — see [[file-upload-ux]]
- File stored in [[storage-and-files|Supabase Storage]]
- [[schema|Document record]] created (status: pending)

### Extract Text
| File Type | Extraction Method |
|-----------|-------------------|
| .txt | Read directly |
| .md | Read directly |
| .pdf | PDF parsing library |
| .docx | Document parsing library |

### Chunk
- Split text into segments
- Size: ~500 tokens per chunk
- Overlap: ~50 tokens between chunks
- Preserve sentence boundaries

### Embed
- Send chunks to embedding API/model
- Receive vector for each chunk
- Dimensions: 1536 (OpenAI compatible)

### Store
- Insert into [[schema|document_chunks table]] — see [[embeddings-and-pgvector]]
- Include: document_id, user_id, content, embedding, chunk_index
- Update document status: completed
- [[rls-policies|RLS]] ensures user isolation

---

## Step 2: Query-Time Retrieval

### Embed Query
- Take user's message
- Generate embedding vector
- Same model as document embedding

### Vector Search
- Search document_chunks WHERE user_id = current user
- Order by cosine similarity
- Return top K chunks (K = 3-5)

### Relevance Filtering
- Optional: threshold on similarity score
- Discard chunks below threshold
- Prevents injecting irrelevant context

### Retrieve Content
- Load chunk text from results
- Include source metadata (filename, chunk_index)

---

## Step 3: Context Injection

### Prompt Structure
```
<system>
You are a helpful assistant. Use the following context to answer questions.
If the context doesn't contain relevant information, say so.
</system>

<context>
[Source: report.pdf, Section 2]
{chunk 1 content}

[Source: notes.md, Paragraph 5]
{chunk 2 content}

[Source: report.pdf, Section 7]
{chunk 3 content}
</context>

<history>
{recent conversation history}
</history>

<user>
{current user message}
</user>
```

### Token Budget
| Component | Allocation |
|-----------|------------|
| System prompt | ~200 tokens |
| Context | ~1500 tokens (3 chunks × 500) |
| History | ~500 tokens (recent messages) |
| User message | ~200 tokens |
| Response buffer | ~1600 tokens |
| **Total** | ~4000 tokens |

---

## Step 4: Generate Response

### Send to Model
- Constructed prompt goes to inference
- Model generates response using context
- Response should reference context when relevant

### Citation Tracking
- Track which chunks were included
- Return citation metadata to client
- UI can show sources

---

## Configuration Options

### RAG Settings
| Setting | Default | Description |
|---------|---------|-------------|
| enable_rag | true | Include document context |
| top_k | 5 | Number of chunks to retrieve |
| similarity_threshold | 0.7 | Minimum similarity score |
| max_context_tokens | 1500 | Token limit for context |

### Per-Request Override
```
{
  "enable_rag": false  // Skip document context
}
```

---

## Quality Considerations

### Chunking Strategy
- Too small: lose context
- Too large: include irrelevant text
- Balance: ~500 tokens with overlap

### Embedding Quality
- Better embeddings = better retrieval
- OpenAI embeddings are high quality
- Self-hosted may vary

### Retrieval Accuracy
- Top K may include false positives
- Similarity threshold helps
- Re-ranking (future) can improve

### Context Relevance
- Model may ignore irrelevant context
- Clear instructions help
- Monitor response quality

---

## Edge Cases

### No Documents
- User hasn't uploaded anything
- Skip RAG, respond from model knowledge
- Suggest uploading relevant documents

### No Relevant Chunks
- Query doesn't match any chunks
- Return empty context
- Model responds without grounding

### Too Much Context
- Many relevant chunks found
- Limit to top K within token budget
- Prioritize by similarity score

### Conflicting Information
- Different documents say different things
- Model should acknowledge conflict
- Cite both sources

---

## Monitoring RAG

### Metrics
| Metric | Purpose |
|--------|---------|
| Chunks retrieved per query | Usage pattern |
| Average similarity score | Quality indicator |
| RAG-enabled vs disabled | Feature usage |
| Citation click rate | User value |

### Quality Signals
- User thumbs up/down on responses
- Follow-up questions (may indicate poor retrieval)
- Document re-uploads (may indicate processing issues)

---

## Future Improvements

### Re-Ranking
- Initial retrieval gets top 20
- Re-ranker model scores for relevance
- Return top 5 after re-ranking

### Hybrid Search
- Combine vector search with keyword search
- BM25 + vector similarity
- Better handling of specific terms

### Multi-Document Reasoning
- Synthesize across multiple documents
- Handle contradictions explicitly
- Build knowledge graphs

---

## Checklist: RAG Implementation

- [ ] Document upload endpoint working
- [ ] Text extraction for supported types
- [ ] Chunking logic implemented
- [ ] Embedding generation working
- [ ] Chunks stored with vectors
- [ ] Query embedding working
- [ ] Vector search returns results
- [ ] Context injection in prompt
- [ ] Citations returned to client
- [ ] Enable/disable flag working


---
tags:
  - roadmap
cssclasses:
  - roadmap-doc
---

# Phase 3: RAG

## Goal

Add document upload and [[rag-flow|retrieval-augmented generation]] for context-aware responses.

For implementation details, see [[rag-flow]] and [[embeddings-and-pgvector]].
Previous: [[phase-2-two-modes]] | Next: [[phase-4-tools]]

---

## Success Criteria

- [ ] User can upload documents
- [ ] Documents are processed and chunked
- [ ] Embeddings stored in database
- [ ] Relevant context retrieved for queries
- [ ] Citations shown in responses

---

## Scope

### In Scope
- [[endpoints-contracts|File upload endpoint]]
- Text extraction (PDF, TXT, MD) — see [[storage-and-files]]
- Chunking and [[embeddings-and-pgvector|embedding]]
- Vector storage with [[embeddings-and-pgvector|pgvector]]
- [[rag-flow|Context retrieval]] in chat
- [[chat-ux|Citation display]] in UI

### Out of Scope
- Advanced file types (docx, xlsx)
- OCR for images
- Multi-modal RAG
- Re-ranking

---

## Milestones

### M3.1: File Upload
**Goal:** Users can upload documents

- POST /files endpoint
- Store file in Supabase Storage
- Create document record
- Validate file type and size

**Acceptance:** File uploads and appears in database

---

### M3.2: Text Extraction
**Goal:** Extract text from uploaded files

- PDF text extraction
- Plain text and markdown reading
- Handle extraction errors
- Update document status

**Acceptance:** Can extract text from test files

---

### M3.3: Chunking
**Goal:** Split documents into searchable chunks

- Define chunk size (~500 tokens)
- Implement overlap (~50 tokens)
- Preserve sentence boundaries
- Store with document reference

**Acceptance:** Document split into reasonable chunks

---

### M3.4: Embedding Generation
**Goal:** Generate embeddings for chunks

- Configure embedding API/model
- Embed all chunks
- Store in vector column
- Handle batching for efficiency

**Acceptance:** Chunks have embeddings in database

---

### M3.5: Vector Search
**Goal:** Retrieve relevant chunks

- Create pgvector index
- Implement similarity search
- Filter by user_id (RLS)
- Return top K results

**Acceptance:** Query returns relevant chunks

---

### M3.6: Context Injection
**Goal:** Use retrieved chunks in chat

- Embed user query
- Search for relevant chunks
- Format context for prompt
- Include in Gateway prompt building

**Acceptance:** Chat responses use document context

---

### M3.7: Citation Display
**Goal:** Show sources in UI

- Return citation metadata from Gateway
- Display sources in message
- Link to original document
- Expandable chunk preview

**Acceptance:** User can see what documents were used

---

## Technical Notes

### Schema Addition
```
documents
- id, user_id, filename, storage_path, status, created_at

document_chunks
- id, document_id, user_id, content, embedding (vector), chunk_index
```

### Processing Pipeline
```
Upload → Extract → Chunk → Embed → Store → Ready
```

### Retrieval Query
```
SELECT content, document.filename
FROM document_chunks
WHERE user_id = $1
ORDER BY embedding <=> $query_vector
LIMIT 5
```

---

## UX Details

### Files Page
- List uploaded files
- Show processing status
- Delete option
- Storage usage (future)

### Chat with RAG
```
[User] What does the report say about Q3 revenue?

[Assistant]
According to your documents, Q3 revenue was $1.2M...

📎 Sources:
• quarterly-report.pdf (page 5)
• revenue-summary.txt
```

---

## Dependencies

- Phase 1 & 2 complete
- Embedding API access (or self-hosted)
- pgvector extension enabled
- Storage bucket configured

---

## Risks

| Risk | Mitigation |
|------|------------|
| Extraction quality varies | Start with simple formats |
| Embedding API costs | Monitor usage, consider self-hosted |
| Retrieval quality poor | Test with real documents, tune parameters |

---

## Timeline

Estimated: 2-3 weeks

| Week | Focus |
|------|-------|
| 1 | M3.1 + M3.2 + M3.3 (Upload + Extract + Chunk) |
| 2 | M3.4 + M3.5 (Embed + Search) |
| 3 | M3.6 + M3.7 (Inject + Citations) |

---

## Exit Criteria

Phase 3 complete when:
- File upload working end-to-end
- RAG improving response quality
- Citations display correctly
- Ready to add tools


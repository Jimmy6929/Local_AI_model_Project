---
tags:
  - database
cssclasses:
  - database-doc
---

# Storage and Files

## Overview

[[Supabase]] Storage for document uploads, with access control via [[rls-policies|RLS-like policies]].

For UI implementation, see [[file-upload-ux]].
For [[rag-flow|RAG]] processing, see [[embeddings-and-pgvector]].

---

## Storage Architecture

```
┌─────────────────────────────────────────────────────┐
│                 SUPABASE STORAGE                    │
│  ┌───────────────────────────────────────────────┐ │
│  │            Bucket: documents                  │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │ user-uuid-1/                            │ │ │
│  │  │   ├── file1.pdf                         │ │ │
│  │  │   └── file2.txt                         │ │ │
│  │  ├─────────────────────────────────────────┤ │ │
│  │  │ user-uuid-2/                            │ │ │
│  │  │   └── report.pdf                        │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## Bucket Configuration

### Main Bucket: documents

| Setting | Value | Reason |
|---------|-------|--------|
| Name | documents | Clear purpose |
| Public | No | Private user data |
| File size limit | 50MB | Reasonable for docs |
| Allowed MIME types | pdf, txt, md, docx | MVP file types |

---

## Folder Structure Convention

### Pattern
```
{bucket}/{user_id}/{filename}
```

### Example
```
documents/a1b2c3d4-5678-90ab-cdef-1234567890ab/report.pdf
```

### Benefits
- Easy per-user isolation
- Simple policy rules
- Logical organization
- Scales with users

---

## Storage Policies

### Policy: User Can Upload to Own Folder
- **Operation:** INSERT
- **Check:** `bucket_id = 'documents' AND (storage.foldername(name))[1] = auth.uid()::text`

### Policy: User Can Read Own Files
- **Operation:** SELECT
- **Using:** `bucket_id = 'documents' AND (storage.foldername(name))[1] = auth.uid()::text`

### Policy: User Can Delete Own Files
- **Operation:** DELETE
- **Using:** `bucket_id = 'documents' AND (storage.foldername(name))[1] = auth.uid()::text`

---

## Upload Flow

### Web App → Gateway → Storage

1. **User selects file** in Web App
2. **Web App sends file** to Gateway
3. **Gateway validates:**
   - User authenticated
   - File type allowed
   - File size within limit
4. **Gateway uploads** to Storage
5. **Gateway creates** documents table entry
6. **Gateway returns** document ID to Web App

### Alternative: Direct Upload

1. **Web App requests** signed upload URL from Gateway
2. **Gateway generates** signed URL for user's folder
3. **Web App uploads** directly to Storage
4. **Web App notifies** Gateway of upload
5. **Gateway creates** documents entry and triggers processing

Direct upload is more efficient for large files.

---

## Download Flow

### Signed URL Pattern

1. **User requests** file download
2. **Gateway verifies** ownership via database
3. **Gateway generates** signed download URL
4. **URL returned** to Web App
5. **Web App redirects** user to signed URL
6. **File downloads** directly from Storage

### Signed URL Settings
| Setting | Value |
|---------|-------|
| Expiration | 60 seconds |
| Transform | None (original file) |

---

## File Processing Pipeline

### Trigger: New File Uploaded

1. **Create document record** with status = 'pending'
2. **Queue processing job**
3. **Job fetches file** from Storage
4. **Extract text** (based on file type)
5. **Chunk text** into segments
6. **Generate embeddings** for each chunk
7. **Store chunks** in document_chunks table
8. **Update document** status = 'completed'

### Processing by File Type

| Type | Extraction Method |
|------|-------------------|
| .txt | Read directly |
| .md | Read directly |
| .pdf | PDF parsing library |
| .docx | Document parsing library |

---

## Supported File Types (MVP)

| Extension | MIME Type | Max Size |
|-----------|-----------|----------|
| .txt | text/plain | 10MB |
| .md | text/markdown | 10MB |
| .pdf | application/pdf | 50MB |

### Future Types
- .docx, .doc (Word)
- .xlsx, .csv (Spreadsheets)
- Images (with OCR)

---

## Error Handling

### Upload Errors
| Error | Handling |
|-------|----------|
| File too large | Reject with size limit message |
| Invalid type | Reject with allowed types message |
| Storage full | Alert admin, user sees generic error |
| Network failure | Retry with exponential backoff |

### Processing Errors
| Error | Handling |
|-------|----------|
| Text extraction fails | Mark document as 'failed', notify user |
| Embedding fails | Retry, then mark 'failed' |
| Partial success | Store what succeeded, mark 'partial' |

---

## Cleanup and Retention

### Deleted Documents
- When user deletes document:
  1. Delete from documents table (cascades to chunks)
  2. Delete file from Storage
  3. Log deletion for audit

### Storage Limits (Future Multi-User)
- Track total storage per user
- Enforce tier-based limits
- Alert when approaching limit
- Block uploads when exceeded

---

## Checklist: Storage Setup

- [ ] Bucket created with correct settings
- [ ] Policies allow user folder access only
- [ ] Upload flow working end-to-end
- [ ] Download flow with signed URLs working
- [ ] File type validation in place
- [ ] File size limits enforced
- [ ] Processing pipeline functional
- [ ] Error handling tested
- [ ] Deletion removes file and records


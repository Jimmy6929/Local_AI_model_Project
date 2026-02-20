---
tags:
  - webapp
cssclasses:
  - webapp-doc
---

# File Upload UX

## Overview

User experience for uploading and managing documents for [[rag-flow|RAG]] context.

For backend implementation, see:
- [[storage-and-files]] — Storage configuration
- [[embeddings-and-pgvector]] — Processing pipeline
- [[endpoints-contracts]] — API endpoints

---

## Upload Methods

### Method 1: Button Click
- Prominent "Upload" button
- Opens file picker
- Single or multi-select

### Method 2: Drag and Drop
```
┌─────────────────────────────────────────┐
│                                         │
│     📂 Drop files here to upload        │
│                                         │
│     or click to browse                  │
│                                         │
└─────────────────────────────────────────┘
```

### Method 3: Paste (Future)
- Paste from clipboard
- Images and text

### Method 4: Chat Attachment
- Clip icon in message input
- Quick upload without leaving chat

---

## Upload Flow

### Step by Step
```
1. Select file(s)
       ↓
2. Validate (type, size)
       ↓
3. Show progress
       ↓
4. Upload to server
       ↓
5. Process (extract, chunk, embed)
       ↓
6. Show completion
```

### Progress Display
```
┌─────────────────────────────────────────┐
│ 📄 report.pdf                           │
│ [████████████░░░░░░░░] 65%              │
│ Uploading... 2.4 MB / 3.7 MB            │
└─────────────────────────────────────────┘
```

### Processing State
```
┌─────────────────────────────────────────┐
│ 📄 report.pdf                           │
│ [████████████████████] ✓ Uploaded       │
│ Processing document... ◌                │
└─────────────────────────────────────────┘
```

---

## File Validation

### Before Upload
| Check | Error Message |
|-------|---------------|
| File type | "Only PDF, TXT, and MD files are supported." |
| File size | "File exceeds 50MB limit." |
| File count | "Maximum 10 files per upload." |

### Visual Feedback
- Red border on invalid file
- Error message below
- Remove option

---

## Supported File Types

### MVP Types
| Type | Extension | Max Size |
|------|-----------|----------|
| PDF | .pdf | 50 MB |
| Plain text | .txt | 10 MB |
| Markdown | .md | 10 MB |

### Future Types
- Word documents (.docx)
- Spreadsheets (.xlsx, .csv)
- Images with OCR

### Unsupported
- Executables
- Archives (.zip)
- Media files

---

## File List UI

### Columns
```
┌────────────────────────────────────────────────────────────┐
│ Name          │ Size    │ Status      │ Date      │       │
├───────────────┼─────────┼─────────────┼───────────┼───────┤
│ 📄 report.pdf │ 3.7 MB  │ ✓ Ready     │ Feb 19    │ ⋮     │
│ 📄 notes.md   │ 24 KB   │ ◌ Processing│ Feb 19    │ ⋮     │
│ 📄 data.txt   │ 1.2 MB  │ ✗ Failed    │ Feb 18    │ ⋮     │
└───────────────┴─────────┴─────────────┴───────────┴───────┘
```

### Status Indicators
| Status | Icon | Color |
|--------|------|-------|
| Uploading | ◐ | Blue |
| Processing | ◌ | Yellow |
| Ready | ✓ | Green |
| Failed | ✗ | Red |

---

## File Actions

### Action Menu (⋮)
- Download original
- View details
- Retry processing (if failed)
- Delete

### Bulk Actions
- Select multiple
- Delete selected
- Download selected (zip)

---

## Error Handling

### Upload Errors
| Error | Display | Action |
|-------|---------|--------|
| Network failure | "Upload failed" | Retry button |
| Server error | "Upload failed" | Retry button |
| Rate limited | "Too many uploads" | Wait message |

### Processing Errors
| Error | Display | Action |
|-------|---------|--------|
| Can't read file | "Failed to process" | Retry or delete |
| Corrupt file | "File appears corrupt" | Re-upload |
| Timeout | "Processing timed out" | Retry |

---

## Integration with Chat

### Using Uploaded Documents
- RAG automatically includes relevant chunks
- Show which documents were cited
- Option to reference specific file

### Upload from Chat
```
User: [Clip icon clicked]
      ┌─────────────────────────┐
      │ Upload a file to        │
      │ include in conversation │
      │                         │
      │ [Select file]           │
      │ or drag here            │
      └─────────────────────────┘
```

---

## File Details Modal

### Display
```
┌─────────────────────────────────────────┐
│ report.pdf                          [X] │
├─────────────────────────────────────────┤
│                                         │
│ Type:       PDF Document                │
│ Size:       3.7 MB                      │
│ Uploaded:   Feb 19, 2026 at 2:30 PM     │
│ Status:     Ready for RAG               │
│ Chunks:     47 text chunks              │
│                                         │
│ [Download]  [Retry Processing]  [Delete]│
└─────────────────────────────────────────┘
```

---

## Storage Quota (Future Multi-User)

### Display
```
Storage Used: 45 MB / 100 MB
[████████████░░░░░░░░░░░░] 45%

Upgrade to Pro for 10 GB storage
```

### Near Limit
- Warning at 80%
- Block uploads at 100%
- Suggest upgrade or cleanup

---

## Performance

### Large Files
- Chunked upload
- Resume on failure
- Background processing

### Many Files
- Parallel upload (up to 3)
- Queue management
- Batch status updates

---

## Accessibility

### Requirements
- Keyboard accessible file picker
- Screen reader announcements
- Focus management
- Progress announcements

---

## Checklist: File Upload UX

- [ ] Upload button works
- [ ] Drag and drop works
- [ ] File validation with feedback
- [ ] Progress display working
- [ ] Processing status shown
- [ ] Error handling complete
- [ ] File list displaying
- [ ] Actions menu working
- [ ] Delete confirmation
- [ ] Details modal
- [ ] Integration with chat
- [ ] Mobile upload works
- [ ] Accessibility compliant


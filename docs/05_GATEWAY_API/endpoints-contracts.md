---
tags:
  - gateway
cssclasses:
  - gateway-doc
---

# API Endpoints and Contracts

## Overview

RESTful API contracts for [[gateway-responsibilities|Gateway]] endpoints. All endpoints require [[auth-and-jwt|JWT authentication]].

For request lifecycle, see [[gateway-responsibilities]] and [[data-flow]].

---

## Base URL

See [[environments-local-staging-prod]] for environment details.

| Environment | URL |
|-------------|-----|
| Local | http://localhost:8000 |
| Staging | https://staging-gateway.example.com |
| Production | https://api.example.com |

---

## Authentication

See [[auth-and-jwt]] for full auth flow.

All requests must include:
```
Authorization: Bearer <jwt_token>
```

Responses for auth failures:
- 401 Unauthorized — missing or invalid token
- 403 Forbidden — valid token, insufficient permissions — see [[rls-policies]]

---

## Endpoint: POST /chat

Send a message and receive AI response.

### Request
```
POST /chat
Content-Type: application/json

{
  "session_id": "uuid | null",
  "message": "string",
  "mode": "instant | think",
  "enable_rag": "boolean (default true)",
  "enable_tools": "boolean (default true)"
}
```

### Fields
| Field | Required | Description |
|-------|----------|-------------|
| session_id | No | Existing session, or null for new |
| message | Yes | User message content |
| mode | No | Force mode, default is instant |
| enable_rag | No | Include document context |
| enable_tools | No | Allow tool execution |

### Response (Success)
```
{
  "session_id": "uuid",
  "message_id": "uuid",
  "content": "string",
  "mode_used": "instant | think",
  "tokens_used": 150,
  "latency_ms": 1200,
  "citations": [
    {
      "source": "document_name.pdf",
      "chunk_id": "uuid",
      "snippet": "relevant text..."
    }
  ],
  "tool_calls": [
    {
      "tool": "web_search",
      "input": {"query": "..."},
      "output": "..."
    }
  ]
}
```

### Response Codes
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Invalid request body |
| 401 | Authentication failed |
| 403 | Session not owned by user |
| 429 | Rate limited |
| 500 | Server error |
| 504 | Inference timeout |

---

## Endpoint: GET /sessions

List user's chat sessions.

### Request
```
GET /sessions?limit=20&offset=0&archived=false
```

### Query Parameters
| Param | Default | Description |
|-------|---------|-------------|
| limit | 20 | Max sessions to return |
| offset | 0 | Pagination offset |
| archived | false | Include archived sessions |

### Response
```
{
  "sessions": [
    {
      "id": "uuid",
      "title": "string",
      "created_at": "ISO8601",
      "updated_at": "ISO8601",
      "message_count": 12,
      "is_archived": false
    }
  ],
  "total": 45,
  "has_more": true
}
```

---

## Endpoint: GET /sessions/{id}/messages

Get messages for a session.

### Request
```
GET /sessions/{session_id}/messages?limit=50&before=<message_id>
```

### Query Parameters
| Param | Default | Description |
|-------|---------|-------------|
| limit | 50 | Max messages to return |
| before | null | Pagination cursor |

### Response
```
{
  "messages": [
    {
      "id": "uuid",
      "role": "user | assistant",
      "content": "string",
      "mode_used": "instant | think | null",
      "created_at": "ISO8601"
    }
  ],
  "has_more": false
}
```

---

## Endpoint: PATCH /sessions/{id}

Update session metadata.

### Request
```
PATCH /sessions/{session_id}
Content-Type: application/json

{
  "title": "New Session Title",
  "is_archived": true
}
```

### Response
```
{
  "id": "uuid",
  "title": "New Session Title",
  "is_archived": true,
  "updated_at": "ISO8601"
}
```

---

## Endpoint: DELETE /sessions/{id}

Delete a session and all messages.

### Request
```
DELETE /sessions/{session_id}
```

### Response
```
{
  "deleted": true
}
```

---

## Endpoint: POST /files

Upload a document.

### Request
```
POST /files
Content-Type: multipart/form-data

file: <binary>
```

### Response
```
{
  "document_id": "uuid",
  "filename": "report.pdf",
  "status": "pending",
  "file_size": 1048576
}
```

### Notes
- File processing is async
- Poll status or use webhook for completion
- Max file size: 50MB

---

## Endpoint: GET /files

List user's documents.

### Request
```
GET /files?status=completed
```

### Response
```
{
  "documents": [
    {
      "id": "uuid",
      "filename": "report.pdf",
      "file_type": "application/pdf",
      "file_size": 1048576,
      "status": "completed",
      "created_at": "ISO8601"
    }
  ]
}
```

---

## Endpoint: DELETE /files/{id}

Delete a document and its chunks.

### Request
```
DELETE /files/{document_id}
```

### Response
```
{
  "deleted": true
}
```

---

## Endpoint: GET /health

Health check (no auth required).

### Request
```
GET /health
```

### Response
```
{
  "status": "healthy",
  "version": "1.0.0",
  "instant_inference": "connected",
  "thinking_inference": "connected",
  "database": "connected"
}
```

---

## Error Response Format

All errors follow this format:
```
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests, please try again later",
    "details": {
      "retry_after_seconds": 60
    }
  }
}
```

### Error Codes
| Code | HTTP Status | Meaning |
|------|-------------|---------|
| INVALID_REQUEST | 400 | Malformed request |
| UNAUTHORIZED | 401 | Auth failed |
| FORBIDDEN | 403 | Not allowed |
| NOT_FOUND | 404 | Resource not found |
| RATE_LIMITED | 429 | Too many requests |
| SERVER_ERROR | 500 | Internal error |
| INFERENCE_TIMEOUT | 504 | Model took too long |

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| POST /chat (instant) | 60/minute |
| POST /chat (think) | 10/minute |
| POST /files | 10/minute |
| GET endpoints | 120/minute |

Headers included in response:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1700000060
```


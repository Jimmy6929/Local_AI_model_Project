---
tags:
  - webapp
cssclasses:
  - webapp-doc
---

# Chat UX

## Overview

Design principles and interactions for the core chat experience.

For backend, see [[gateway-responsibilities]] and [[endpoints-contracts]].
For inference modes, see [[inference-instant]] and [[inference-thinking]].

---

## Core Principles

### 1. Instant Feedback
- Every action has immediate response
- Loading states prevent confusion
- Optimistic updates where safe

### 2. Minimal Friction
- Single input for everything
- Smart defaults
- Progressive disclosure

### 3. Context Awareness
- Remember user preferences
- Maintain conversation context — see [[schema|sessions and messages]]
- Show relevant information — see [[rag-flow|citations]]

### 4. Graceful Degradation
- Work offline (read mode)
- Handle errors elegantly
- Provide clear recovery paths

---

## Message Input

### Behavior
- Auto-resize with content
- Submit on Enter (configurable)
- Shift+Enter for new line
- Max length indicator

### States
| State | Display |
|-------|---------|
| Empty | Placeholder text |
| Typing | Character count (if near limit) |
| Sending | Disabled, spinner |
| Error | Red border, error message |

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| Enter | Send message |
| Shift+Enter | New line |
| Esc | Clear input |
| Ctrl+/ | Toggle Deep Think |

---

## Message Display

### Rendering
- Markdown support
- Syntax highlighting for code
- Link detection
- Image preview (future)

### Timestamps
- Relative for recent ("2 min ago")
- Absolute for older ("Feb 19, 2026")
- Tooltip with full timestamp

### Long Messages
- Expand/collapse for very long
- "Show more" button
- Smooth animation

---

## Thinking Mode UX

### Toggle Design
- Clearly visible but not intrusive
- Brain icon or similar
- Tooltip explaining purpose

### When Active
```
┌─────────────────────────────────────────┐
│ 🧠 Deep Thinking...                     │
│                                         │
│ [Animated indicator]                    │
│                                         │
│ This may take 30 seconds to a minute.   │
│ The AI is doing deeper analysis.        │
└─────────────────────────────────────────┘
```

### Response Display
- "Thinking Mode" badge on message
- Different visual treatment (subtle)
- Show time taken

---

## Session Management

### New Session
- "New Chat" button always visible
- Keyboard shortcut (Ctrl+N)
- Confirm if current has unsent text

### Session Switching
- Click to switch
- Load messages progressively
- Maintain scroll position per session

### Session Title
- Auto-generate from first message
- Click to edit
- Update on significant topic change (future)

### Session Organization
- Recent first
- Archive old sessions
- Search across sessions (future)

---

## RAG and Citations

### When Citing Documents
```
┌─────────────────────────────────────────┐
│ Based on your documents, here's what    │
│ I found...                              │
│                                         │
│ [content with citations]                │
│                                         │
│ 📎 Sources:                             │
│   • report.pdf (section 2)              │
│   • notes.md (paragraph 5)              │
└─────────────────────────────────────────┘
```

### Citation Interaction
- Click to expand source
- Show relevant chunk
- Link to original document

---

## Tool Usage Display

### When Tools Are Used
```
┌─────────────────────────────────────────┐
│ Let me search for that...               │
│                                         │
│ 🔍 Searching web for "AI trends 2026"   │
│                                         │
│ [Results incorporated into response]    │
│                                         │
│ 🔧 Used: web_search                     │
│   └─ Click to see details               │
└─────────────────────────────────────────┘
```

### Tool Details (Expandable)
- Tool name
- Input provided
- Output received
- Time taken

---

## Error Handling

### Network Errors
- Toast notification
- Retry button
- Message marked as failed
- Option to resend

### Rate Limiting
- Clear message about limit
- Countdown to retry
- Suggest alternatives

### Model Errors
- Apologetic message
- Retry option
- Fall back to other mode (if possible)

---

## Streaming Responses

### Display
- Show text as it arrives
- Cursor animation
- Smooth scrolling

### Interruption
- Stop button visible during stream
- Cancel cleanly
- Keep partial response

---

## Accessibility

### Requirements
- Screen reader compatible
- Keyboard navigable
- High contrast mode
- Focus indicators

### Implementation
- ARIA labels on controls
- Semantic HTML
- Skip links
- Announce new messages

---

## Performance

### Optimizations
- Virtual list for long history
- Lazy load old messages
- Debounce typing events
- Cache session data

### Targets
| Metric | Target |
|--------|--------|
| Time to interactive | < 2 seconds |
| Input response | < 50ms |
| Message appear | < 100ms |
| Session switch | < 500ms |

---

## Empty States

### No Sessions
```
Welcome to Local AI!
Start a new chat to begin.
[New Chat]
```

### No Messages
```
What can I help you with today?

Suggestions:
• Help me write code
• Analyze this document
• Plan my project
```

### No Files
```
Upload documents to give the AI
context about your work.
[Upload File]
```

---

## Checklist: Chat UX

- [ ] Message input responsive
- [ ] Markdown rendering working
- [ ] Code syntax highlighting
- [ ] Thinking mode UI complete
- [ ] Session sidebar functional
- [ ] Citations display working
- [ ] Tool usage visible
- [ ] Error handling graceful
- [ ] Loading states smooth
- [ ] Keyboard shortcuts working
- [ ] Accessibility audit passed
- [ ] Performance targets met


---
tags:
  - webapp
cssclasses:
  - webapp-doc
---

# UI Pages and Components

## Overview

Web application structure for the chat interface.

For related UX docs, see:
- [[chat-ux]] — Chat interaction design
- [[auth-ux]] — Authentication flow
- [[file-upload-ux]] — Document uploads

For backend integration, see [[gateway-responsibilities]] and [[endpoints-contracts]].

---

## Page Structure

```
/
├── /login              # Auth page
├── /signup             # Registration (optional, can be on login)
├── /app
│   ├── /chat           # Main chat interface
│   ├── /chat/[id]      # Specific session
│   ├── /files          # Document management
│   └── /settings       # User preferences
└── /404                # Not found
```

---

## Page: Login (/login)

See [[auth-ux]] for detailed UX flow.

### Purpose
- Authenticate existing users via [[auth-and-jwt|Supabase Auth]]
- Link to signup

### Components
- Email input
- Password input
- Submit button
- "Forgot password" link
- "Sign up" link
- OAuth buttons (future: Google, GitHub)

### Behavior
- Validate input before submit
- Show loading state
- Display errors inline
- Redirect to /app/chat on success

---

## Page: Main Chat (/app/chat)

### Layout
```
┌────────────────────────────────────────────────────────────┐
│ Header                                              [User] │
├──────────────┬─────────────────────────────────────────────┤
│              │                                             │
│   Sessions   │              Chat Area                      │
│   Sidebar    │                                             │
│              │  ┌─────────────────────────────────────┐   │
│   - Session1 │  │ Message bubbles                     │   │
│   - Session2 │  │                                     │   │
│   - Session3 │  │ user: ...                           │   │
│   + New Chat │  │ assistant: ...                      │   │
│              │  │                                     │   │
│              │  └─────────────────────────────────────┘   │
│              │                                             │
│              │  ┌─────────────────────────────────────┐   │
│              │  │ Input area              [Think] [▶] │   │
│              │  └─────────────────────────────────────┘   │
└──────────────┴─────────────────────────────────────────────┘
```

### Components

**Header**
- Logo/app name
- User avatar/menu
- Settings link

**Sessions Sidebar**
- List of chat sessions
- New chat button
- Search sessions (future)
- Archive filter

**Chat Area**
- Message list (scrollable)
- Auto-scroll to bottom
- Load more on scroll up

**Input Area**
- Text input (multiline)
- Deep Think toggle
- Send button
- File attach button

---

## Component: Message Bubble

### Variants
| Variant | Style |
|---------|-------|
| User message | Right-aligned, colored background |
| Assistant message | Left-aligned, neutral background |
| System message | Center, muted |

### Content
- Message text (markdown rendered)
- Timestamp
- Mode indicator (if thinking)
- Citations (expandable)
- Tool calls (expandable)

### Actions
- Copy message
- Regenerate (future)
- Edit (future)

---

## Component: Deep Think Toggle

### States
| State | Display |
|-------|---------|
| Off (default) | Simple toggle, muted |
| On | Toggle highlighted, brain icon |
| Thinking | Animated spinner, "Thinking..." |

### Behavior
- Click to toggle
- Persists for session
- Resets on new session

---

## Component: Session Item

### Display
- Session title
- Last message preview
- Updated timestamp
- Unread indicator (future)

### Actions
- Click to select
- Right-click menu: rename, archive, delete

---

## Page: Files (/app/files)

### Layout
```
┌────────────────────────────────────────────────────────────┐
│ Header                                              [User] │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  [Upload File]                              [Search: ___]  │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ File List                                            │ │
│  │                                                      │ │
│  │ 📄 report.pdf         1.2 MB    Completed    [...]  │ │
│  │ 📄 notes.md           24 KB     Completed    [...]  │ │
│  │ 📄 data.txt           156 KB    Processing   [...]  │ │
│  │                                                      │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Components

**Upload Button**
- Drag and drop zone
- File picker
- Upload progress

**File List**
- Filename
- Size
- Status (pending, processing, completed, failed)
- Actions menu (download, delete)

---

## Page: Settings (/app/settings)

### Sections

**Profile**
- Display name
- Email (read-only)
- Avatar

**Preferences**
- Default mode (instant/think)
- Theme (light/dark)
- Notification settings

**Usage (Future)**
- Requests this month
- Storage used
- Upgrade plan

**Security**
- Change password
- Active sessions
- Delete account

---

## Responsive Design

### Breakpoints
| Size | Behavior |
|------|----------|
| Desktop (>1024px) | Full layout, sidebar visible |
| Tablet (768-1024px) | Collapsible sidebar |
| Mobile (<768px) | Full-screen chat, slide-out sidebar |

### Mobile Adjustments
- Hamburger menu for sidebar
- Full-width message input
- Smaller message bubbles
- Touch-friendly buttons

---

## State Management

### Global State
- User authentication status
- Current session ID
- Sessions list
- Theme preference

### Local State
- Message input text
- Deep think toggle
- Scroll position
- Modal visibility

### Server State
- Sessions data
- Messages data
- Files data

### Tools
- React Context for auth
- SWR or React Query for server state
- Local state in components

---

## Loading States

| Component | Loading Display |
|-----------|-----------------|
| Sessions list | Skeleton loaders |
| Messages | Skeleton bubbles |
| File upload | Progress bar |
| Send message | Button disabled, spinner |
| Thinking mode | Animated indicator |

---

## Error States

| Error | Display |
|-------|---------|
| Auth failed | Redirect to login |
| Network error | Toast notification, retry button |
| Rate limited | Toast with retry time |
| File too large | Inline error message |
| Processing failed | Status badge, retry option |

---

## Checklist: UI Implementation

- [ ] Login page functional
- [ ] Main chat layout built
- [ ] Sessions sidebar working
- [ ] Message list rendering
- [ ] Message input working
- [ ] Deep Think toggle functional
- [ ] Send message flow complete
- [ ] Files page basic version
- [ ] Upload working
- [ ] Settings page basic version
- [ ] Responsive design tested
- [ ] Loading states implemented
- [ ] Error handling complete


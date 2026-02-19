---
tags:
  - webapp
cssclasses:
  - webapp-doc
---

# Auth UX

## Overview

Authentication user experience design for secure, frictionless access.

For backend implementation, see [[auth-and-jwt]].
For security context, see [[threat-model]] and [[secrets-management]].

---

## Auth Flows

### Login Flow
```
┌─────────────────────────────────────────┐
│           Login Page                    │
│                                         │
│   Email: [________________]             │
│   Password: [______________]            │
│                                         │
│   [Forgot Password?]                    │
│                                         │
│   [    Login    ]                       │
│                                         │
│   ─────────── or ───────────           │
│                                         │
│   [Continue with Google] (future)       │
│                                         │
│   Don't have an account? [Sign up]      │
└─────────────────────────────────────────┘
```

### Signup Flow
```
┌─────────────────────────────────────────┐
│           Create Account                │
│                                         │
│   Email: [________________]             │
│   Password: [______________]            │
│   Confirm: [_______________]            │
│                                         │
│   [    Create Account    ]              │
│                                         │
│   Already have an account? [Login]      │
└─────────────────────────────────────────┘
```

---

## Password Requirements

### Rules
- Minimum 8 characters
- At least one number
- At least one special character
- No common passwords

### Visual Feedback
```
Password Strength:
[████████░░░░░░░░] Medium

✓ 8+ characters
✓ Contains number
✗ Special character needed
```

---

## Email Verification

### Flow (if enabled)
1. User signs up
2. Verification email sent
3. User clicks link
4. Account activated
5. Redirect to app

### UI States
- "Check your email" screen
- Resend verification button
- Error if link expired

---

## Password Reset

### Flow
```
┌─────────────────────────────────────────┐
│        Forgot Password                  │
│                                         │
│   Enter your email to receive a         │
│   password reset link.                  │
│                                         │
│   Email: [________________]             │
│                                         │
│   [    Send Reset Link    ]             │
│                                         │
│   [Back to Login]                       │
└─────────────────────────────────────────┘
```

### States
- Success: "Check your email"
- Not found: Generic message (don't reveal)
- Rate limited: "Try again in X minutes"

---

## Session Management

### Token Handling
- Access token: short-lived (1 hour)
- Refresh token: longer-lived (1 week)
- Silent refresh before expiry

### Session Persistence
- Remember me checkbox
- Without: session ends on browser close
- With: session persists until logout

---

## Logout

### Behavior
- Clear all tokens
- Redirect to login
- Show confirmation toast

### Where to Trigger
- User menu dropdown
- Settings page
- Session expired automatically

---

## Error Messages

### Principles
- Clear and helpful
- Don't reveal security info
- Suggest next steps

### Examples
| Error | Message |
|-------|---------|
| Wrong password | "Invalid email or password. Please try again." |
| Account not found | "Invalid email or password. Please try again." |
| Email taken | "An account with this email already exists." |
| Weak password | "Password must be at least 8 characters with a number and special character." |
| Rate limited | "Too many attempts. Please try again in 5 minutes." |

---

## OAuth (Future)

### Supported Providers
- Google
- GitHub
- Apple (mobile)

### Flow
1. User clicks "Continue with Google"
2. Redirect to Google OAuth
3. User authorizes
4. Callback to app
5. Create/link account
6. Redirect to app

### Account Linking
- Same email links automatically
- Different email creates new account
- Option to merge accounts

---

## Security Considerations

### Login Protection
- Rate limit login attempts
- Detect suspicious patterns
- CAPTCHA after failures (future)

### Session Security
- httpOnly cookies
- Secure flag in production
- SameSite attribute

### Logout Security
- Clear all client tokens
- Optionally revoke server-side
- Clear all sessions option

---

## Loading States

### Login Button
```
Normal: [    Login    ]
Loading: [  ◌  Logging in...  ]
```

### Page Load
- Show skeleton of login form
- Auto-focus email field
- Check existing session first

---

## Mobile Considerations

### Touch Targets
- Minimum 44x44px buttons
- Adequate spacing
- Easy thumb reach

### Keyboard
- Email keyboard for email field
- Show/hide password toggle
- Next/Done buttons

### Biometrics (Future)
- Face ID / Touch ID support
- Fall back to password
- Opt-in setting

---

## Checklist: Auth UX

- [ ] Login form functional
- [ ] Signup form functional
- [ ] Password validation with feedback
- [ ] Password reset flow
- [ ] Error messages helpful
- [ ] Loading states smooth
- [ ] Session persistence working
- [ ] Logout clears all data
- [ ] Mobile optimized
- [ ] Accessibility compliant
- [ ] Rate limiting in place


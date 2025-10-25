# Authentication Flow

## JWT Token Flow Across All Services

```
┌──────────────────────────────────────────────────────────────┐
│                    JWT Authentication Flow                     │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Initial Authentication                                    │
│  ─────────────────────────                                   │
│  User → Login Form → Resume-SSO                              │
│           │                │                                  │
│           │                ├→ Validate credentials (D1)       │
│           │                ├→ Generate JWT tokens             │
│           │                ├→ Store refresh token (KV)       │
│           │                └→ Return tokens to client        │
│           │                                                   │
│  2. Token Structure                                          │
│  ─────────────────────                                       │
│  Access Token (15 min):                                      │
│  {                                                           │
│    "sub": "user_id",                                         │
│    "email": "user@email.com",                              │
│    "role": "user|admin",                                     │
│    "permissions": [],                                        │
│    "iat": 1234567890,                                       │
│    "exp": 1234568790                                        │
│  }                                                          │
│                                                              │
│  Refresh Token (30 days):                                   │
│  {                                                          │
│    "sub": "user_id",                                        │
│    "token_id": "unique_id",                                │
│    "iat": 1234567890,                                      │
│    "exp": 1237159890                                       │
│  }                                                          │
│                                                              │
│  3. Token Validation Flow                                   │
│  ──────────────────────────                                │
│  Client Request → Service Middleware → Resume-SSO           │
│                        │                    │               │
│                        ├→ Extract token     │               │
│                        ├→ Verify signature  │               │
│                        ├→ Check expiration  │               │
│                        ├→ Validate permissions              │
│                        └→ Inject user context              │
│                                                             │
│  4. Token Refresh Flow                                     │
│  ──────────────────────                                   │
│  Client → Resume-SSO/refresh                              │
│           │                                                │
│           ├→ Validate refresh token (KV)                  │
│           ├→ Check token rotation                         │
│           ├→ Generate new access token                    │
│           ├→ Rotate refresh token (optional)              │
│           └→ Return new tokens                            │
│                                                            │
│  5. Cross-Service Authentication                          │
│  ─────────────────────────────────                       │
│  Service A → Service B                                    │
│      │          │                                         │
│      ├→ Include JWT in Authorization header              │
│      │          │                                        │
│      │          ├→ Validate token locally               │
│      │          ├→ Check service permissions            │
│      │          └→ Process request                      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## OAuth Flow Integration

```
User → Resume-UI → Resume-SSO → OAuth Provider (Google/GitHub/etc)
                        │                │
                        │                └→ Callback with user data
                        ├→ Create/update user in D1
                        ├→ Generate JWT tokens
                        └→ Return tokens to Resume-UI
```

# Security Architecture

## Defense in Depth

```
┌─────────────────────────────────────────────────────────────┐
│                   Security Architecture                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Layer 1: Network Security                                   │
│ ─────────────────────────                                  │
│ - Cloudflare DDoS Protection                               │
│ - Web Application Firewall (WAF)                           │
│ - Rate Limiting per IP/User                                │
│ - Bot Management                                           │
│                                                             │
│ Layer 2: Authentication & Authorization                    │
│ ────────────────────────────────────────                  │
│ - JWT with RSA-256 signing                                 │
│ - Refresh token rotation                                   │
│ - Role-Based Access Control (RBAC)                         │
│ - Multi-Factor Authentication (MFA)                        │
│                                                             │
│ Layer 3: Application Security                              │
│ ─────────────────────────────                             │
│ - Input validation and sanitization                        │
│ - SQL injection prevention (parameterized queries)         │
│ - XSS protection (Content Security Policy)                 │
│ - CSRF tokens for state-changing operations               │
│                                                             │
│ Layer 4: Data Security                                     │
│ ──────────────────────                                    │
│ - Encryption at rest (D1, R2)                              │
│ - Encryption in transit (TLS 1.3)                          │
│ - PII data masking in logs                                │
│ - Secure key storage (Cloudflare Secrets)                  │
│                                                             │
│ Layer 5: Compliance & Auditing                            │
│ ───────────────────────────────                           │
│ - GDPR compliance (data portability, right to delete)      │
│ - Audit logging for all admin actions                      │
│ - Security headers (HSTS, CSP, X-Frame-Options)           │
│ - Regular security assessments                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

# API Contracts

## Service API Endpoints

```
┌────────────────────────────────────────────────────────────────┐
│                        API Contract Map                         │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│ Resume-SSO API (auth.resume-builder.com)                      │
│ ├── POST   /auth/register                                     │
│ ├── POST   /auth/login                                        │
│ ├── POST   /auth/logout                                       │
│ ├── POST   /auth/refresh                                      │
│ ├── POST   /auth/verify-email                                │
│ ├── POST   /auth/reset-password                              │
│ ├── GET    /auth/validate                                    │
│ ├── POST   /oauth/google                                     │
│ ├── POST   /oauth/github                                     │
│ └── GET    /oauth/callback                                   │
│                                                               │
│ Resume-API (api.resume-builder.com)                         │
│ ├── GET    /resumes                      [Auth Required]     │
│ ├── POST   /resumes                      [Auth Required]     │
│ ├── GET    /resumes/:id                  [Auth Required]     │
│ ├── PUT    /resumes/:id                  [Auth Required]     │
│ ├── DELETE /resumes/:id                  [Auth Required]     │
│ ├── POST   /resumes/:id/clone            [Auth Required]     │
│ ├── POST   /resumes/:id/export           [Auth Required]     │
│ ├── GET    /resumes/:id/export/:jobId    [Auth Required]     │
│ ├── POST   /resumes/:id/collaborate      [Auth Required]     │
│ ├── WS     /resumes/:id/live             [Auth Required]     │
│ ├── GET    /templates                                        │
│ ├── GET    /templates/:id                                   │
│ ├── POST   /ai/enhance                   [Auth Required]     │
│ └── POST   /ai/suggest                   [Auth Required]     │
│                                                               │
│ Resume-Account-API (account.resume-builder.com)             │
│ ├── GET    /profile                      [Auth Required]     │
│ ├── PUT    /profile                      [Auth Required]     │
│ ├── POST   /profile/avatar               [Auth Required]     │
│ ├── GET    /subscription                 [Auth Required]     │
│ ├── POST   /subscription/upgrade         [Auth Required]     │
│ ├── POST   /subscription/cancel          [Auth Required]     │
│ ├── GET    /billing/history              [Auth Required]     │
│ ├── GET    /billing/invoices/:id         [Auth Required]     │
│ ├── POST   /payment-methods              [Auth Required]     │
│ ├── DELETE /payment-methods/:id          [Auth Required]     │
│ ├── GET    /usage                        [Auth Required]     │
│ └── PUT    /settings                     [Auth Required]     │
│                                                               │
│ Resume-Admin-API (admin.resume-builder.com)                 │
│ ├── GET    /users                        [Admin Required]    │
│ ├── GET    /users/:id                    [Admin Required]    │
│ ├── PUT    /users/:id                    [Admin Required]    │
│ ├── DELETE /users/:id                    [Admin Required]    │
│ ├── GET    /resumes                      [Admin Required]    │
│ ├── DELETE /resumes/:id                  [Admin Required]    │
│ ├── GET    /analytics/overview           [Admin Required]    │
│ ├── GET    /analytics/users              [Admin Required]    │
│ ├── GET    /analytics/revenue            [Admin Required]    │
│ ├── GET    /audit-logs                   [Admin Required]    │
│ ├── GET    /system/health                [Admin Required]    │
│ ├── PUT    /system/config                [Admin Required]    │
│ └── POST   /support/tickets              [Admin Required]    │
│                                                               │
└────────────────────────────────────────────────────────────────┘
```

## API Response Formats

```typescript
// Standard Success Response
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "version": "1.0.0"
  }
}

// Standard Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": { ... }
  },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z",
    "request_id": "req_123456"
  }
}

// Paginated Response
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  },
  "meta": { ... }
}
```

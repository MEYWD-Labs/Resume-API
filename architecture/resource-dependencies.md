# Resource Dependencies

## Complete Cloudflare Resource Mapping

```
┌────────────────────────────────────────────────────────────────────┐
│                    Cloudflare Resource Dependencies                 │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│ Resume-SSO                                                         │
│ ├── D1: users, auth_sessions, oauth_states                        │
│ ├── KV: refresh_tokens, rate_limits, blacklisted_tokens          │
│ ├── Workers: Authentication endpoints                             │
│ └── Email Workers: Verification, password reset                   │
│                                                                    │
│ Resume-API                                                        │
│ ├── D1: resumes, templates, sections, experiences                │
│ ├── R2: resume_images/, user_uploads/                           │
│ ├── KV: resume_cache, api_cache                                 │
│ ├── Durable Objects: ResumeCollaboration, ResumeState          │
│ ├── Queues: pdf_generation, ai_enhancement                     │
│ └── Workers: Core API endpoints                                │
│                                                                 │
│ Resume-Account-API                                            │
│ ├── D1: user_profiles, subscriptions, billing, invoices       │
│ ├── KV: subscription_cache, payment_sessions                  │
│ ├── R2: profile_images/, invoices/                           │
│ └── Workers: Account management endpoints                     │
│                                                                │
│ Resume-Admin-API                                             │
│ ├── D1: admin_users, audit_logs, system_settings            │
│ ├── KV: admin_sessions, analytics_cache                     │
│ ├── Workers Analytics: Usage metrics, performance data      │
│ └── Workers: Admin endpoints                                │
│                                                              │
│ Resume-Processor                                            │
│ ├── D1: job_queue, job_status                              │
│ ├── R2: generated_pdfs/, export_files/                    │
│ ├── Queues: Consumer for all job types                    │
│ ├── Workers AI: Content generation, optimization          │
│ └── Email Workers: Job completion notifications            │
│                                                             │
│ Resume-UI (Pages)                                          │
│ ├── Pages: Static site hosting                            │
│ ├── KV: page_cache, user_preferences (edge-side)         │
│ └── Functions: Edge-side rendering                        │
│                                                            │
│ Resume-Account-UI (Pages)                                 │
│ ├── Pages: Static site hosting                           │
│ └── Functions: Edge-side rendering                       │
│                                                           │
│ Resume-Admin (Pages)                                     │
│ ├── Pages: Static site hosting                          │
│ └── Functions: Edge-side rendering                      │
│                                                          │
│ Resume-Mobile-UI                                        │
│ └── API consumption only (no direct Cloudflare resources) │
│                                                           │
│ Resume-Mobile-Admin                                      │
│ └── API consumption only (no direct Cloudflare resources) │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## Resource Usage Patterns

```
**D1 Database Tables Structure:**

| Database | Tables |
|----------|--------|
| resume_auth_db | users, sessions, oauth_states, password_resets, login_attempts |
| resume_core_db | resumes, templates, sections, experiences, education, skills, projects, certifications |
| resume_account_db | user_profiles, subscriptions, billing_info, invoices, usage, payment_methods, plans |
| resume_admin_db | admin_users, roles, permissions, audit_logs, system_config, analytics_snapshots |

**KV Namespaces:**

| Namespace | Key Pattern | TTL |
|-----------|-------------|-----|
| auth_tokens | refresh:{user_id}, blacklist:{token} | 30d, 24h |
| api_cache | resume:{id}:v{ver}, template:{id} | 1h, 24h |
| rate_limits | ip:{address}, user:{id} | 1h, 1h |

**R2 Buckets:**

| Bucket | Path Structure |
|--------|---------------|
| resume-assets | /users/{user_id}/images/, /users/{user_id}/documents/ |
| resume-exports | /pdfs/{job_id}/, /exports/{format}/{job_id}/ |
| resume-static | /templates/, /assets/ |
```

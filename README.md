# Resume-API

**Core Backend Service** for the Resume Builder SaaS platform.

## Overview

Resume-API is the primary backend service that handles all resume management, payment processing, AI features, website generation, and collaboration functionality. Built on Cloudflare Workers with D1 database, R2 storage, and Workers AI.

## Technology Stack

- **Runtime**: Cloudflare Workers (serverless)
- **Database**: Cloudflare D1 (SQLite)
- **Storage**: Cloudflare R2 (PDF files, images)
- **Cache**: Cloudflare KV (rate limiting, feature flags)
- **Queue**: Cloudflare Queues (PDF generation, email sending)
- **AI**: Cloudflare Workers AI (content suggestions, parsing)
- **Analytics**: Cloudflare Analytics Engine
- **Framework**: Hono (lightweight web framework)
- **Payment**: Stripe API
- **Language**: TypeScript

## Key Features

### MVP-1: Core Resume API
- Resume CRUD operations with auto-save
- Section management (education, experience, skills, etc.)
- Template selection and management
- Basic PDF export with queue processing
- Multi-resume support

### MVP-2: Payment & Feature Gating
- Stripe subscription integration
- Webhook handling for payment events
- Feature gating based on subscription tier
- Usage tracking (resume count, export count)
- Subscription management (upgrade, downgrade, cancel)

### MVP-3: Import & Advanced Export
- PDF import with AI parsing
- DOCX import
- LinkedIn profile import via OAuth
- Advanced PDF export (custom fonts, watermarks)
- Batch export functionality

### MVP-4: Website Generation
- Static website generation from resume
- Deploy to Cloudflare Pages
- Custom domain support
- Theme customization
- SEO optimization

### MVP-5: AI Features
- AI content suggestions (powered by Cloudflare Workers AI)
- Resume analysis and ATS scoring
- Grammar and style improvements
- Keyword optimization
- Tailored resume generation for job descriptions

### MVP-6: Collaboration & Analytics
- Real-time collaboration (Cloudflare Durable Objects)
- Share resumes with view/edit permissions
- Version history and rollback
- Analytics tracking (views, exports, shares)
- User activity logging

## Service Dependencies

**Depends On**:
- **Resume-SSO** (MVP-1.4: Login with JWT) - For user authentication and authorization

**Services that depend on Resume-API**:
- Resume-UI (all user-facing features)
- Resume-Admin-API (shared database access)

## API Endpoints

### Authentication (Protected)
All endpoints require JWT token from Resume-SSO in `Authorization: Bearer <token>` header.

### Resume Management
- `GET /api/resumes` - List user's resumes
- `POST /api/resumes` - Create new resume
- `GET /api/resumes/:id` - Get resume details
- `PUT /api/resumes/:id` - Update resume (auto-save)
- `DELETE /api/resumes/:id` - Delete resume
- `POST /api/resumes/:id/duplicate` - Duplicate resume

### Resume Sections
- `GET /api/resumes/:id/sections/:type` - Get section content
- `PUT /api/resumes/:id/sections/:type` - Update section
- `POST /api/resumes/:id/sections/:type/items` - Add section item
- `PUT /api/resumes/:id/sections/:type/items/:itemId` - Update item
- `DELETE /api/resumes/:id/sections/:type/items/:itemId` - Delete item

### Templates
- `GET /api/templates` - List available templates
- `GET /api/templates/:id` - Get template details
- `POST /api/resumes/:id/template` - Apply template

### Export & Import
- `POST /api/resumes/:id/export/pdf` - Queue PDF generation
- `GET /api/resumes/:id/export/pdf/:jobId` - Check export status
- `POST /api/resumes/import/pdf` - Import from PDF
- `POST /api/resumes/import/docx` - Import from DOCX
- `POST /api/resumes/import/linkedin` - Import from LinkedIn

### Payments (Stripe)
- `POST /api/payments/checkout` - Create checkout session
- `POST /api/payments/portal` - Create customer portal session
- `GET /api/payments/subscription` - Get subscription status
- `POST /api/webhooks/stripe` - Handle Stripe webhooks

### Website Generation
- `POST /api/websites/:resumeId/generate` - Generate static website
- `GET /api/websites/:resumeId` - Get website details
- `PUT /api/websites/:resumeId/theme` - Update website theme
- `POST /api/websites/:resumeId/domain` - Set custom domain

### AI Features
- `POST /api/ai/suggest` - Get content suggestions
- `POST /api/ai/analyze` - Analyze resume (ATS scoring)
- `POST /api/ai/improve` - Improve grammar and style
- `POST /api/ai/tailor` - Tailor resume for job description

### Collaboration
- `POST /api/resumes/:id/share` - Share resume
- `GET /api/resumes/:id/collaborators` - List collaborators
- `DELETE /api/resumes/:id/share/:userId` - Remove collaborator
- `GET /api/resumes/:id/versions` - Get version history
- `POST /api/resumes/:id/versions/:versionId/restore` - Restore version

## Development Setup

### Prerequisites
- Node.js 18+
- Wrangler CLI (`npm install -g wrangler`)
- Cloudflare account
- Stripe account

### Environment Variables

Create a `.dev.vars` file:

```bash
# Database
DATABASE_URL=<d1-database-url>

# Cloudflare
R2_BUCKET=resume-files
KV_NAMESPACE=resume-cache
QUEUE_NAME=resume-jobs

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID_PRO=price_...
STRIPE_PRICE_ID_PREMIUM=price_...

# Workers AI
AI_MODEL_SUGGESTION=@cf/meta/llama-2-7b-chat-int8
AI_MODEL_ANALYSIS=@cf/mistral/mistral-7b-instruct-v0.1

# Feature Flags
ENABLE_AI_FEATURES=true
ENABLE_COLLABORATION=true
```

### Quick Start

```bash
# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Seed sample data
npm run db:seed

# Start development server
npm run dev

# Run tests
npm test

# Deploy to production
npm run deploy
```

## Database Schema

See [HIGH_LEVEL_PLAN.md](./HIGH_LEVEL_PLAN.md) for complete schema definitions.

Key tables:
- `resumes` - Resume metadata
- `resume_sections` - Resume content sections
- `templates` - Resume templates
- `subscriptions` - User subscription data
- `analytics_events` - User activity tracking
- `shares` - Resume sharing permissions
- `versions` - Resume version history

## Subscription Tiers

| Feature | Free | Pro ($9.99/mo) | Premium ($19.99/mo) |
|---------|------|----------------|---------------------|
| Resumes | 1 | Unlimited | Unlimited |
| PDF Exports | 5/month | Unlimited | Unlimited |
| Templates | 10 | 50+ | 100+ |
| AI Suggestions | No | Yes | Yes |
| Custom Domains | No | No | Yes |
| Collaboration | No | No | Yes |
| Advanced Export | No | Yes | Yes |

## Queue Processing

The service uses Cloudflare Queues for background jobs:

- **PDF Generation**: Queued to avoid blocking API requests
- **Email Sending**: Verification, notifications
- **Analytics Processing**: Event aggregation
- **AI Processing**: Content analysis and suggestions

## Performance Targets

- API response time: **< 300ms** (95th percentile)
- PDF generation: **< 10s**
- Auto-save: **< 200ms**
- AI suggestions: **< 5s**
- Database queries: **< 100ms**

## Testing

```bash
# Run all tests
npm test

# Run unit tests
npm run test:unit

# Run integration tests
npm run test:integration

# Run E2E tests
npm run test:e2e

# Test coverage
npm run test:coverage
```

## Development Plan

See [HIGH_LEVEL_PLAN.md](./HIGH_LEVEL_PLAN.md) for the complete dependency-tree based development plan with MVPs, epics, and critical path.

## GitHub Issues

Track development progress in [GitHub Issues](https://github.com/MEYWD-Labs/Resume-API/issues):
- [MVP-1 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-1) - Core Resume API
- [MVP-2 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-2) - Payment & Feature Gating
- [MVP-3 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-3) - Import & Advanced Export
- [MVP-4 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-4) - Website Generation
- [MVP-5 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-5) - AI Features
- [MVP-6 Epics](https://github.com/MEYWD-Labs/Resume-API/issues?q=is%3Aissue+is%3Aopen+MVP-6) - Collaboration & Analytics

## Deployment

```bash
# Deploy to production
wrangler deploy

# Deploy with migrations
npm run deploy:migrate

# View logs
wrangler tail

# Monitor analytics
wrangler analytics
```

## Monitoring & Alerts

- Cloudflare Analytics Engine for metrics
- Error tracking via Workers Analytics
- Performance monitoring via Grafana/Prometheus
- Stripe webhook monitoring

## Support

For issues or questions, create a GitHub issue in this repository.

## License

Proprietary - All rights reserved
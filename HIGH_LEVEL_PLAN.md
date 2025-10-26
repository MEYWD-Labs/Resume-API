# High-Level Development Plan - Resume-API

**Service Type**: Core Backend API (Cloudflare Workers)
**Build Approach**: AI-Automated Development
**Priority**: HIGH - Required for all frontend features

---

## Dependency Tree Overview

```
Resume-API (Core Backend Service)
├── DEPENDS ON:
│   └── Resume-SSO (authentication/JWT validation)
│
├── BLOCKS:
│   ├── Resume-UI (requires API for all resume operations)
│   ├── Resume-Mobile-UI (requires API for mobile resume operations)
│   └── Resume-Processor (provides data for PDF generation, AI processing)

Services that DEPEND on Resume-API:
├── Resume-UI (MVP-1: Resume CRUD, templates, exports)
├── Resume-Mobile-UI (MVP-1: Mobile resume operations)
├── Resume-Processor (MVP-1.5: PDF generation, AI features)
└── Resume-Admin (indirect: for monitoring resume metrics)

MVP Breakdown:
├── MVP-1: Core Resume API ⟶ Blocks: Resume-UI, Resume-Mobile-UI basic features
│   ├── Database Schema & Setup
│   ├── Authentication Middleware (uses SSO JWT)
│   ├── Resume CRUD Operations
│   ├── Template Management System
│   └── Basic PDF Export
│
├── MVP-2: Payment & Feature Gating ⟶ Blocks: Premium features in Resume-UI, Resume-Mobile-UI
│   ├── Stripe Integration
│   ├── Subscription Status Tracking
│   └── Feature Gating Middleware
│
├── MVP-3: Import & Advanced Export ⟶ Blocks: Import features in Resume-UI, Resume-Mobile-UI
│   ├── PDF Import & Parsing
│   ├── DOCX Import
│   ├── LinkedIn Import (via OAuth)
│   └── Multi-format Export
│
├── MVP-4: Website Generation ⟶ Blocks: Personal website features in Resume-UI
│   ├── Static Site Generation
│   ├── Cloudflare Pages Deployment
│   └── Custom Domain Management
│
├── MVP-5: AI Features ⟶ Blocks: AI features in Resume-UI, Resume-Processor AI jobs
│   ├── Content Suggestions (Cloudflare AI)
│   ├── Resume Analysis & ATS Scoring
│   └── Smart Rewriting
│
└── MVP-6: Collaboration & Analytics ⟶ Blocks: Advanced features in Resume-UI, Resume-Admin
    ├── Real-time Collaboration (Durable Objects)
    ├── Sharing System
    └── Usage Analytics
```

---

## MVP-1: Core Resume API (Foundation)

**Status**: MUST BUILD AFTER Resume-SSO JWT validation is ready
**Dependencies**: Resume-SSO (MVP-2.1: JWT Validation)
**Blocks**: Resume-UI, Resume-Mobile-UI, Resume-Processor (for PDF/AI jobs)

### Epic 1.1: Database Schema & Setup
**What**: D1 database structure for resume management
**Complexity**: Medium
**Dependencies**: Resume-SSO database exists

**Tasks**:
- [ ] Create D1 database schema (users, resumes, templates)
- [ ] Set up wrangler.toml with D1 bindings
- [ ] Create migration scripts
- [ ] Seed 5 base templates (Modern, Classic, ATS-Friendly, Minimalist, Professional)
- [ ] Store template preview images in R2

**Acceptance Criteria**:
- [ ] D1 database created and accessible
- [ ] Schema matches feature plan specifications
- [ ] 5 base templates seeded and accessible
- [ ] R2 bucket configured for template previews

---

### Epic 1.2: Authentication Middleware
**What**: Validate JWT tokens from Resume-SSO
**Complexity**: Medium
**Dependencies**: Epic 1.1, Resume-SSO (MVP-2.1: JWT Validation)

**Tasks**:
- [ ] JWT validation middleware using jose library
- [ ] Extract user context from JWT payload (user_id, email, subscription_tier)
- [ ] Permission checking middleware
- [ ] Rate limiting per user (100/hour free, 1000/hour pro)
- [ ] Error handling for invalid/expired tokens

**Acceptance Criteria**:
- [ ] JWT tokens validated correctly
- [ ] User context injected into request handlers
- [ ] Rate limits enforced per subscription tier
- [ ] Appropriate error responses for auth failures

---

### Epic 1.3: Resume CRUD Operations
**What**: Core resume management endpoints
**Complexity**: Large
**Dependencies**: Epic 1.2 (Authentication)

**Tasks**:
- [ ] `POST /api/resumes` - Create new resume
- [ ] `GET /api/resumes` - List user's resumes
- [ ] `GET /api/resumes/:id` - Get resume by ID
- [ ] `PUT /api/resumes/:id` - Update resume
- [ ] `DELETE /api/resumes/:id` - Delete resume
- [ ] `POST /api/resumes/:id/duplicate` - Duplicate resume
- [ ] Auto-save system with debouncing
- [ ] Optimistic updates support
- [ ] Conflict resolution logic
- [ ] Version tracking

**Acceptance Criteria**:
- [ ] All CRUD operations functional
- [ ] Users can only access their own resumes
- [ ] Auto-save works with debouncing
- [ ] Free tier limited to 2 resumes
- [ ] Proper error handling and validation

---

### Epic 1.4: Template Management
**What**: Template retrieval and management APIs
**Complexity**: Medium
**Dependencies**: Epic 1.1 (Database)

**Tasks**:
- [ ] `GET /api/templates` - List all templates
- [ ] `GET /api/templates/:id` - Get template by ID
- [ ] Template preview image serving from R2
- [ ] Filter templates by category
- [ ] Mark premium templates (requires Pro tier)
- [ ] Cache templates in Cloudflare KV

**Acceptance Criteria**:
- [ ] Templates returned with preview URLs
- [ ] Free users see only free templates
- [ ] Premium templates marked appropriately
- [ ] Templates cached for performance

---

### Epic 1.5: Basic PDF Export
**What**: Generate PDF from resume data
**Complexity**: Large
**Dependencies**: Epic 1.3 (Resume CRUD)

**Tasks**:
- [ ] `POST /api/resumes/:id/export/pdf` - Generate PDF
- [ ] HTML to PDF conversion using Puppeteer or pdf-lib
- [ ] Template styling preservation
- [ ] Store generated PDFs in R2 (temporary, 1 hour)
- [ ] Signed URL generation for download
- [ ] Cloudflare Queue for async processing
- [ ] PDF processing queue consumer
- [ ] Status tracking for generation progress
- [ ] Retry logic for failed generations

**Acceptance Criteria**:
- [ ] PDFs generated with correct styling
- [ ] Free tier: 10 exports/month limit enforced
- [ ] Queue processing handles load
- [ ] Generated PDFs auto-expire after 1 hour
- [ ] Status endpoint shows generation progress

---

## MVP-2: Payment & Feature Gating (Premium Enablement)

**Status**: Required for Pro/Premium tier features
**Dependencies**: MVP-1
**Blocks**: All premium features in Resume-UI

### Epic 2.1: Stripe Integration
**What**: Payment processing and subscription management
**Complexity**: Large
**Dependencies**: Epic 1.2 (Authentication)

**Tasks**:
- [ ] `POST /api/billing/create-checkout` - Create Stripe checkout session
- [ ] `POST /api/billing/webhook` - Handle Stripe webhooks
- [ ] `GET /api/billing/subscription` - Get current subscription status
- [ ] `POST /api/billing/cancel` - Cancel subscription
- [ ] `POST /api/billing/update` - Update payment method
- [ ] Store subscription data in D1 (user_id, tier, status, stripe_subscription_id)
- [ ] Handle webhook events (payment_succeeded, payment_failed, subscription_updated)
- [ ] Sync subscription status to Resume-SSO user table

**Acceptance Criteria**:
- [ ] Users can upgrade to Pro/Premium
- [ ] Webhooks processed correctly
- [ ] Subscription status synced across services
- [ ] Failed payments handled gracefully
- [ ] Users can cancel subscriptions

---

### Epic 2.2: Feature Gating Middleware
**What**: Enforce subscription tier limits
**Complexity**: Medium
**Dependencies**: Epic 2.1 (Stripe Integration)

**Tasks**:
- [ ] Feature gating middleware checks user tier
- [ ] Enforce resume count limits (2 free, unlimited paid)
- [ ] Enforce export limits (10/month free, unlimited paid)
- [ ] Premium template access control
- [ ] API rate limit enforcement per tier
- [ ] Return clear error messages (403 with upgrade prompt)

**Acceptance Criteria**:
- [ ] Free users blocked from premium features
- [ ] Appropriate error codes and messages returned
- [ ] Limits reset correctly (monthly for exports)
- [ ] Pro/Premium users have full access

---

## MVP-3: Import & Advanced Export (User Experience)

**Status**: Enhances user onboarding and workflows
**Dependencies**: MVP-2
**Blocks**: Import features in Resume-UI

### Epic 3.1: PDF Import & Parsing
**What**: Extract resume data from PDF files
**Complexity**: Large
**Dependencies**: Epic 2.2 (Feature Gating - Pro tier feature)

**Tasks**:
- [ ] `POST /api/import/pdf` - Upload and parse PDF
- [ ] File upload to R2
- [ ] Extract text content using pdf-parse or similar
- [ ] Structure detection (identify sections: experience, education, skills)
- [ ] Use Cloudflare AI for intelligent parsing
- [ ] Map extracted data to resume JSON structure
- [ ] Return structured resume data

**Acceptance Criteria**:
- [ ] PDFs uploaded to R2 successfully
- [ ] Text extracted accurately
- [ ] Sections identified correctly (>80% accuracy)
- [ ] Structured data returned in resume format
- [ ] Only available to Pro/Premium users

---

### Epic 3.2: DOCX Import
**What**: Parse Word documents and extract resume data
**Complexity**: Medium
**Dependencies**: Epic 3.1 (PDF Import pattern established)

**Tasks**:
- [ ] `POST /api/import/docx` - Upload and parse Word doc
- [ ] Use mammoth.js for DOCX parsing
- [ ] Preserve formatting where possible
- [ ] Extract structured data
- [ ] Section detection logic
- [ ] Map to resume JSON structure

**Acceptance Criteria**:
- [ ] DOCX files parsed successfully
- [ ] Formatting preserved
- [ ] Data structured correctly
- [ ] Only available to Pro/Premium users

---

### Epic 3.3: LinkedIn Import
**What**: Import resume data from LinkedIn profile
**Complexity**: Large
**Dependencies**: Epic 3.2, Resume-SSO (MVP-3.2: LinkedIn OAuth)

**Tasks**:
- [ ] `POST /api/import/linkedin` - Import from LinkedIn
- [ ] Integration with Resume-SSO LinkedIn OAuth
- [ ] Fetch LinkedIn profile data via API
- [ ] Map LinkedIn data to resume structure
- [ ] Extract: name, headline, summary, experience, education, skills
- [ ] Handle profile photos (store in R2)

**Acceptance Criteria**:
- [ ] LinkedIn data imported successfully
- [ ] Profile data mapped accurately
- [ ] Experience and education parsed correctly
- [ ] Only available to Pro/Premium users

---

### Epic 3.4: Multi-format Export
**What**: Export resumes in multiple formats
**Complexity**: Medium
**Dependencies**: Epic 1.5 (PDF Export)

**Tasks**:
- [ ] `POST /api/resumes/:id/export/docx` - Generate Word document
- [ ] Use docx library for DOCX generation
- [ ] `POST /api/resumes/:id/export/batch` - Multi-format export
- [ ] Queue-based processing for batch exports
- [ ] ZIP file creation with all formats (PDF, DOCX, HTML, JSON)
- [ ] Signed URL for ZIP download

**Acceptance Criteria**:
- [ ] DOCX exports generate correctly
- [ ] Batch export produces ZIP with all formats
- [ ] Templates applied correctly to all formats
- [ ] Only available to Pro/Premium users

---

## MVP-4: Website Generation (Premium Feature)

**Status**: Premium user value-add
**Dependencies**: MVP-3
**Blocks**: Personal website features in Resume-UI

### Epic 4.1: Static Site Generation
**What**: Generate personal website from resume
**Complexity**: Large
**Dependencies**: Epic 1.3 (Resume CRUD)

**Tasks**:
- [ ] `POST /api/websites/generate` - Generate website from resume
- [ ] Convert resume JSON to HTML/CSS/JS
- [ ] Use website templates (5 designs)
- [ ] Deploy to Cloudflare Pages via API
- [ ] Assign custom subdomain (username.resumebuilder.dev)
- [ ] Store website metadata in D1

**Acceptance Criteria**:
- [ ] Websites generated from resume data
- [ ] Deployed to Cloudflare Pages
- [ ] Custom subdomains assigned
- [ ] Pro users: 1 website, Premium users: 5 websites

---

### Epic 4.2: Website Management
**What**: Manage and update personal websites
**Complexity**: Medium
**Dependencies**: Epic 4.1 (Site Generation)

**Tasks**:
- [ ] `GET /api/websites/:id` - Get website details
- [ ] `PUT /api/websites/:id` - Update website
- [ ] `DELETE /api/websites/:id` - Delete website
- [ ] `POST /api/websites/:id/publish` - Publish changes
- [ ] Track website version history
- [ ] Rollback functionality

**Acceptance Criteria**:
- [ ] Websites can be updated
- [ ] Changes published to Cloudflare Pages
- [ ] Version history maintained
- [ ] Rollback works correctly

---

### Epic 4.3: Custom Domain Support
**What**: Allow users to use custom domains for websites
**Complexity**: Large
**Dependencies**: Epic 4.2 (Website Management)

**Tasks**:
- [ ] DNS management via Cloudflare API
- [ ] Domain verification flow
- [ ] SSL certificate provisioning (automatic via Cloudflare)
- [ ] CNAME record validation
- [ ] Domain status tracking

**Acceptance Criteria**:
- [ ] Custom domains can be connected
- [ ] DNS configuration validated
- [ ] SSL certificates issued automatically
- [ ] Only available to Premium users

---

## MVP-5: AI Features (Premium Enhancement)

**Status**: AI-powered resume enhancement
**Dependencies**: MVP-4
**Blocks**: AI features in Resume-UI

### Epic 5.1: Content Suggestions
**What**: AI-generated bullet points and content
**Complexity**: Large
**Dependencies**: Epic 1.3 (Resume CRUD)

**Tasks**:
- [ ] `POST /api/ai/suggest-bullets` - Generate bullet points
- [ ] Integration with Cloudflare AI (@cf/meta/llama-2-7b-chat-int8)
- [ ] Context-aware suggestions based on job title and industry
- [ ] Industry-specific content library
- [ ] Rate limiting (100 suggestions/month Pro, unlimited Premium)
- [ ] Cache common suggestions in KV

**Acceptance Criteria**:
- [ ] AI generates relevant bullet points
- [ ] Suggestions contextually appropriate
- [ ] Rate limits enforced per tier
- [ ] Response time < 3 seconds

---

### Epic 5.2: Resume Analysis & ATS Scoring
**What**: Analyze resume for ATS compatibility
**Complexity**: Large
**Dependencies**: Epic 5.1 (AI Integration pattern)

**Tasks**:
- [ ] `POST /api/ai/analyze` - Analyze resume
- [ ] ATS score calculation (keyword density, formatting, length)
- [ ] Keyword extraction and matching
- [ ] Section completeness check
- [ ] Improvement suggestions with priority
- [ ] Industry benchmark comparison

**Acceptance Criteria**:
- [ ] ATS score calculated accurately
- [ ] Keywords extracted correctly
- [ ] Actionable improvement suggestions provided
- [ ] Only available to Pro/Premium users

---

### Epic 5.3: Smart Rewriting
**What**: AI-powered content rewriting
**Complexity**: Medium
**Dependencies**: Epic 5.1 (AI Integration pattern)

**Tasks**:
- [ ] `POST /api/ai/rewrite` - Rewrite content
- [ ] Tone adjustment (professional, casual, formal)
- [ ] Grammar and spelling correction
- [ ] Length optimization (shorten/expand)
- [ ] Maintain key information

**Acceptance Criteria**:
- [ ] Content rewritten maintaining meaning
- [ ] Tone adjustment works correctly
- [ ] Grammar errors fixed
- [ ] Only available to Premium users

---

## MVP-6: Collaboration & Analytics (Advanced Features)

**Status**: Advanced user features
**Dependencies**: MVP-5
**Blocks**: Collaboration features in Resume-UI

### Epic 6.1: Sharing System
**What**: Share resumes with others
**Complexity**: Medium
**Dependencies**: Epic 1.3 (Resume CRUD)

**Tasks**:
- [ ] `POST /api/resumes/:id/share` - Create share link
- [ ] `GET /api/shared/:token` - Access shared resume
- [ ] Permission levels (view, comment, edit)
- [ ] Expiring links (configurable duration)
- [ ] Share token generation and validation
- [ ] Track share access (analytics)

**Acceptance Criteria**:
- [ ] Share links generated with tokens
- [ ] Permission levels enforced
- [ ] Links expire as configured
- [ ] Access tracked for analytics

---

### Epic 6.2: Real-time Collaboration
**What**: Multiple users editing same resume
**Complexity**: Large
**Dependencies**: Epic 6.1 (Sharing System)

**Tasks**:
- [ ] Use Cloudflare Durable Objects for state management
- [ ] WebSocket connections for real-time updates
- [ ] Operational transformation for conflict resolution
- [ ] Presence detection (show active editors)
- [ ] Cursor position sharing
- [ ] Change history tracking

**Acceptance Criteria**:
- [ ] Multiple users can edit simultaneously
- [ ] Changes synced in real-time
- [ ] No data loss during conflicts
- [ ] Presence indicators work correctly
- [ ] Only available to Premium users

---

### Epic 6.3: Usage Analytics
**What**: Track resume and website usage
**Complexity**: Medium
**Dependencies**: Epic 4.2 (Website Management)

**Tasks**:
- [ ] `GET /api/analytics/resume/:id` - Resume analytics
- [ ] `GET /api/analytics/website/:id` - Website analytics
- [ ] Track resume views, downloads, shares
- [ ] Track website visits (integrate Cloudflare Analytics)
- [ ] Store aggregated metrics in D1
- [ ] Time-series data for trends

**Acceptance Criteria**:
- [ ] Resume analytics tracked accurately
- [ ] Website analytics integrated with Cloudflare
- [ ] Metrics aggregated correctly
- [ ] Historical data retained
- [ ] Only available to Pro/Premium users

---

## Critical Path (Build Order)

1. **MVP-1.1**: Database Schema ⟶ Foundation for everything
2. **MVP-1.2**: Authentication Middleware ⟶ Depends on SSO JWT (blocks all endpoints)
3. **MVP-1.3**: Resume CRUD ⟶ Core functionality
4. **MVP-1.4**: Template Management ⟶ Required for resume rendering
5. **MVP-1.5**: Basic PDF Export ⟶ Essential user feature
6. **MVP-2.1**: Stripe Integration ⟶ Enable paid features
7. **MVP-2.2**: Feature Gating ⟶ Enforce tier limits
8. **MVP-3.1**: PDF Import ⟶ User onboarding
9. **MVP-3.2**: DOCX Import ⟶ Additional import option
10. **MVP-3.3**: LinkedIn Import ⟶ Fast resume creation
11. **MVP-3.4**: Multi-format Export ⟶ User flexibility
12. **MVP-4.1**: Static Site Generation ⟶ Premium feature
13. **MVP-4.2**: Website Management ⟶ Website updates
14. **MVP-4.3**: Custom Domain ⟶ Premium enhancement
15. **MVP-5.1**: Content Suggestions ⟶ AI value-add
16. **MVP-5.2**: Resume Analysis ⟶ ATS optimization
17. **MVP-5.3**: Smart Rewriting ⟶ Premium AI feature
18. **MVP-6.1**: Sharing System ⟶ Collaboration foundation
19. **MVP-6.2**: Real-time Collaboration ⟶ Advanced collaboration
20. **MVP-6.3**: Usage Analytics ⟶ User insights

---

## What Blocks What

| This Epic | Blocks These Services/Features |
|-----------|--------------------------------|
| MVP-1.1 (Database) | Everything in Resume-API |
| MVP-1.2 (Authentication) | All API endpoints requiring auth |
| MVP-1.3 (Resume CRUD) | Resume-UI editor, Resume-Mobile-UI editor, Resume-Processor jobs |
| MVP-1.4 (Templates) | Resume-UI template selection, Resume-Mobile-UI templates |
| MVP-1.5 (PDF Export) | Resume-UI downloads, Resume-Mobile-UI downloads, Resume-Processor PDF jobs |
| MVP-2.1 (Stripe) | All Pro/Premium features across Resume-UI, Resume-Mobile-UI |
| MVP-2.2 (Feature Gating) | Premium enforcement in Resume-UI, Resume-Mobile-UI |
| MVP-3.3 (LinkedIn Import) | LinkedIn features in Resume-UI, Resume-Mobile-UI |
| MVP-4.1 (Site Generation) | Website builder in Resume-UI |
| MVP-5.1 (AI Suggestions) | AI features in Resume-UI, Resume-Processor AI jobs |
| MVP-6.1 (Sharing) | Share functionality in Resume-UI, Resume-Mobile-UI |
| MVP-6.2 (Collaboration) | Real-time editing in Resume-UI |
| MVP-6.3 (Analytics) | Analytics dashboard in Resume-Admin |

**External Dependencies**:
- Resume-SSO (MVP-1.4: Login with JWT) ⟶ Blocks Epic 1.2 (Authentication Middleware)
- Resume-SSO (MVP-2.1: JWT Validation) ⟶ Required for all authenticated endpoints
- Resume-SSO (MVP-3.2: LinkedIn OAuth) ⟶ Required for Epic 3.3 (LinkedIn Import)

---

## Testing Strategy

### Unit Tests
- Resume CRUD operations
- PDF generation logic
- Feature gating middleware
- AI prompt engineering
- Template rendering

### Integration Tests
- Full resume creation → PDF export flow
- Import PDF → save as resume flow
- Website generation → deployment flow
- Stripe webhook → subscription update flow
- Real-time collaboration sync

### Performance Tests
- PDF generation under load
- API response times
- Concurrent user handling
- Queue processing throughput
- Database query optimization

### Security Tests
- Authorization checks (users can only access own resumes)
- JWT validation
- SQL injection attempts
- File upload validation
- Rate limit enforcement

---

## API Endpoints Summary

### Public Endpoints (with authentication)
- `POST /api/resumes` - Create resume
- `GET /api/resumes` - List resumes
- `GET /api/resumes/:id` - Get resume
- `PUT /api/resumes/:id` - Update resume
- `DELETE /api/resumes/:id` - Delete resume
- `POST /api/resumes/:id/duplicate` - Duplicate resume
- `GET /api/templates` - List templates
- `GET /api/templates/:id` - Get template
- `POST /api/resumes/:id/export/pdf` - Export PDF
- `POST /api/resumes/:id/export/docx` - Export DOCX (Pro+)
- `POST /api/resumes/:id/export/batch` - Batch export (Pro+)

### Import Endpoints (Pro+ only)
- `POST /api/import/pdf` - Import from PDF
- `POST /api/import/docx` - Import from Word
- `POST /api/import/linkedin` - Import from LinkedIn

### Payment Endpoints
- `POST /api/billing/create-checkout` - Start subscription
- `POST /api/billing/webhook` - Stripe webhook
- `GET /api/billing/subscription` - Get subscription status
- `POST /api/billing/cancel` - Cancel subscription

### Website Endpoints (Pro+ only)
- `POST /api/websites/generate` - Generate website
- `GET /api/websites/:id` - Get website
- `PUT /api/websites/:id` - Update website
- `DELETE /api/websites/:id` - Delete website
- `POST /api/websites/:id/publish` - Publish website

### AI Endpoints (Pro+ only)
- `POST /api/ai/suggest-bullets` - AI bullet points
- `POST /api/ai/analyze` - Resume analysis
- `POST /api/ai/rewrite` - Content rewriting (Premium)

### Collaboration Endpoints (Premium only)
- `POST /api/resumes/:id/share` - Create share link
- `GET /api/shared/:token` - Access shared resume
- WebSocket `/api/collaborate/:id` - Real-time collaboration

### Analytics Endpoints (Pro+ only)
- `GET /api/analytics/resume/:id` - Resume analytics
- `GET /api/analytics/website/:id` - Website analytics

---

## Environment Variables

```bash
# Cloudflare Bindings
DB=<D1 database binding>
R2_BUCKET=<R2 bucket binding>
KV_CACHE=<KV namespace for caching>
PDF_QUEUE=<Queue binding for PDF generation>
AI=<Workers AI binding>

# Stripe Configuration
STRIPE_SECRET_KEY=<stripe-secret>
STRIPE_WEBHOOK_SECRET=<webhook-secret>

# JWT Configuration (from Resume-SSO)
JWT_PUBLIC_KEY=<rsa-public-key>

# Cloudflare Pages (for website deployment)
CF_PAGES_API_TOKEN=<pages-api-token>
CF_ACCOUNT_ID=<account-id>

# Rate Limiting
RATE_LIMIT_FREE=100/hour
RATE_LIMIT_PRO=1000/hour
RATE_LIMIT_PREMIUM=5000/hour
```

---

## Success Metrics

- [ ] 100% API endpoint test coverage
- [ ] < 200ms response time for CRUD operations
- [ ] < 5s PDF generation time (p95)
- [ ] Zero data loss in resume operations
- [ ] 99.9% uptime
- [ ] Feature gating prevents unauthorized access
- [ ] AI suggestions rated useful by 80%+ users
- [ ] Website generation success rate > 95%

# BEFORE YOU CODE

## Development Process for Resume Builder SaaS

Follow this process for ALL development work in this repository. **DO NOT skip steps.**

---

## 1. LOG EVERYTHING

- Add comprehensive console logging at key points
- **Log to file locally** for debugging and analysis
- Log function entry/exit points
- Log important variables and state changes
- Log errors with full context (stack traces, input data)
- Use structured logging with clear prefixes:
  ```typescript
  console.log('[ServiceName] Action: details');
  console.error('[ServiceName] Error:', error);
  ```
- **Local Development**: Logs written to console/terminal for debugging
- **Production**: Cloudflare handles all production logging (Workers Analytics, Logpush)
- **Enhanced Logging**: Use JSON.stringify for complex objects

---

## 2. TDD (Test-Driven Development)

**Write tests BEFORE writing implementation code.**

### TDD Cycle:
1. **RED**: Write a failing test
2. **GREEN**: Write minimal code to make it pass
3. **REFACTOR**: Clean up code while keeping tests green

### Test Structure:
- API tests: `Resume-API/src/__tests__/`
- SSO tests: `Resume-SSO/src/__tests__/`
- UI tests: `Resume-UI/__tests__/` or `Resume-UI/src/__tests__/`
- Admin tests: `Resume-Admin/__tests__/`
- Use descriptive test names: `should [expected behavior] when [condition]`

### Running Tests:
```bash
# For Cloudflare Workers (using Vitest)
cd Resume-API
npm test

# For Next.js apps
cd Resume-UI
npm test

# Type check
npm run type-check
```

---

## 3. BUILD

Always build the project to catch TypeScript errors:

```bash
# For Cloudflare Workers
cd Resume-API
npm run build
# OR
wrangler deploy --dry-run

# For Next.js apps
cd Resume-UI
npm run build

# Type check without building
npm run type-check
```

Fix ALL build errors before proceeding.

---

## 4. RUN TESTS

Run the test suite to ensure nothing is broken:

```bash
# Type check
npm run type-check

# Lint
npm run lint

# Run tests
npm test

# Run tests in watch mode
npm test -- --watch
```

**All tests must pass before committing.**

---

## 5. DEBUG ISSUES SYSTEMATICALLY

**When encountering bugs or errors, follow this debugging process:**

### Debugging Workflow:

1. **Check Logs First**
   - Review console/terminal output from dev servers
   - Check browser console for frontend errors
   - Check Cloudflare Workers logs: `wrangler tail`
   - Look for error patterns and stack traces
   - Use structured log prefixes to filter: `[ServiceName]`, `[Auth]`, `[Error]`

2. **If Logs Are Insufficient, Add More Logging**
   - Add comprehensive logging at key decision points
   - Log function entry/exit with parameters
   - Log state changes and variable values
   - Use JSON.stringify for complex objects:
     ```typescript
     console.error('[ServiceName] Error Details:', JSON.stringify(error, null, 2));
     ```

3. **Ensure Logs Are Reachable and Actionable**
   - **Local Development**: Logs go to console/terminal (`wrangler dev`)
   - **Production**: Use `wrangler tail` or Cloudflare Dashboard logs
   - Logs must contain enough context to reproduce and fix issues
   - Include: timestamp, service name, action, input data, error details, stack traces

4. **Fix Until All Tests Pass**
   - **No exceptions**: All tests must pass, no matter the reason for failure
   - If a test fails, fix the underlying issue (don't skip, comment out, or disable tests)
   - If test is flaky, fix the flakiness (don't just re-run)
   - If test is outdated, update the test and implementation together

5. **Ensure All Services Run Without Errors**
   - **Development standard**: All services should run cleanly with zero errors
   - Fix all TypeScript errors: `npm run type-check` must pass
   - Fix all linting errors: `npm run lint` must pass
   - If dev servers show errors, fix them before committing
   - Common errors to watch for:
     - D1 database connection failures
     - R2 bucket connection issues
     - TypeScript compilation errors
     - Missing environment variables
     - Port conflicts
     - CORS issues
     - JWT validation errors

### Debugging Checklist:
- [ ] Logs reviewed and error identified
- [ ] Additional logging added if needed (with structured prefixes)
- [ ] Logs are reachable and contain actionable context
- [ ] Root cause identified (not just symptoms)
- [ ] Fix implemented and tested
- [ ] All tests pass: `npm test` and `npm run type-check`
- [ ] All dev servers run without errors
- [ ] No error messages in console/terminal
- [ ] Enhanced logging left in place (don't remove debugging logs that add value)

### Common Debugging Scenarios:

**D1 Database Connection Errors:**
```bash
# Symptom: "Failed to connect to D1"
# Fix: Check wrangler.toml bindings and restart dev server
wrangler dev
```

**R2 Storage Issues:**
```bash
# Symptom: "R2 bucket not found" or "Access denied"
# Fix: Verify R2 bucket name in wrangler.toml and permissions
```

**Browser Cache Issues:**
```bash
# Symptom: Code changes not reflected in browser
# Fix: Hard refresh to clear cache
# Ctrl+Shift+R (Linux/Windows) or Cmd+Shift+R (Mac)
```

**Environment Binding Errors:**
```typescript
// Symptom: "Cannot read property 'DB' of undefined"
// Fix: Use c.env instead of env in Hono routes
const db = c.env.DB;  // ✅ CORRECT
```

**JWT/Authentication Failures:**
```bash
# Symptom: "Invalid token" or "Unauthorized"
# Fix: Check JWT secret configuration and token expiration
```

---

## 6. RESEARCH, RESEARCH, RESEARCH

Before writing ANY new code:

### Research Checklist:
- [ ] Search the codebase for existing implementations
- [ ] Check documentation:
  - Root `CLAUDE.md` - Project overview and architecture
  - Service-specific READMEs in each subdirectory
  - Feature plans in `Resume-UI/github-issues/`
- [ ] Review similar patterns in the codebase
- [ ] Look for existing utilities/helpers in `src/lib/` or `src/utils/`
- [ ] Check database schema in `schema.sql` or migrations
- [ ] Check test files for usage examples
- [ ] Read related issues/PRs in GitHub
- [ ] Review Cloudflare Workers documentation for the feature you're implementing

**If unsure, ASK. Don't guess.**

---

## 7. CHECK CODEBASE & DOCUMENTATION BEFORE WRITING NEW CODE

### Before Creating New Files:
1. Search for existing files that might do similar things
2. Check if the functionality exists elsewhere
3. Review the multi-repo structure to understand where new code belongs:
   - `Resume-UI/` - Next.js frontend (user-facing)
   - `Resume-API/` - Hono API backend (Cloudflare Workers)
   - `Resume-SSO/` - Authentication service (Cloudflare Workers)
   - `Resume-Admin/` - Admin dashboard (Next.js)
   - `Resume-Admin-API/` - Admin backend (Cloudflare Workers)
   - `Resume-Processor/` - Background jobs (Cloudflare Queues)

### Before Writing New Functions:
1. Search for similar functions in the codebase
2. Check utility files in `src/lib/` or `src/utils/`
3. Review database schema and models
4. Check if shared types or utilities can be extracted

### Code Reuse Priorities:
1. **ALWAYS** prefer editing existing files over creating new ones
2. **ALWAYS** prefer using existing utilities over writing new ones
3. **ALWAYS** follow established patterns in the codebase
4. **ALWAYS** maintain API contracts between services

---

## 8. PULL, COMMIT, PUSH

### Before Starting Work:
```bash
# Navigate to the specific repository you're working on
cd Resume-API  # or Resume-UI, Resume-SSO, etc.

# Pull latest changes
git pull origin main
```

### After Making Changes:
```bash
# Stage changes
git add .

# Commit with descriptive message
git commit -m "type(scope): description

- Detailed bullet point 1
- Detailed bullet point 2

Fixes #issue-number (if applicable)"

# Push to branch
git push origin <branch-name>
```

### Commit Message Format:
- `feat(api):` New feature in Resume-API
- `feat(ui):` New feature in Resume-UI
- `feat(sso):` New feature in Resume-SSO
- `feat(admin):` New feature in Resume-Admin
- `feat(processor):` New feature in Resume-Processor
- `fix(api):` Bug fix in Resume-API
- `refactor(ui):` Code refactoring in Resume-UI
- `test(sso):` Adding/updating tests in Resume-SSO
- `docs:` Documentation changes
- `chore:` Maintenance tasks
- `build:` Build system or dependency changes

### Scope Examples:
- `feat(api): add resume export to Word format`
- `fix(sso): correct JWT token expiration handling`
- `refactor(ui): improve resume editor performance`
- `feat(processor): add PDF generation queue processing`
- `docs: update CLAUDE.md with authentication flow`

---

## 9. REVIEW CODE

Before opening a PR, perform self-review:

### Code Review Checklist:
- [ ] Code follows existing patterns and style
- [ ] All console.log statements are meaningful (not debug leftovers)
- [ ] No commented-out code
- [ ] **NO TODO comments** - Create GitHub issues instead
- [ ] Type safety maintained (no `any` types without justification)
- [ ] Error handling is comprehensive
- [ ] Edge cases are handled
- [ ] Performance implications considered
- [ ] Security implications considered (especially for auth, file uploads, payments)
- [ ] Cloudflare Workers patterns followed correctly:
  - Use `c.env.bindingName` not `env.bindingName`
  - Proper D1, R2, KV, Queues, Durable Objects usage
  - JWT validation configured correctly
  - Rate limiting implemented where needed
- [ ] Feature gating based on subscription tier works correctly
- [ ] Database migrations are included (if schema changes)
- [ ] API contracts maintained (if API changes affect multiple services)

---

## 10. OPEN PRs

### PR Requirements:
- Descriptive title following commit message format
- Clear description of what changed and why
- Link to related issues
- Screenshots/videos for UI changes
- Test results included
- All CI checks passing
- Specify which services are affected

### PR Template:
```markdown
## Description
[What does this PR do?]

## Motivation
[Why is this change needed?]

## Changes
- Change 1
- Change 2

## Services Affected
- [ ] Resume-UI (Next.js frontend)
- [ ] Resume-API (Backend API)
- [ ] Resume-SSO (Authentication)
- [ ] Resume-Admin (Admin dashboard)
- [ ] Resume-Admin-API (Admin backend)
- [ ] Resume-Processor (Background jobs)

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed
- [ ] All tests passing
- [ ] Type checking passes: `npm run type-check`
- [ ] Build succeeds: `npm run build`

## Database Changes
- [ ] No database changes
- [ ] Migrations included (describe them)

## API Changes
- [ ] No API changes
- [ ] API changes documented (describe contract changes)

## Feature Gating
- [ ] No premium features added
- [ ] Premium features properly gated by subscription tier

## Related Issues
Fixes #[issue-number]

## Screenshots
[If applicable]

## Deployment Notes
[Any special deployment considerations]
```

---

## Environment Notes

- **Package Manager**: Each repository uses **npm** (standard Node.js package manager)
- **Build System**: Each service is independent
- **Local Development**:
  - Resume-UI: http://localhost:3000 (Next.js)
  - Resume-API: http://localhost:8787 (Wrangler dev)
  - Resume-SSO: http://localhost:8788 (Wrangler dev)
  - Resume-Admin: http://localhost:3001 (Next.js)
- **Production**: Cloudflare handles all production infrastructure
- **No TODOs in code**: Create GitHub issues instead of leaving TODO comments

---

## Quick Reference

```bash
# Complete development workflow (for a Cloudflare Worker service)

# 1. Navigate to service directory
cd Resume-API  # or Resume-SSO, Resume-Admin-API, Resume-Processor

# 2. Pull latest
git pull origin main

# 3. Create branch
git checkout -b feat/my-feature

# 4. Install dependencies (if needed)
npm install

# 5. Start dev server
wrangler dev
# OR for Next.js apps
npm run dev

# 6. Make changes and write tests (TDD)

# 7. Type check
npm run type-check

# 8. Lint
npm run lint

# 9. Run tests
npm test

# 10. Build (test deployment for Workers)
wrangler deploy --dry-run
# OR for Next.js
npm run build

# 11. Commit
git add .
git commit -m "feat(api): add feature description"

# 12. Push
git push origin feat/my-feature

# 13. Open PR on GitHub
```

---

## Service-Specific Commands

### Cloudflare Workers (Resume-API, Resume-SSO, Resume-Admin-API, Resume-Processor)

```bash
# Install dependencies
npm install

# Dev server with live reload
wrangler dev

# Type checking
npm run type-check

# Run tests
npm test

# Deploy to production
wrangler deploy

# View logs
wrangler tail

# Manage D1 database
wrangler d1 execute resume-db --command "SELECT * FROM users LIMIT 10"

# Apply migrations
wrangler d1 migrations apply resume-db

# Manage KV
wrangler kv:key put --binding=CACHE "mykey" "myvalue"
wrangler kv:key get --binding=CACHE "mykey"

# Manage R2
wrangler r2 bucket list
wrangler r2 object list resume-files

# Manage secrets
wrangler secret put JWT_SECRET
wrangler secret put STRIPE_SECRET_KEY
```

### Next.js Apps (Resume-UI, Resume-Admin)

```bash
# Install dependencies
npm install

# Dev server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Lint
npm run lint

# Run tests
npm test
```

---

## Common Violations to AVOID

❌ **DON'T:**
- Write code before tests
- Skip the build step
- Commit without running tests and type checking
- Create new files without checking for existing ones
- Push directly to main
- Skip code review
- Leave debug logging in production code
- Ignore TypeScript errors
- Guess at implementation without research
- Add TODO comments (create GitHub issues instead)
- Use `env.bindingName` in Cloudflare Workers (use `c.env.bindingName`)
- Forget to apply database migrations before deployment
- Break subscription tier feature gating
- Ignore CORS configuration
- Expose sensitive data in logs (passwords, API keys, payment info)

✅ **DO:**
- Follow the 10-step process
- Keep tests green
- Review your own code
- Ask questions when unsure
- Document your changes
- Follow established patterns
- Add meaningful logs with structured prefixes
- Fix all warnings
- Create GitHub issues for future work
- Use Cloudflare Workers patterns correctly
- Test feature gating (Free vs Pro vs Premium)
- Validate JWT tokens properly
- Handle errors gracefully
- Use environment variables for sensitive data
- Apply database migrations before deployment
- Test API endpoints with authentication

---

## Critical Patterns for This Codebase

### 1. Cloudflare Workers Bindings (Hono Framework)
```typescript
// ❌ WRONG
const db = env.DB;
const bucket = env.R2_BUCKET;

// ✅ CORRECT
const db = c.env.DB;
const bucket = c.env.R2_BUCKET;
```

### 2. Authentication & JWT
```typescript
// ✅ Validate JWT in middleware
import { jwt } from 'hono/jwt';

app.use('/api/*', jwt({
  secret: c.env.JWT_SECRET,
}));

// ✅ Access user from JWT payload
const userId = c.get('jwtPayload').sub;
```

### 3. Feature Gating by Subscription Tier
```typescript
// ✅ Check subscription tier before premium features
const user = await db.prepare('SELECT subscription_tier FROM users WHERE id = ?')
  .bind(userId)
  .first();

if (user.subscription_tier === 'free' && action === 'import-resume') {
  return c.json({ error: 'Upgrade to Pro required' }, 403);
}
```

### 4. Database Queries (D1)
```typescript
// ✅ Use prepared statements to prevent SQL injection
const resumes = await c.env.DB.prepare(
  'SELECT * FROM resumes WHERE user_id = ? ORDER BY updated_at DESC'
)
  .bind(userId)
  .all();
```

### 5. R2 Storage Operations
```typescript
// ✅ Upload file to R2
await c.env.RESUME_FILES.put(`pdfs/${resumeId}.pdf`, pdfBuffer, {
  httpMetadata: {
    contentType: 'application/pdf',
  },
});

// ✅ Get file from R2
const object = await c.env.RESUME_FILES.get(`pdfs/${resumeId}.pdf`);
if (!object) {
  return c.json({ error: 'File not found' }, 404);
}
```

### 6. Background Jobs (Cloudflare Queues)
```typescript
// ✅ Enqueue job
await c.env.PDF_GENERATION_QUEUE.send({
  resumeId: resumeId,
  userId: userId,
  timestamp: Date.now(),
});

// ✅ Process job in Resume-Processor
export default {
  async queue(batch, env) {
    for (const message of batch.messages) {
      const { resumeId, userId } = message.body;
      // Process PDF generation
    }
  }
};
```

### 7. Error Logging with Context
```typescript
// ✅ Use structured logging with JSON.stringify for objects
console.error('[Resume-API] PDF Generation Failed:', JSON.stringify({
  resumeId,
  userId,
  error: error.message,
  stack: error.stack,
}, null, 2));
```

### 8. Rate Limiting
```typescript
// ✅ Implement rate limiting for API endpoints
const rateLimiter = new RateLimiter(c.env.RATE_LIMIT_KV);
const allowed = await rateLimiter.check(userId, 100); // 100 requests per hour
if (!allowed) {
  return c.json({ error: 'Rate limit exceeded' }, 429);
}
```

### 9. CORS Configuration
```typescript
// ✅ Configure CORS for API endpoints
import { cors } from 'hono/cors';

app.use('/api/*', cors({
  origin: ['https://resume-builder.com', 'http://localhost:3000'],
  credentials: true,
}));
```

---

## Architecture Decision Records (ADRs)

For significant architectural decisions:
1. Document in service-specific README or create `docs/architecture/` directory
2. Include problem statement, proposed solution, alternatives considered
3. Reference feature plans in `Resume-UI/github-issues/`

---

**Remember: Quality > Speed. Take time to do it right the first time.**

**Each service is an independent repository. Always navigate to the correct service directory before running commands.**

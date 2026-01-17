# Algorithm Trainer - Epic Breakdown

## Epic Overview

This document breaks down the Algorithm Trainer project into implementable epics and stories, optimized for demonstrating full-stack engineering skills for graduate software engineering roles.

**Key differentiators for interviews:**
- Async worker architecture with Redis/RQ
- Production-ready infrastructure (Docker, CI/CD, observability)
- Comprehensive testing strategy
- End-to-end deployment

---

## Epic 1 — Foundations & Infrastructure

### Story 1.1: Repository structure and tooling

**As a developer**, I want a well-organized monorepo with consistent tooling so the project is easy to run and contribute to.

**Acceptance criteria:**
- Repository structure:
  ```
  /frontend        # Vite + React + TypeScript
  /backend         # FastAPI + Python
  /docs            # PRD, epics, architecture diagrams
  docker-compose.yml
  .env.example
  ```
- Root `README.md` includes:
  - One-sentence project summary
  - Tech stack overview
  - Quick start guide (Docker Compose)
  - Architecture diagram (simple: client → API → worker → DB)
  - Contributing guidelines
- `docs/PRD.md` exists
- `docs/EPICS.md` exists (this file)
- `.gitignore` configured for Python, Node, env files
- Backend uses `.env` for secrets (never committed)
- Frontend uses `.env.local` for config

**Technical notes:**
- Use Vite for frontend (fast HMR, modern tooling)
- Use Poetry or pip-tools for Python dependency management
- Setup pre-commit hooks (linting, formatting)

---

### Story 1.2: Docker development environment

**As a developer**, I want a containerized local environment so setup is consistent across machines and mirrors production.

**Acceptance criteria:**
- `docker-compose.yml` includes services:
  - `frontend` (Vite dev server, hot reload)
  - `backend` (FastAPI with auto-reload)
  - `postgres` (Supabase local or standalone)
  - `redis` (for RQ)
  - `worker` (RQ worker process)
- `Dockerfile.backend` (production-ready, multi-stage build)
- `Dockerfile.frontend` (nginx serving static build)
- `make` or npm scripts for common commands:
  - `make dev` - starts all services
  - `make test` - runs all tests
  - `make migrate` - runs database migrations
- Health check endpoints for backend and worker
- All services start with `docker compose up`
- Volume mounts for hot-reload in development

**Technical notes:**
- Use multi-stage Docker builds to minimize image size
- Backend base image: `python:3.11-slim`
- Frontend production: `nginx:alpine`
- Environment variable interpolation for different stages (dev/prod)

---

### Story 1.3: Database migrations and schema

**As a developer**, I want database migrations so schema changes are tracked and reproducible.

**Acceptance criteria:**
- Alembic configured in backend
- Initial migration creates tables:
  - `users` (minimal, Supabase Auth handles most fields)
  - `questions` (id, user_id, title, leetcode_url, prompt_summary, tags, created_at)
  - `attempts` (id, user_id, question_id, explanation_text, timer_seconds, status, created_at)
  - `feedbacks` (id, attempt_id, rubric_scores json, key_issues text[], improved_example text, corrected_complexity text, confidence_rating int)
  - `analysis_jobs` (id, attempt_id, status, error_message, created_at, completed_at)
- Migration runs automatically in Docker on startup
- `make migrate` command for manual migration
- Indexes on foreign keys and frequently queried columns
- Migrations tested in CI

**Technical notes:**
- Use Alembic async support for FastAPI
- Store rubric scores as JSONB for flexibility
- Add enum for attempt status: `pending`, `processing`, `completed`, `failed`
- Add created_at, updated_at timestamps on all tables

---

### Story 1.4: Testing infrastructure

**As a developer**, I want comprehensive testing setup so I can develop with confidence and demonstrate testing skills.

**Acceptance criteria:**

**Backend:**
- `pytest` configured with async support
- Test fixtures for:
  - Database session (rollback after each test)
  - Redis client (flushed after tests)
  - Authenticated test user
  - Sample questions and attempts
- Separate test database (configured in `pytest.ini`)
- Coverage reporting (target: >80%)
- Test directory structure:
  ```
  /backend/tests
    /unit          # Business logic, utils
    /integration   # API endpoints, DB queries
    /workers       # RQ job tests
  ```

**Frontend:**
- Vitest + React Testing Library
- Test coverage for:
  - Component rendering
  - User interactions
  - API integration (mocked)
- Coverage reporting (target: >70%)

**Commands:**
- `make test-backend` - runs backend tests
- `make test-frontend` - runs frontend tests
- `make test` - runs all tests
- `make coverage` - generates coverage report

**Technical notes:**
- Use `pytest-asyncio` for async tests
- Use `fakeredis` for Redis tests (no external dependency)
- Use `respx` or `httpx` mock for external API calls (OpenAI)
- Frontend: Mock API calls with MSW (Mock Service Worker)

---

### Story 1.5: CI/CD pipeline

**As a developer**, I want automated checks on every PR so main branch stays healthy.

**Acceptance criteria:**
- GitHub Actions workflow on PR and push to main:
  - **Lint:** Backend (ruff/black) + Frontend (eslint/prettier)
  - **Type check:** Backend (mypy) + Frontend (tsc)
  - **Test:** Backend (pytest) + Frontend (vitest)
  - **Coverage:** Report to PR comments
  - **Build:** Docker images build successfully
- Separate jobs for backend/frontend (run in parallel)
- Cache dependencies for faster runs
- All checks must pass before merge
- Status badges in README

**Technical notes:**
- Use GitHub Actions matrix for parallel jobs
- Cache Docker layers
- Use `actions/cache` for npm and pip dependencies
- Consider adding security scanning (Snyk, Dependabot)

---

### Story 1.6: API documentation

**As a developer**, I want auto-generated API docs so the contract between frontend and backend is clear.

**Acceptance criteria:**
- FastAPI auto-generates OpenAPI/Swagger docs at `/docs`
- All endpoints documented with:
  - Request/response schemas (Pydantic models)
  - HTTP status codes
  - Error response examples
- Endpoint groups organized by feature:
  - `/auth/*` - Authentication
  - `/questions/*` - Question CRUD
  - `/attempts/*` - Attempt CRUD
  - `/analysis/*` - Analysis jobs
  - `/stats/*` - User statistics
- Include example requests in documentation
- Docs accessible in Docker environment

**Technical notes:**
- Use Pydantic models for request/response validation
- Add `summary` and `description` to each endpoint
- Use FastAPI tags for grouping
- Add `responses` parameter to document error cases

---

## Epic 2 — Authentication & User Identity

### Story 2.1: Supabase Auth integration

**As a user**, I want to sign up and log in so my practice history is saved.

**Acceptance criteria:**

**Backend:**
- Supabase Auth SDK configured
- JWT verification middleware for protected routes
- Helper to extract `user_id` from verified JWT
- Error handling for invalid/expired tokens
- `GET /auth/me` endpoint returns current user info

**Frontend:**
- Supabase client configured with env variables
- Sign up form (email + password)
- Login form (email + password)
- Logout button
- Auth state persisted in localStorage
- Protected routes redirect to login if unauthenticated
- JWT included in all API requests (Authorization header)

**UI states:**
- Loading (checking auth)
- Logged out (show login/signup)
- Logged in (show app)

**Technical notes:**
- Use Supabase JWT verification (not custom)
- Store token in httpOnly cookie or secure localStorage
- Add auth context/provider in React
- Use React Router for protected routes

---

### Story 2.2: User session management

**As a user**, I want to stay logged in across sessions and have secure token refresh.

**Acceptance criteria:**
- Access token refresh before expiry
- Backend validates token on every request
- Frontend handles 401 responses (redirect to login)
- "Remember me" option extends session
- Logout clears all client-side state

**Technical notes:**
- Supabase handles refresh token rotation
- Set session persistence in Supabase client config
- Add request interceptor for token refresh

---

## Epic 3 — Questions (Link + Short Prompt)

### Story 3.1: Create a question

**As a user**, I want to add a LeetCode question with a summary so I can practice explaining it later.

**Acceptance criteria:**

**Backend:**
- `POST /questions` endpoint
  - Request: `{ title, leetcode_url, prompt_summary, tags[] }`
  - Response: Created question object
  - Validation:
    - Title required (1-200 chars)
    - URL must be valid HTTPS and contain "leetcode.com"
    - Prompt summary required (10-1000 chars)
    - Tags optional (max 5 tags)
  - Associates question with authenticated user

**Frontend:**
- "Add Question" form with fields:
  - Title (text input)
  - LeetCode URL (URL input with validation)
  - Short summary prompt (textarea, 10-1000 chars)
  - Tags (multi-select or comma-separated input)
- Client-side validation before submit
- Success message on creation
- Redirect to questions list after creation
- Error handling (display validation errors)

**Technical notes:**
- URL validation regex: `^https://leetcode\.com/problems/[\w-]+/?$`
- Backend uses Pydantic for validation
- Frontend uses React Hook Form + Zod

---

### Story 3.2: View and manage questions

**As a user**, I want to browse my questions and see practice history so I can track what I've practiced.

**Acceptance criteria:**

**Backend:**
- `GET /questions` endpoint
  - Returns user's questions with metadata:
    - Question details
    - Attempt count
    - Last attempted date
    - Average rubric score (if attempts exist)
  - Query parameters:
    - `tag` - filter by tag
    - `search` - search in title/prompt
    - `page`, `limit` - pagination (default 20 per page)
    - `sort` - by date, attempts, score
- `GET /questions/{id}` - get single question with full attempt history
- `PUT /questions/{id}` - update question
- `DELETE /questions/{id}` - soft delete (keep attempts)

**Frontend:**
- Questions list page showing:
  - Title, tags, attempt count, last attempted, avg score
  - Search bar
  - Filter by tag dropdown
  - Sort options
  - Pagination controls
- Click question to view detail page:
  - Full question info
  - All attempts for that question
  - "Start new attempt" button
- Edit/delete buttons (with confirmation for delete)

**Technical notes:**
- Use SQL JOIN to get attempt counts in single query
- Implement cursor or offset pagination
- Frontend: Debounce search input
- Cache questions list in React Query/SWR

---

## Epic 4 — Daily Drill Suggestion

### Story 4.1: Suggestion algorithm

**As a user**, I want the app to suggest what to practice next so I don't have to decide.

**Acceptance criteria:**

**Backend:**
- `GET /suggestions/daily-drill` endpoint
- Suggestion algorithm (v1):
  1. Get all user's questions
  2. Calculate attempt count per question
  3. Calculate average score per tag
  4. Select least attempted question
  5. Tie-break by weakest tag (lowest avg score)
  6. Return suggested question with reasoning
- Response includes:
  ```json
  {
    "question": { /* question object */ },
    "reason": "Least practiced (0 attempts)",
    "weak_tags": ["dynamic-programming"]
  }
  ```
- Returns 404 if user has no questions

**Frontend:**
- Dashboard has "Start Daily Drill" card
- Shows suggested question title and reason
- "Start Drill" button (navigates to attempt page)
- "Choose Different" link (navigates to questions list)
- Skeleton loader while fetching

**Technical notes:**
- Endpoint should be fast (<500ms)
- Consider caching suggestion for 24hrs
- Algorithm can be improved later (ML, spaced repetition)

---

## Epic 5 — Attempt Creation (Guided Explanation)

### Story 5.1: Create an attempt

**As a user**, I want to write my explanation with guided prompts and an optional timer.

**Acceptance criteria:**

**Backend:**
- `POST /attempts` endpoint
  - Request: `{ question_id, explanation_text, timer_seconds }`
  - Creates attempt with status `pending`
  - Queues analysis job (Epic 6)
  - Response: Attempt object with `analysis_job_id`
- `GET /attempts/{id}` - get attempt details
- Input validation:
  - Explanation required (min 50 chars, max 10,000 chars)
  - Question must exist and belong to user
  - Timer optional (0 = no timer)

**Frontend:**
- Attempt page for a question shows:
  - Question title and LeetCode link (opens in new tab)
  - User's prompt summary
  - Guided template prompts (visible, not editable):
    ```
    1. Problem Summary
    2. Approach & Algorithm
    3. Step-by-Step Explanation
    4. Time Complexity (with justification)
    5. Space Complexity (with justification)
    6. Edge Cases Considered
    ```
  - Large textarea for explanation (below prompts)
  - Character count (50-10,000)
  - Optional timer:
    - Toggle to enable
    - Presets: Daily Drill (20 min), Mock Interview (45 min), Custom
    - Countdown display
    - Alert when time's up (doesn't auto-submit)
  - "Submit for Feedback" button
- After submit:
  - Show "Analyzing..." state
  - Poll for job completion (every 2-3 seconds)
  - Disable back navigation during analysis
- Error handling:
  - Show errors from validation
  - Show retry button if submission fails

**Technical notes:**
- Store timer value even if time's up (for stats)
- Use localStorage to save draft explanation
- Clear draft after successful submit
- Textarea auto-resize or fixed height with scroll

---

### Story 5.2: Confidence self-rating

**As a user**, I want to rate my confidence after seeing feedback so I can track my subjective improvement.

**Acceptance criteria:**
- After feedback loads, show confidence rating prompt:
  - "How confident are you in your explanation now?"
  - 1-5 scale (1 = Not confident, 5 = Very confident)
  - Can be skipped
- `POST /attempts/{id}/confidence` endpoint saves rating
- Rating stored in `feedbacks` table
- Rating shown in attempt history

**Technical notes:**
- Show as modal or inline form after feedback
- Optional field in DB (nullable)
- Consider showing rating trend over time

---

## Epic 6 — AI Analysis Pipeline (Async)

### Story 6.1: Analysis job creation

**As a user**, I want AI feedback generated reliably even if it takes time, without blocking the app.

**Acceptance criteria:**

**Backend:**
- When attempt submitted, create entry in `analysis_jobs` table:
  - `status = 'pending'`
  - Store attempt_id
- Enqueue RQ job with attempt_id
- Return immediately with job_id
- `GET /analysis/jobs/{id}` endpoint returns:
  ```json
  {
    "id": "...",
    "status": "pending|processing|completed|failed",
    "error_message": null,
    "created_at": "...",
    "completed_at": null
  }
  ```

**Frontend:**
- After submitting attempt, poll job status endpoint
- Show different states:
  - `pending`: "Queued for analysis..."
  - `processing`: "Analyzing your explanation..."
  - `completed`: Redirect to feedback page
  - `failed`: Show error, offer retry
- Polling interval: 2 seconds (max 60 seconds / 30 polls)
- Timeout fallback: "Analysis taking longer than expected. Check back in a few minutes."

**Technical notes:**
- Use Redis for RQ queue
- Store job metadata in both RQ and DB (DB is source of truth)
- Consider websockets instead of polling (nice-to-have)

---

### Story 6.2: Worker processes analysis

**As a developer**, I want analysis to run in a background worker so the API stays responsive.

**Acceptance criteria:**

**Worker setup:**
- RQ worker runs in separate Docker container
- Worker code in `/backend/workers/analyze_attempt.py`
- Worker consumes jobs from Redis queue
- Logs to stdout (captured by Docker)

**Analysis job logic:**
1. Fetch attempt and question from DB
2. Update job status to `processing`
3. Build prompt for OpenAI:
   - Question summary
   - User's explanation
   - Rubric criteria
4. Call OpenAI API (structured output mode)
5. Validate response schema
6. Save feedback to DB
7. Update job status to `completed`
8. On failure: Retry with exponential backoff (max 3 retries)
9. After max retries: Update job status to `failed` with error message

**Structured output schema:**
```json
{
  "rubric_scores": {
    "clarity": 1-5,
    "correctness": 1-5,
    "edge_cases": 1-5,
    "complexity_analysis": 1-5
  },
  "key_issues": ["issue 1", "issue 2", ...],
  "improved_explanation": "Corrected example...",
  "corrected_complexity": {
    "time": "O(n log n)",
    "space": "O(n)",
    "justification": "..."
  }
}
```

**Error handling:**
- OpenAI rate limit: Retry with backoff
- OpenAI error: Retry, then fail gracefully
- Invalid response: Log and mark failed
- Timeout: 30 second limit per job

**Technical notes:**
- Use OpenAI structured outputs (JSON mode) for reliability
- Store OpenAI response raw JSON for debugging
- Log job start/end with attempt_id
- Use RQ retry decorator
- Monitor worker health via health check endpoint

---

### Story 6.3: Observability and monitoring

**As a developer**, I want visibility into job processing so I can debug issues and monitor performance.

**Acceptance criteria:**

**Logging:**
- Structured JSON logs for all services (backend, worker)
- Log levels: DEBUG (dev), INFO (prod)
- Include in logs:
  - Request ID / correlation ID
  - User ID (where applicable)
  - Job ID for worker logs
  - Timestamp, service name, log level

**Metrics:**
- Backend tracks:
  - API response times (p50, p95, p99)
  - Request count per endpoint
  - Error rate
- Worker tracks:
  - Job processing time
  - Job success/failure rate
  - Queue depth
  - OpenAI API latency

**Health checks:**
- `GET /health` endpoint:
  ```json
  {
    "status": "healthy",
    "db": "connected",
    "redis": "connected"
  }
  ```
- Worker health endpoint (separate)
- Docker health checks configured

**Error tracking:**
- Sentry integration (or similar) for production
- Capture exceptions with context
- Frontend errors sent to Sentry
- Worker errors sent to Sentry

**Technical notes:**
- Use `structlog` for Python logging
- Use `winston` for frontend logging (if needed)
- Prometheus metrics (nice-to-have)
- Grafana dashboard (nice-to-have)

---

## Epic 7 — Feedback UI

### Story 7.1: View feedback report

**As a user**, I want structured feedback so I know exactly how to improve.

**Acceptance criteria:**

**Backend:**
- `GET /attempts/{id}/feedback` endpoint
  - Returns full feedback object
  - Includes attempt details and question
  - 404 if feedback not yet generated

**Frontend - Feedback page:**
- Header section:
  - Question title
  - Date/time of attempt
  - Timer value (if used)
- **Rubric scores** (visual):
  - 4 categories with 1-5 scores
  - Progress bars or star ratings
  - Overall average score (prominent)
- **Your explanation** (expandable section):
  - User's original explanation text
- **Key issues identified** (list):
  - Bulleted list of problems
  - Each item clear and actionable
- **Improved explanation** (side-by-side or below):
  - AI-generated corrected version
  - Highlight differences (nice-to-have)
- **Complexity analysis**:
  - Corrected time/space complexity
  - Justification explanation
- **Next steps** (call to action):
  - "Try again" button (new attempt for same question)
  - "Practice similar" button (suggests question with same tag)
- Confidence rating input (Story 5.2)

**Technical notes:**
- Make feedback shareable (nice-to-have: unique URL)
- Export as PDF or Markdown (nice-to-have)
- Cache feedback data (doesn't change after creation)

---

## Epic 8 — History & Progress

### Story 8.1: Attempts history

**As a user**, I want to see all my past attempts and feedback so I can track my practice.

**Acceptance criteria:**

**Backend:**
- `GET /attempts` endpoint:
  - Returns user's attempts with:
    - Attempt metadata (date, question, timer)
    - Rubric scores (if feedback exists)
    - Status (pending/completed/failed)
  - Query params:
    - `question_id` - filter by question
    - `page`, `limit` - pagination
    - `sort` - by date (default newest first)

**Frontend:**
- **All attempts page:**
  - Table/list of all attempts:
    - Question title (link to question)
    - Date/time
    - Overall score (if completed)
    - Status badge
  - Click attempt to view feedback
  - Pagination
- **Per-question attempts view:**
  - From question detail page
  - Shows attempts for that question only
  - Chart showing score progression (if multiple attempts)

**Technical notes:**
- Infinite scroll or pagination
- Cache with React Query
- Show skeleton loaders

---

### Story 8.2: Progress statistics and visualization

**As a user**, I want to see my improvement over time so I stay motivated.

**Acceptance criteria:**

**Backend:**
- `GET /stats/progress` endpoint returns:
  ```json
  {
    "total_attempts": 42,
    "avg_score_last_7": 3.8,
    "avg_score_lifetime": 3.2,
    "attempts_by_date": [
      { "date": "2025-01-15", "count": 3, "avg_score": 4.0 },
      ...
    ],
    "scores_by_tag": [
      { "tag": "dynamic-programming", "avg_score": 2.8, "count": 5 },
      ...
    ],
    "weakest_category": {
      "name": "edge_cases",
      "avg_score": 2.5
    }
  }
  ```

**Frontend - Dashboard/Stats page:**
- **Overview cards:**
  - Total attempts
  - Current streak (nice-to-have)
  - Avg score (last 7 vs lifetime)
- **Line chart:** Score over time
  - X-axis: Date
  - Y-axis: Average rubric score
  - Show trend line
- **Bar chart:** Performance by tag
  - X-axis: Tags
  - Y-axis: Average score
  - Highlight weakest tag
- **Heatmap:** Daily activity (like GitHub)
  - Calendar view of last 3-6 months
  - Color intensity = attempt count
- **Weakest category callout:**
  - "Your weakest area is **Edge Cases** (avg 2.5/5)"
  - Link to resources or practice suggestions

**Technical notes:**
- Use Recharts, Chart.js, or Victory for visualization
- Cache stats (recompute on new attempt only)
- Make charts responsive
- Export stats as CSV (nice-to-have)

---

## Epic 9 — Production Readiness

### Story 9.1: Rate limiting and abuse prevention

**As a user**, I want fair usage limits so the service is available to everyone.

**Acceptance criteria:**

**Backend:**
- Rate limiting on expensive endpoints:
  - `POST /attempts` - 10 per hour per user
  - `POST /questions` - 20 per hour per user
  - Global rate limit: 100 requests/minute per IP
- Rate limit middleware using Redis
- Response when limited:
  - HTTP 429 Too Many Requests
  - Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
  - Body: `{ "error": "Rate limit exceeded. Try again in 15 minutes." }`

**Frontend:**
- Display rate limit errors gracefully
- Show countdown timer until reset
- Prevent spam-clicking submit buttons

**Additional protections:**
- Input sanitization on all text fields
- XSS prevention (React escapes by default, but verify)
- SQL injection prevention (use ORM/parameterized queries)
- CORS configured for frontend domain only
- Content Security Policy headers

**Technical notes:**
- Use `slowapi` or custom Redis-based limiter
- Different limits for authenticated vs anonymous
- Admin override (nice-to-have)

---

### Story 9.2: End-to-end testing

**As a developer**, I want E2E tests so critical user flows are verified.

**Acceptance criteria:**

**Setup:**
- Playwright or Cypress configured
- Test environment with:
  - Seeded test database
  - Mock Supabase Auth (or test account)
  - Mock OpenAI API responses

**Test scenarios:**
1. **User signup and login**
   - Create account → login → see dashboard
2. **Create question and attempt**
   - Add question → start attempt → submit → see feedback
3. **View history**
   - See attempts list → view feedback → navigate back
4. **Daily drill flow**
   - Click daily drill → see suggestion → start attempt

**CI integration:**
- E2E tests run in GitHub Actions
- Run against Docker Compose environment
- Headless mode for CI, headed for local dev
- Video recording on failure

**Technical notes:**
- Use Playwright (faster, better assertions)
- Run in CI on every PR (separate job)
- Consider visual regression testing (nice-to-have)

---

### Story 9.3: Deployment

**As a developer**, I want the app deployed so others can use it and I can showcase it.

**Acceptance criteria:**

**Backend deployment:**
- Deploy to Railway, Render, or Fly.io
- Environment variables configured:
  - Database URL (Supabase or hosted Postgres)
  - Redis URL (Redis Cloud or Upstash)
  - OpenAI API key
  - Supabase keys
- Worker deployed as separate service
- Health checks configured
- Auto-deploy on push to main (CD)

**Frontend deployment:**
- Deploy to Vercel or Netlify
- Environment variables:
  - API base URL
  - Supabase public key
- Auto-deploy on push to main
- Custom domain (optional)

**Infrastructure:**
- Database: Supabase (free tier) or hosted Postgres
- Redis: Upstash or Redis Cloud (free tier)
- All services use HTTPS

**Monitoring:**
- Sentry error tracking configured
- Uptime monitoring (UptimeRobot or similar)
- Log aggregation (consider Papertrail or Logtail)

**Documentation:**
- `docs/DEPLOYMENT.md` with:
  - Environment variables reference
  - Deployment steps
  - Troubleshooting guide
- Live demo URL in README
- Screenshots or demo video

**Technical notes:**
- Use environment-specific configs (dev/staging/prod)
- Database backups enabled
- Consider CDN for frontend assets
- SSL/TLS for all connections

---

## Build Order (Recommended)

This order maximizes learning while keeping you unblocked:

### Phase 1: Foundation (Week 1-2)
1. Story 1.1 - Repo structure
2. Story 1.2 - Docker setup
3. Story 1.3 - Database migrations
4. Story 1.4 - Testing infrastructure
5. Story 1.5 - CI pipeline
6. Story 1.6 - API docs

**Checkpoint:** Empty app runs in Docker with CI passing

### Phase 2: Core Features (Week 3-4)
7. Epic 2 - Authentication (Stories 2.1, 2.2)
8. Epic 3 - Questions CRUD (Stories 3.1, 3.2)
9. Epic 5 - Attempt creation (Story 5.1 only, no AI yet)

**Checkpoint:** Can create questions and attempts, no feedback yet

### Phase 3: Async Pipeline (Week 5-6)
10. Epic 6 - Analysis pipeline (Stories 6.1, 6.2, 6.3)
11. Epic 7 - Feedback UI (Story 7.1)
12. Story 5.2 - Confidence rating

**Checkpoint:** End-to-end AI feedback working

### Phase 4: User Value (Week 7-8)
13. Epic 4 - Daily drill (Story 4.1)
14. Epic 8 - History and stats (Stories 8.1, 8.2)

**Checkpoint:** Full MVP feature-complete

### Phase 5: Production (Week 9-10)
15. Epic 9 - Production readiness (Stories 9.1, 9.2, 9.3)
16. Polish: Error states, loading states, responsive design
17. Documentation: README, deployment guide, demo video

**Checkpoint:** Deployed and ready to showcase

---

## Nice-to-Haves (Post-MVP)

Only tackle these after full deployment:

- [ ] Streak counter and gamification
- [ ] Export feedback as Markdown/PDF
- [ ] Question templates (pre-filled for common patterns)
- [ ] Mock interview mode (multiple questions in sequence)
- [ ] Websockets for real-time job updates (replace polling)
- [ ] Follow-up question suggestions
- [ ] Collaborative mode (practice with a peer)
- [ ] Browser extension to add questions from LeetCode
- [ ] Mobile-responsive improvements
- [ ] Dark mode
- [ ] Accessibility audit (WCAG 2.1 AA)

---

## Interview Talking Points

When discussing this project in interviews, emphasize:

1. **System Design:**
   - Async architecture with worker queue (scalability)
   - Why Redis/RQ over alternatives
   - Polling vs WebSockets trade-offs

2. **Testing:**
   - Unit, integration, E2E pyramid
   - Fixture patterns for clean test data
   - Mocking external services (OpenAI)

3. **DevOps:**
   - Docker Compose for local dev
   - CI/CD pipeline design
   - Database migration strategy
   - Deployment architecture

4. **API Design:**
   - RESTful conventions
   - Error handling and validation
   - OpenAPI documentation

5. **Frontend:**
   - State management patterns
   - Polling with timeout/fallback
   - Optimistic UI updates
   - Data visualization choices

6. **Challenges Solved:**
   - Handling OpenAI rate limits and errors
   - Keeping feedback consistent with structured outputs
   - Balancing latency vs reliability in async jobs
   - Securing user data and preventing abuse

7. **Trade-offs:**
   - RQ vs Celery (simplicity vs features)
   - Polling vs WebSockets (implementation time vs UX)
   - Monorepo vs separate repos (coordination vs independence)

---

## Success Metrics

Track these to demonstrate impact:

- [ ] All CI checks passing on main
- [ ] >80% backend test coverage
- [ ] >70% frontend test coverage
- [ ] E2E tests cover 3+ critical paths
- [ ] Deployed to production with <1% error rate
- [ ] Analysis jobs complete in <20s (p95)
- [ ] API response times <500ms (p95)
- [ ] Zero security vulnerabilities (Snyk/Dependabot)
- [ ] OpenAPI docs 100% coverage of endpoints

---

**Last updated:** 2025-01-17

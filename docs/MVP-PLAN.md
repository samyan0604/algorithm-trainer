# Algorithm Trainer - Simplified MVP Plan

**Philosophy: Build first, polish later. Learn by doing.**

This is the streamlined version focused on getting features working. You'll learn the advanced stuff (Docker, comprehensive testing, observability) by needing them, not by planning for them.

---

## Phase 1: Get Something Running (Week 1)

### 1.1: Basic repo setup

**Goal:** Run "hello world" backend and frontend locally

**Tasks:**
- [ ] Create `/frontend` and `/backend` folders
- [ ] Frontend: `npm create vite@latest` (React + TypeScript)
- [ ] Backend: Create FastAPI app with one `/health` endpoint
- [ ] Backend: `pip install fastapi uvicorn python-dotenv`
- [ ] Create `.env` file in backend (add to `.gitignore`)
- [ ] Test: Visit `http://localhost:8000/health` and `http://localhost:5173`

**Learn as you go:**
- How FastAPI works (it's just Python functions with decorators)
- How Vite dev server works
- Environment variables

**Commands to run:**
```bash
# Backend (in /backend folder)
uvicorn main:app --reload

# Frontend (in /frontend folder)
npm run dev
```

---

### 1.2: Supabase setup

**Goal:** Create database and enable auth

**Tasks:**
- [ ] Sign up for Supabase (free tier)
- [ ] Create new project
- [ ] In Supabase dashboard → Table Editor, create tables:
  - `questions` table (use GUI, add columns: id, user_id, title, leetcode_url, prompt_summary, tags, created_at)
  - `attempts` table (id, user_id, question_id, explanation_text, timer_seconds, created_at)
  - `feedbacks` table (id, attempt_id, rubric_scores, key_issues, improved_example, created_at)
- [ ] Enable Email auth in Authentication → Providers
- [ ] Copy API keys to backend `.env`:
  ```
  SUPABASE_URL=your-project-url
  SUPABASE_KEY=your-anon-key
  ```

**Learn as you go:**
- Supabase is just Postgres with a nice UI
- The GUI is easier than writing SQL migrations initially
- You can always export SQL later

**Skip for now:**
- Migrations (use Supabase GUI)
- Database indexes (add when it's slow)
- Foreign key constraints (can add later)

---

## Phase 2: Authentication (Week 1-2)

### 2.1: User can sign up and login

**Goal:** Protected routes that require login

**Backend tasks:**
- [ ] Install: `pip install supabase`
- [ ] Create Supabase client in backend
- [ ] Create `/auth/verify` endpoint that checks JWT
- [ ] Create dependency `get_current_user()` to extract user from JWT
- [ ] Test with Postman/Bruno by passing Authorization header

**Frontend tasks:**
- [ ] Install: `npm install @supabase/supabase-js`
- [ ] Create Supabase client (use env variables)
- [ ] Create signup page with email/password form
- [ ] Create login page
- [ ] Store session in localStorage (Supabase does this automatically)
- [ ] Create `AuthContext` to share user state
- [ ] Add logout button

**Learn as you go:**
- How JWTs work (they're just encoded JSON)
- React Context for global state
- Form handling in React

**Skip for now:**
- Token refresh (Supabase handles it)
- Social login (Google, GitHub)
- Email verification
- Password reset

**Example code structure:**
```
frontend/src/
  contexts/
    AuthContext.tsx    # User state
  pages/
    Login.tsx
    Signup.tsx
  lib/
    supabase.ts        # Supabase client
```

---

## Phase 3: Questions CRUD (Week 2-3)

### 3.1: Create and view questions

**Goal:** Add LeetCode questions to practice later

**Backend tasks:**
- [ ] `POST /questions` endpoint
  - Takes: title, leetcode_url, prompt_summary, tags (optional)
  - Validates: URL contains "leetcode.com"
  - Saves to Supabase questions table
  - Returns created question
- [ ] `GET /questions` endpoint
  - Returns all questions for logged-in user
  - Include attempt count (SQL JOIN or separate query)

**Frontend tasks:**
- [ ] Create "Add Question" page with form
- [ ] Create "Questions List" page
- [ ] Install: `npm install @tanstack/react-query` for data fetching
- [ ] Show: title, attempt count, last attempted date
- [ ] Click question → navigate to attempt page

**Learn as you go:**
- React Router for navigation
- React Query for API calls (handles loading/error states)
- Basic form validation

**Skip for now:**
- Edit/delete questions
- Pagination (add when you have 50+ questions)
- Search/filter (add when needed)
- Tag autocomplete

**Simple validation:**
```python
# Backend
if "leetcode.com" not in leetcode_url:
    raise HTTPException(400, "Invalid LeetCode URL")
```

---

## Phase 4: Attempt Creation (Week 3-4)

### 4.1: Write explanation (no AI feedback yet)

**Goal:** Save your explanation to the database

**Backend tasks:**
- [ ] `POST /attempts` endpoint
  - Takes: question_id, explanation_text, timer_seconds
  - Saves to attempts table
  - Returns attempt object
- [ ] `GET /attempts/{id}` endpoint

**Frontend tasks:**
- [ ] Create attempt page for a question
- [ ] Show question title and LeetCode link
- [ ] Show template prompts (just static text):
  ```
  1. Problem Summary
  2. Approach & Algorithm
  3. Step-by-Step Explanation
  4. Time Complexity
  5. Space Complexity
  6. Edge Cases
  ```
- [ ] Large textarea for explanation
- [ ] Optional timer (use `useState` + `setInterval`)
  - Toggle on/off
  - Preset buttons: 20 min, 45 min
  - Alert when done (just `alert()` for now)
- [ ] Submit button → saves attempt

**Learn as you go:**
- Managing timer state in React
- Calling API on form submit
- Redirecting after successful submit

**Skip for now:**
- Draft saving (localStorage)
- Auto-save
- Fancy timer UI
- Character limit warnings

---

## Phase 5: AI Feedback (Week 4-6)

**This is the big learning phase - async jobs and AI integration**

### 5.1: OpenAI integration (synchronous first)

**Goal:** Get AI feedback working, even if it's slow

**Backend tasks:**
- [ ] Sign up for OpenAI, get API key
- [ ] Add to `.env`: `OPENAI_API_KEY=sk-...`
- [ ] Install: `pip install openai`
- [ ] Create function `analyze_explanation()`:
  - Takes attempt_id
  - Fetches attempt + question from DB
  - Builds prompt with rubric criteria
  - Calls OpenAI API (use JSON mode)
  - Saves response to feedbacks table
- [ ] Call this function directly in `POST /attempts` endpoint
- [ ] Return feedback immediately (will be slow, 10-20 seconds)

**Learn as you go:**
- OpenAI API basics
- Prompt engineering (how to get consistent output)
- JSON mode for structured responses

**OpenAI prompt structure:**
```python
prompt = f"""
You are grading an algorithm explanation.

Question: {question.title}
Student explanation: {attempt.explanation_text}

Grade on these criteria (1-5 scale):
- Clarity: Is it easy to follow?
- Correctness: Is the logic right?
- Edge cases: Did they consider edge cases?
- Complexity: Did they correctly analyze time/space?

Return JSON:
{{
  "rubric_scores": {{"clarity": 4, "correctness": 5, ...}},
  "key_issues": ["Issue 1", "Issue 2"],
  "improved_explanation": "...",
  "corrected_complexity": {{"time": "O(n)", "space": "O(1)"}}
}}
"""
```

**Skip for now:**
- Async jobs (do this next)
- Retry logic
- Error handling (just let it fail for now)

---

### 5.2: Make it async with Redis + RQ

**Goal:** Don't block the API while OpenAI processes

**Why you need this:**
- The synchronous version from 5.1 will timeout if OpenAI is slow
- Users can't do anything while waiting
- This is how real apps handle long tasks

**Backend tasks:**
- [ ] Install Redis locally: `brew install redis` (Mac) or Docker
- [ ] Start Redis: `redis-server`
- [ ] Install: `pip install rq`
- [ ] Create `/backend/workers/analyze_job.py`:
  - Move `analyze_explanation()` function here
  - This is the "worker" code
- [ ] Modify `POST /attempts`:
  - Instead of calling function directly, queue a job:
    ```python
    from rq import Queue
    from redis import Redis

    redis_conn = Redis()
    queue = Queue(connection=redis_conn)
    job = queue.enqueue('workers.analyze_job.analyze_explanation', attempt_id)
    ```
  - Return immediately with `job_id`
- [ ] Create `GET /jobs/{job_id}` endpoint to check status
- [ ] Start worker in separate terminal:
  ```bash
  rq worker --with-scheduler
  ```

**Frontend tasks:**
- [ ] After submitting attempt, get `job_id` from response
- [ ] Poll `GET /jobs/{job_id}` every 3 seconds
- [ ] Show states:
  - "Queued..." (job not started)
  - "Analyzing..." (job running)
  - "Complete!" → redirect to feedback page
  - "Failed" → show error

**Learn as you go:**
- What a job queue is (Redis is just a fast database)
- How workers process jobs (it's a separate Python process)
- Polling (checking status repeatedly)

**This is your key differentiator - make sure you understand it!**

---

## Phase 6: Feedback UI (Week 6-7)

### 6.1: Display feedback

**Goal:** Show the AI analysis in a nice format

**Backend tasks:**
- [ ] `GET /attempts/{id}/feedback` endpoint
  - Returns feedback object for an attempt
  - Include attempt details and question

**Frontend tasks:**
- [ ] Create feedback page
- [ ] Show rubric scores (use progress bars or simple numbers)
- [ ] Show key issues as bullet list
- [ ] Show improved explanation
- [ ] Show corrected complexity
- [ ] Add "Try Again" button (new attempt for same question)

**Learn as you go:**
- Layout design
- Conditional rendering (if feedback exists)
- CSS for progress bars

**Keep it simple:**
- Plain HTML + CSS first
- Can use component library later (shadcn/ui, MUI)

---

## Phase 7: History & Stats (Week 7-8)

### 7.1: View past attempts

**Backend tasks:**
- [ ] `GET /attempts` endpoint
  - Returns all attempts for user
  - Include question info and scores

**Frontend tasks:**
- [ ] Create history page
- [ ] Show table of attempts:
  - Date
  - Question title (clickable)
  - Overall score
- [ ] Click to view feedback

---

### 7.2: Basic stats

**Backend tasks:**
- [ ] `GET /stats` endpoint returns:
  ```json
  {
    "total_attempts": 10,
    "avg_score_last_7": 3.5,
    "avg_score_lifetime": 3.2
  }
  ```

**Frontend tasks:**
- [ ] Show stats on dashboard
- [ ] Simple cards with numbers (no charts yet)

**Skip for now:**
- Charts/graphs (add in polish phase)
- Tag analysis
- Trend lines

---

## Phase 8: Daily Drill (Week 8)

### 8.1: Suggest a question

**Backend tasks:**
- [ ] `GET /suggestions/daily` endpoint
- [ ] Logic: Return question with fewest attempts
- [ ] Tie-break by random

**Frontend tasks:**
- [ ] Dashboard has "Start Daily Drill" button
- [ ] Shows suggested question
- [ ] Click → start attempt

---

## Phase 9: Deploy MVP (Week 9)

### 9.1: Get it live

**Backend deployment:**
- [ ] Sign up for Render or Railway (free tier)
- [ ] Create new Web Service
- [ ] Connect GitHub repo
- [ ] Set environment variables (Supabase keys, OpenAI key)
- [ ] Deploy (they auto-detect FastAPI)
- [ ] Add Redis addon (Render/Railway provide this)
- [ ] Deploy worker as separate service (same code, different start command)

**Frontend deployment:**
- [ ] Sign up for Vercel or Netlify
- [ ] Connect GitHub repo
- [ ] Set env variable: API URL (your backend URL)
- [ ] Deploy (auto-detects Vite)

**Test:**
- [ ] Visit live URL
- [ ] Sign up, create question, submit attempt
- [ ] Verify feedback works

**Learn as you go:**
- How deployment platforms work
- Environment variables in production
- CORS (you'll get errors and fix them)

---

## Phase 10: Polish (Week 10)

**Now go back and add the "production" features:**

- [ ] Add basic tests (just a few for practice)
- [ ] Add error handling (try/catch, show user-friendly errors)
- [ ] Add loading states (spinners, skeletons)
- [ ] Make it responsive (mobile-friendly)
- [ ] Add simple visualization (one chart on stats page)
- [ ] Write good README with screenshots
- [ ] Record demo video (Loom, 2-3 minutes)

---

## What to Add Later (After MVP Works)

Once you have the MVP deployed and understand how everything works, **then** go back to the full EPICS.md and add:

1. **Docker** (Week 11) - When you're tired of "works on my machine"
2. **Database migrations** (Week 11) - When Supabase GUI gets annoying
3. **Comprehensive tests** (Week 12) - When you want to refactor confidently
4. **CI/CD** (Week 12) - When you want automatic checks
5. **Monitoring** (Week 13) - When you need to debug production
6. **E2E tests** (Week 13) - When you want to prevent regressions

**You'll appreciate these features more after feeling the pain of not having them.**

---

## File Structure (Keep It Simple)

```
algorithm-trainer/
├── backend/
│   ├── main.py              # FastAPI app
│   ├── database.py          # Supabase client
│   ├── auth.py              # Auth helpers
│   ├── routers/
│   │   ├── questions.py     # Question endpoints
│   │   ├── attempts.py      # Attempt endpoints
│   │   └── stats.py         # Stats endpoints
│   ├── workers/
│   │   └── analyze_job.py   # RQ worker
│   ├── requirements.txt
│   └── .env
│
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   ├── Login.tsx
│   │   │   ├── Signup.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Questions.tsx
│   │   │   ├── AddQuestion.tsx
│   │   │   ├── Attempt.tsx
│   │   │   ├── Feedback.tsx
│   │   │   └── History.tsx
│   │   ├── components/      # Reusable components
│   │   ├── contexts/
│   │   │   └── AuthContext.tsx
│   │   ├── lib/
│   │   │   ├── supabase.ts
│   │   │   └── api.ts       # API calls
│   │   └── App.tsx
│   ├── package.json
│   └── .env.local
│
├── docs/
│   ├── PRD.md
│   ├── EPICS.md             # Full version
│   └── MVP-PLAN.md          # This file
│
└── README.md
```

---

## Learning Resources (Use When Stuck)

**FastAPI:**
- Official tutorial: https://fastapi.tiangolo.com/tutorial/
- Watch: "FastAPI Tutorial" by Tech With Tim (YouTube)

**React + TypeScript:**
- React docs: https://react.dev
- TypeScript in 5 minutes: https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html

**Supabase:**
- Quickstart: https://supabase.com/docs/guides/getting-started
- Watch: "Supabase Crash Course" (YouTube)

**Redis + RQ:**
- RQ docs: https://python-rq.org/
- Concept: "What is a job queue?" (Google this)

**OpenAI API:**
- Quickstart: https://platform.openai.com/docs/quickstart
- JSON mode: https://platform.openai.com/docs/guides/structured-outputs

**Deployment:**
- Render docs: https://render.com/docs
- Vercel docs: https://vercel.com/docs

---

## How to Use This Document

1. **Work phase by phase** - Don't jump ahead
2. **Check off tasks** as you complete them
3. **Add notes** when you learn something important
4. **Modify freely** - This is your plan
5. **When stuck** - Google the error, ask ChatGPT/Claude, read docs
6. **When done with MVP** - Revisit full EPICS.md for production features

---

## Success = Deployed MVP

**Your goal: Get a working app live that:**
- ✅ Users can sign up and log in
- ✅ Add LeetCode questions
- ✅ Write explanations
- ✅ Get AI feedback (async with RQ)
- ✅ View history
- ✅ Deployed and accessible via URL

**Everything else is polish.** Get this working first, then make it production-grade.

---

**Start with Phase 1. Good luck! 🚀**

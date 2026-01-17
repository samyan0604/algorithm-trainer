# Product Requirements Document (PRD)

## 1) One Sentence Summary

A web app that helps LeetCode-focused students and interview candidates practise explaining algorithm solutions in clear, structured language, then uses an AI rubric to give targeted feedback on correctness, edge cases, and communication quality.

## 2) Problem and Motivation

Many LeetCode practisers can write correct code but struggle to explain their reasoning under pressure. Common issues include:

- Blanking when asked to talk
- Giving disorganised explanations
- Being unable to justify time/space complexity clearly

General chatbots can help, but they do not:

- Enforce interview-style structure
- Provide consistent rubric scoring
- Track progress over time

This product provides a deliberate practice loop: **guided explanation → rubric-based AI feedback → measurable improvement**.

## 3) Target Users and Use Cases

**Primary users:** Intermediate LeetCode practisers (mostly Medium problems), with secondary support for beginners and advanced users.

**Primary use case:** Practice explaining a solution without coding, using a guided template, for daily drills and pre-interview preparation.

**Question source (v1):** User pastes a LeetCode link; app stores the link and a short user-written summary prompt (no scraping or redistributing full statements).

**Collaboration:** Solo practice in v1.

## 4) Goals and Non-Goals

### Goals (v1)

- Improve the structure and clarity of algorithm explanations
- Improve ability to state and justify time and space complexity
- Improve correctness by identifying missing reasoning and edge cases via rubric feedback

### Non-Goals (v1)

- No code execution / online judge
- No PvP or ranked modes
- No scraping or redistributing full LeetCode statements
- No speech-to-text
- No advanced ML personalisation
- No mobile app release

### Engineering Priority (Interview Focus)

Deliver a clean full-stack system with clear API boundaries and a maintainable architecture (React frontend, FastAPI backend, Postgres persistence).

## 5) MVP Scope

### Must Have

**Auth:** Required

**Add/Select Question:**

- LeetCode link + short user-written prompt summary + optional tags

**Attempt Page:**

- One large text input
- Visible guided structure prompts
- Optional timer with presets (Daily Drill, Mock Interview)

**AI Analysis (server-side)** returning structured feedback:

- Rubric scores: clarity, correctness, edge cases, complexity
- Key issues
- Improved explanation
- Corrected time/space complexity

**Feedback Page:** Renders rubric and actionable suggestions

**History Page:** Attempts per question + basic progress stats

### Nice to Have (only if time)

- Simple streak counter
- Export feedback as Markdown

## 6) User Journey and Screens

1. **Login**
2. **Dashboard** with two actions:
   - Start Daily Drill
   - Questions List
3. **Daily Drill** suggests least practised / weakest tag question
4. **Attempt** (guided prompts + optional timer)
5. **Feedback** (rubric + improvements)
6. **History/Stats** (trends + weakest categories)

## 7) Success Metrics

**Active day:** User completes at least one attempt

**Learning:**

- Rolling average rubric score improves over time
- Confidence self-rating trend improves (confidence asked after feedback)

**System:**

- AI feedback completes within 20 seconds (v1 target)
- High analysis success rate

## 8) Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| **Cost** | Daily attempt limit per user + caching |
| **AI inconsistency** | Structured outputs + schema validation + retry/fallback |
| **Missing context** | AI gives feedback but flags low confidence and requests missing details |
| **LeetCode ToS** | Link out + user-written short prompt only |
| **Abuse** | Server-side calls, rate limiting, basic prompt injection hardening |
| **Motivation drop-off** | Daily drill entry point + progress tracking |

## 9) Roadmap

- **C)** Streaks and light gamification
- **E)** Mock interview follow-up question mode
- **A)** Speech-to-text for verbal practice
- **G)** Mobile app focused on voice drills

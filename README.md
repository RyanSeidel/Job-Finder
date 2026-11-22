# Job-Finder
an application made for tracking jobs, reading emails, using an AI Agent, and motivates user to keep applying

## Features (planned)
- **Job tracker**: a Kanban board and table of applications (saved → applied → interviewing → offer / rejected / ghosted)
- **Gmail integration**: reads job-related emails (read-only) and links them to applications
- **AI agent** (runs locally on your PC for free via Ollama):
  - Classifies emails (confirmations, rejections, interview requests, offers, recruiter outreach) and updates the tracker automatically
  - Finds new job leads and scores how well they match your resume
  - Tailors cover letters and suggests resume tweaks for each job
  - Coaches you: interview prep, follow-up reminders, encouragement after rejections
- **Motivation**: daily and weekly goals, streaks, badges, morning summaries and evening reminders

## Tech Stack
| Layer | Choice | Why |
|---|---|---|
| Desktop shell | **Electron** + electron-vite | All TypeScript; packages to a Windows .exe |
| UI | **React + TypeScript + Tailwind + shadcn/ui** | Fast to build, looks good |
| Local DB | **SQLite** via `better-sqlite3` + **Drizzle ORM** | One local file, no server, private |
| Email | **Gmail API** (`googleapis`), OAuth 2.0 desktop flow | Read-only scope `gmail.readonly` |
| AI | **Ollama** (`ollama` npm package) running a small open model (e.g. Llama or Qwen, 7–8B) | Free, offline, private; supports JSON output and tool calling |
| Job search | **Adzuna API** + public **Greenhouse/Lever** job-board endpoints | Legal and stable; no scraping of LinkedIn or Indeed |
| Secrets | Electron `safeStorage` (Windows DPAPI) | Tokens and API keys are never stored in plaintext |
| Notifications | Electron `Notification` + `node-cron` | Reminders and nudges |

## Architecture
```
Renderer (React UI)  ──IPC──>  Main process (Node)
                                 ├─ db/        SQLite + Drizzle schema
                                 ├─ gmail/     OAuth + sync
                                 ├─ ai/        Ollama client, agent + tools
                                 ├─ jobs/      Adzuna / Greenhouse / Lever fetchers
                                 └─ scheduler/ cron: email sync, daily goals, nudges
```
API keys and tokens stay in the main process. The UI talks to it through a typed `preload` bridge.

## Data Model
- `applications`: company, role, url, status, dates, salary, notes
- `emails`: gmail_id, thread_id, from, subject, snippet, received_at, classification, application_id
- `job_leads`: source, title, company, location, url, description, match_score, dismissed
- `documents`: base resume, generated cover letters and tailored resumes
- `activity_log` + `goals`: daily and weekly targets, streaks, stats
- `settings`

## Roadmap
Each phase leaves you with a usable app.

### Phase 1: Skeleton + manual tracker
- [ ] Scaffold with `npm create @quick-start/electron` (React + TS), add Tailwind, shadcn, Drizzle, SQLite
- [ ] Kanban board (drag between statuses), table view, add/edit form, search and filter
- [ ] Dashboard: counts by status, applications per week

### Phase 2: Gmail integration
- [ ] Google Cloud project, Gmail API enabled, OAuth client of type "Desktop app"
- [ ] OAuth via the system browser and a localhost redirect; refresh token stored with `safeStorage`
- [ ] Initial sync (`newer_than:90d` plus job keywords), then incremental `historyId` sync every 15 minutes
- [ ] Inbox view

### Phase 3: AI email classification
- [ ] Local model with structured JSON output: `{category, company, role, confidence}`
- [ ] Keyword rules run first (e.g. "unfortunately", "interview", "thank you for applying") so the AI only handles emails the rules can't classify
- [ ] Match each email to an application (or create one) and advance its status; ask you to confirm when confidence is low
- [ ] Notify on interview requests

### Phase 4: Job finding + matching
- [ ] Resume upload (PDF parsed with `pdf-parse`) and job preferences (titles, location, remote, salary)
- [ ] Daily leads from Adzuna and Greenhouse/Lever boards
- [ ] AI match score (0–100) with a short "why it fits"; save, dismiss, or apply from the feed

### Phase 5: Resume & cover letter tailoring
- [ ] Generate a cover letter and resume suggestions per application
- [ ] In-app editing; export to .docx or PDF

### Phase 6: Coach agent + motivation
- [ ] Chat agent with tools: `list_applications`, `get_stats`, `search_jobs`, `draft_cover_letter`, `set_goal`, `prep_interview`
- [ ] Goals, streaks, progress ring, badges
- [ ] Morning summary, evening reminder, follow-up prompts after 10 days with no reply
- [ ] Encouragement and a suggested next step after rejections

### Phase 7: Polish & ship
- [ ] Settings page and first-run onboarding wizard
- [ ] Windows installer with `electron-builder`; optional auto-update via GitHub Releases

## Setup Requirements
1. **Ollama**: install from [ollama.com](https://ollama.com), then pull a model, e.g. `ollama pull llama3.1:8b` (free)
2. **Google Cloud** project with the Gmail API enabled and a Desktop OAuth client ID (free for personal use)
3. **Adzuna** app ID and key: [developer.adzuna.com](https://developer.adzuna.com) (free)

**Hardware:** a 7–8B model needs about 8 GB of free RAM. A GPU makes it faster, but it isn't required.
Target dev machine: **RTX 3080 (10 GB VRAM)**. A 4-bit 7–8B model fits fully in VRAM (fast); models up to about 12–14B also fit at 4-bit with a smaller context.

## Cost
**$0.** Every tool and API used here is free for personal use, and the AI runs on your own computer.

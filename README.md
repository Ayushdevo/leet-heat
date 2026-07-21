# Leet Heat

Developer analytics platform for tracking coding performance and growth — log your solves, watch your streak, and see a GitHub-style contribution heatmap that literally gets *hotter* the more you code.

**Stack:** React (Vite) · FastAPI · PostgreSQL

## Features (MVP)

- **Auth** — JWT-based register/login
- **Submission tracking** — log solved problems with difficulty, status, language, runtime
- **Streak engine** — current streak, longest streak, computed server-side from solve dates
- **Heatmap** — 365-day activity calendar, color-coded from "cold" (no activity) to "hot" (peak activity) — the app's signature visual
- **Stats overview** — total solved, difficulty breakdown, last-30-days activity

## Project structure

```
leet-heat/
├── backend/          FastAPI app
│   └── app/
│       ├── main.py          entrypoint, CORS, router registration
│       ├── models.py        SQLAlchemy models (User, Problem, Submission)
│       ├── schemas.py       Pydantic request/response schemas
│       ├── database.py      engine/session setup
│       ├── core/
│       │   ├── config.py    settings (reads .env)
│       │   └── security.py  password hashing, JWT
│       └── routers/
│           ├── auth.py         /auth/register, /auth/login
│           ├── problems.py     /problems
│           ├── submissions.py  /submissions
│           └── stats.py        /stats/overview, /stats/heatmap  ← streak + heatmap logic
├── frontend/         React (Vite) app
│   └── src/
│       ├── pages/           Login, Register, Dashboard
│       ├── components/      Heatmap, StatCard, SubmissionForm, SubmissionList
│       └── api/client.js    axios instance with auth interceptor
└── docker-compose.yml  local Postgres
```

## Getting started

### 1. Database

```bash
docker compose up -d
```

This starts Postgres on `localhost:5432` with db `leetheat` / user `leetheat` / password `leetheat`.

### 2. Backend

```bash
cd backend
python -m venv venv && source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload
```

API runs at `http://localhost:8000`. Tables are auto-created on startup (no Alembic migration needed for the MVP — swap to `alembic upgrade head` once you add migrations).

Interactive API docs: `http://localhost:8000/docs`

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

App runs at `http://localhost:5173`.

## API quick reference

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | `/auth/register` | – | Create account, returns token |
| POST | `/auth/login` | – | Returns token |
| POST | `/submissions` | ✓ | Log a solve (creates problem inline if new) |
| GET | `/submissions` | ✓ | List recent submissions |
| GET | `/stats/overview` | ✓ | Streak, difficulty breakdown, totals |
| GET | `/stats/heatmap?days=365` | ✓ | Per-day solve counts for the heatmap |
| GET | `/problems` | ✓ | List tracked problems |

## Notes / next steps

- **Streak logic** lives in `app/routers/stats.py::_compute_streaks` — it's pure and unit-testable in isolation if you want to add tests.
- **LeetCode API integration**: there's no official public LeetCode API. If you want auto-sync instead of manual logging, the common approach is scraping the unofficial GraphQL endpoint (`leetcode.com/graphql`) — worth a spike before demo day if that's a stretch goal.
- **Auth** currently uses a single long-lived JWT (7-day expiry) with no refresh flow — fine for a hackathon demo, swap for refresh tokens before any real deployment.
- Password hashing uses `bcrypt==4.0.1` pinned explicitly — newer bcrypt releases break `passlib`'s version detection, so don't bump that dependency without testing.

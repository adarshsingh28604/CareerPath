# CareerPath

CareerPath is an AI-assisted career platform. Upload a resume and get an analysis, job matching, an application tracker, interview preparation, and (on supported portals) browser-based application auto-fill with optional saved portal sessions.

The repository is a monorepo with a **FastAPI** backend and a **Next.js (App Router)** frontend, orchestrated with **Docker Compose**.

<!-- Add screenshots or a demo GIF here, e.g.
![Dashboard](docs/dashboard.png)
-->

## Features

- **Resume parsing:** upload a PDF or text resume and get a structured profile stored in PostgreSQL
- **Career analysis:** overview, skills gap, and career path, with streaming chat and history
- **ATS-style scoring:** score a resume against a pasted job description
- **Job recommendations:** role match scoring and recommended jobs
- **Application tracker:** CRUD and stats with a Kanban-style UI
- **Auto-fill:** async browser automation for supported job portals, with progress polling and screenshots
- **Portal sessions:** import or log in once so automation can reuse the signed-in state
- **Interview prep:** question flow and answer evaluation
- **Auth:** optional JWT authentication

## Tech stack

| Area          | Technology                                                           |
| ------------- | -------------------------------------------------------------------- |
| Backend       | FastAPI (Python 3.13), Uvicorn, LangChain / LangGraph, Google Gemini |
| Data          | PostgreSQL 16 + pgvector, SQLAlchemy                                 |
| Automation    | Playwright (Chromium), Gemini Vision for form interaction            |
| Jobs / search | Tavily, optional JSearch (RapidAPI)                                  |
| Frontend      | Next.js 15, React 19, TanStack Query, shadcn/ui, Tailwind CSS 4      |
| Tooling       | Docker Compose, `uv` (Python), `ruff`                                |

## Architecture

```
Next.js frontend (3000)
        |
        v
FastAPI backend (8000)
   |-- LangChain / LangGraph agents  --> Google Gemini
   |-- RAG / embeddings              --> PostgreSQL + pgvector (5432)
   |-- Playwright automation         --> job portals (Gemini Vision)
   '-- Job search                    --> Tavily / JSearch
```

## Repository layout

- `backend/`: API, agents, RAG/embeddings, Playwright automation, database layer
- `frontend/`: Next.js UI (dashboard, resume flow, jobs, applications, interview prep)
- `compose.yaml`: root Compose file that includes `backend/compose.yaml` and `frontend/compose.yaml`

## Prerequisites

- **Docker (full stack):** Docker Compose v2 (e.g. Docker Desktop)
- **Backend (local):** Python 3.13 and [`uv`](https://docs.astral.sh/uv/)
- **Frontend (local):** Node.js 20+
- **API keys:** `GOOGLE_API_KEY` (Gemini) and `TAVILY_API_KEY`

## Quickstart with Docker

1. Create the env file (Compose requires it) and add your keys:

   ```bash
   cp backend/.env.example backend/.env.local
   ```

   On Windows (PowerShell):

   ```powershell
   Copy-Item backend\.env.example backend\.env.local
   ```

2. Start everything from the repository root:

   ```bash
   docker compose up --build
   ```

| Service                    | Port |
| -------------------------- | ---- |
| PostgreSQL (pgvector)      | 5432 |
| FastAPI                    | 8000 |
| Next.js (production image) | 3000 |

A one-off `db-init` job prepares the database schema and exits when finished.

- App: http://localhost:3000
- API docs (OpenAPI): http://localhost:8000/docs

## Local development

**Backend**

```bash
cd backend
uv sync
uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 --app-dir src
```

For routes that use browser automation, install Chromium once:

```bash
uv run playwright install chromium
```

**Frontend**

```bash
cd frontend
npm install
npm run dev
```

The frontend defaults `NEXT_PUBLIC_API_URL` to `http://localhost:8000` (see `frontend/src/lib/config.ts`). Override it if your API runs elsewhere.

## Environment variables

Configure `backend/.env.local` (never commit secrets; the file is gitignored).

| Variable          | Required | Description                                              |
| ----------------- | -------- | -------------------------------------------------------- |
| `GOOGLE_API_KEY`  | Yes      | Google Gemini API key                                    |
| `TAVILY_API_KEY`  | Yes      | Tavily search API key                                    |
| `DATABASE_URL`    | No       | SQLAlchemy URL, only when not using the Compose database |
| `MODEL_NAME`      | No       | Gemini model, e.g. `gemini-2.5-flash`                    |
| `JSEARCH_API_KEY` | No       | Optional RapidAPI JSearch integration                    |

See `backend/.env.example` for placeholders, and `backend/README.md` for model options and troubleshooting.

## Notes

- The default Postgres credentials in Compose are `postgres` / `postgres`. Fine for local demos, not for public deployments.
- CORS is permissive by default. Tighten `CORS_ORIGINS` for any shared or production host.
- Auto-fill drives a headless browser against third-party sites. Terms of use and consent are the operator's responsibility.

## More docs

- `backend/README.md`: backend env, run commands, linting
- `frontend/README.md`: Next.js dev server notes
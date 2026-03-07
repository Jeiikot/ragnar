# Ragnar

RAG-based code analysis tool. Users upload source code (ZIP) or PDF documents, index them into ChromaDB, and ask natural-language questions via a LangChain RAG pipeline.

## Tech Stack

- **Backend:** FastAPI + Python 3.11+ + LangChain + ChromaDB + pydantic-settings
- **Frontend:** React 19 + TypeScript 5.9 + Vite 7 + Tailwind CSS 4
- **Infrastructure:** Docker Compose (Ollama + Backend + Frontend)
- **LLM Providers:** OpenAI, Ollama, HuggingFace (auto-selection: Ollama > OpenAI > HuggingFace)

## Project Structure

```
ragnar/
├── backend/          # FastAPI REST API (DDD-lite architecture)
│   ├── api/          # Presentation layer (routers, schemas, dependencies)
│   ├── application/  # Use cases (indexing service)
│   ├── domain/       # Entities + Protocol-based ports
│   ├── infrastructure/ # Adapters (providers, chat engine, indexing, retriever)
│   ├── shared/       # Configuration (pydantic-settings)
│   └── tests/        # unit/, integration/, e2e/
├── frontend/         # React SPA
│   ├── src/          # Components, hooks, API client
│   └── tests/        # unit/, integration/, e2e/
├── .claude/
│   ├── agents/       # 8 specialized subagents
│   └── skills/       # 6 reusable skills (auto-loaded on demand)
└── docker-compose.yml
```

## Quick Start

```bash
# Full stack with Docker Compose (from root)
docker compose up --build

# Backend only (from backend/)
uv sync --extra dev
uv run uvicorn api.main:app --host 0.0.0.0 --port 8765 --reload

# Frontend only (from frontend/)
npm install
npm run dev
```

## Running Tests

```bash
# Backend
cd backend
uv run pytest -q                          # unit tests
uv run pytest -m integration              # integration tests
uv run pytest -m e2e                      # e2e tests
uv run ruff check .                       # linting

# Frontend
cd frontend
npm test                                  # unit + integration (vitest)
npm run test:e2e                          # e2e (playwright)
npm run lint                              # eslint
npm run build                             # typecheck + build
```

## Architecture

The backend follows **DDD-lite (Ports & Adapters)**:
- **Domain** defines `Protocol`-based ports and frozen dataclass entities
- **Infrastructure** provides concrete adapters, wired via `functools.partial` in `adapters.py`
- **Application** orchestrates use cases through ports; imports only from `domain` and `shared`
- **API** is the composition root: `api/dependencies.py` builds port bundles and injects them via `Depends()`

See the `ddd-architecture` skill for full layer rules and data flows.

## Key Conventions

- **Never add `Co-Authored-By: Claude` to commit messages** in any of the three repos
- API schemas mirror between `backend/api/schemas/` and `frontend/src/api/client.ts`

See skills for detailed conventions: `python-conventions` (backend), `react-conventions` + `design-system` (frontend), `conventional-commits` + `git-flow` (version control).

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/index/code` | Upload and index a source code ZIP |
| POST | `/api/v1/index/documents` | Upload and index PDFs or ZIP of PDFs |
| GET | `/api/v1/index/status` | Get indexed sources for a session |
| POST | `/api/v1/index/clear` | Clear indexed data for a session |
| POST | `/api/v1/chat` | Ask a question about indexed content |
| GET | `/api/v1/health` | Health check |

## Version Control

Backend and frontend each have their own independent git repository with Git Flow. Changelog generation handled by `git-cliff` using `cliff.toml` in each sub-project root.

- `backend/` — git repo with Git Flow (`main`, `develop`)
- `frontend/` — git repo with Git Flow (`main`, `develop`)

## Sub-Agents

Specialized agents in `.claude/agents/`:
- **architect** — Read-only architecture advisor (no file modifications)
- **backend** — Python/FastAPI specialist (reads and writes backend code)
- **frontend** — React/TypeScript specialist (reads and writes frontend code)
- **ux-ui** — UX/UI design specialist (accessibility, layout, visual hierarchy)
- **docs-sync** — Keeps agent context files and CLAUDE.md accurate after structural changes
- **commit-backend** — Git commit specialist for backend; runs ruff + mypy, creates Conventional Commits
- **commit-frontend** — Git commit specialist for frontend; runs ESLint + TypeScript typecheck
- **commit-root** — Git commit specialist for root repo; handles submodule pointer updates

## Skills

Reusable skills in `.claude/skills/` — auto-loaded when relevant to your request:

| Skill | When it activates |
|-------|------------------|
| `python-conventions` | Writing or reviewing Python/backend code |
| `ddd-architecture` | Discussing architecture, adding features, reviewing layer structure |
| `react-conventions` | Writing or reviewing React/TypeScript/frontend code |
| `design-system` | Designing or reviewing UI components, layout, accessibility |
| `conventional-commits` | Creating commits or reviewing commit messages |
| `git-flow` | Creating branches, merging, or discussing release workflow |

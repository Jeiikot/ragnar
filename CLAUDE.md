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
- **Application** orchestrates use cases through ports; imports only from `domain` and `shared` — port bundles are injected as arguments, never constructed inside services
- **API** is the composition root: `api/dependencies.py` builds port bundles via `build_indexing_ports`/`build_document_ports` from `infrastructure.indexing.adapters` and injects them via `Depends()`; routers delegate to application services

## Key Conventions

- All Python files: `from __future__ import annotations`
- Domain ports: `typing.Protocol` (never ABC)
- Domain entities: `@dataclass(frozen=True)`
- Request schemas: `ConfigDict(extra="forbid")`
- Python line length: 99 (ruff)
- Frontend: function components only, Tailwind utilities in JSX, UI text in Spanish
- API schemas mirror between `backend/api/schemas/` and `frontend/src/api/client.ts`

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

Backend and frontend each have their own independent git repository with Git Flow initialized (branches: `main` and `develop`). Changelog generation is handled by `git-cliff` using a `cliff.toml` config in each sub-project root; the generated `CHANGELOG.md` is updated during release branches.

- `backend/` — git repo with Git Flow (`main`, `develop`)
- `frontend/` — git repo with Git Flow (`main`, `develop`)

## Sub-Agents

Specialized agents are defined in `.claude/agents/`:
- **architect** — Read-only architecture advisor (no file modifications)
- **backend** — Python/FastAPI specialist (reads and writes backend code)
- **frontend** — React/TypeScript specialist (reads and writes frontend code)
- **ux-ui** — UX/UI design specialist (accessibility, layout, visual hierarchy)
- **docs-sync** — Keeps agent context files and CLAUDE.md accurate after structural changes; invoke after renaming/moving files, refactoring layers, adding endpoints, or restructuring modules
- **commit-backend** — Git commit specialist for backend; runs ruff + mypy, creates Conventional Commits following Git Flow
- **commit-frontend** — Git commit specialist for frontend; runs ESLint + TypeScript typecheck, creates Conventional Commits following Git Flow

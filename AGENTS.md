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
- **Application** orchestrates use cases through ports (no infrastructure imports)
- **API** is a thin FastAPI shell delegating to services/engines

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

## Specialized Context Files

There are detailed role-specific instruction files in `.claude/agents/` that contain deep knowledge about each area of the project. Read them for additional context:

- **`.claude/agents/architect.md`** — Architecture patterns, DDD layer rules, dependency direction enforcement, cross-cutting concerns, API design guidelines
- **`.claude/agents/backend.md`** — Full backend directory map, coding conventions per layer, data flows (indexing, chat, provider selection), testing patterns, how to add endpoints/ports/providers
- **`.claude/agents/frontend.md`** — Component patterns (App, ChatWindow, IndexForm, MessageBubble), useChat hook internals, Tailwind styling conventions, API client pattern, testing with vitest/playwright

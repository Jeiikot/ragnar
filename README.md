# Ragnar

RAG-based document analysis tool. Upload source code (ZIP) or PDF documents, index them into ChromaDB, and ask natural-language questions via a LangChain RAG pipeline.

## Quick Start

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/Jeiikot/ragnar.git
cd ragnar

# Copy env and start full stack
cp backend/.env.example backend/.env
docker compose up --build
```

Open http://localhost:5173 in your browser.

## Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI + LangChain + ChromaDB + Python 3.11+ |
| Frontend | React 19 + TypeScript 5.9 + Vite 7 + Tailwind CSS 4 |
| Infrastructure | Docker Compose (Ollama + Backend + Frontend) |
| LLM Providers | OpenAI, Ollama, HuggingFace (auto-selection) |

## Repository Structure

`backend/` and `frontend/` are independent git submodules.

```
ragnar/
├── backend/           → github.com/Jeiikot/ragnar-backend
├── frontend/          → github.com/Jeiikot/ragnar-frontend
├── docker-compose.yml
└── spec/              # Architecture decisions and feature specs
```

## Submodules

```bash
# After cloning without --recurse-submodules
git submodule update --init --recursive

# Update submodules to latest develop
git submodule update --remote
```

See [backend/README.md](backend/README.md) and [frontend/README.md](frontend/README.md) for individual setup.

## LLM Provider Setup

Auto-selection order: **Ollama** (if reachable) → **OpenAI** (if `OPENAI_API_KEY`) → **HuggingFace** (if `HUGGINGFACE_API_KEY`)

```bash
# Recommended local setup
ollama pull qwen2.5:14b
ollama pull nomic-embed-text
```

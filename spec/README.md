# Ragnar — Specs

History of technical decisions and implementation plans for the Ragnar project.

- `decisions/` — Architecture Decision Records (ADR): explain **why** each approach was chosen.
- `features/` — Feature specifications: describe **what** was implemented or is planned.

---

## Architecture Decision Records

| ID | Title | Status |
|----|-------|--------|
| [ADR-001](decisions/001-ddd-lite-architecture.md) | DDD-lite / Ports & Adapters Architecture | ✅ done |
| [ADR-002](decisions/002-chromadb-vector-store.md) | ChromaDB as vector store | ✅ done |
| [ADR-003](decisions/003-multi-provider-llm.md) | Multi-provider LLM system (auto-selection) | ✅ done |
| [ADR-004](decisions/004-per-session-isolation.md) | Per-session isolation (multi-tenancy) | ✅ done |
| [ADR-005](decisions/005-language-aware-chunking.md) | Language-aware chunking for source code | ✅ done |

---

## Features — Backend

| File | Description | Status |
|------|-------------|--------|
| [backend/001-rag-pipeline.md](features/backend/001-rag-pipeline.md) | RAG pipeline with conversation memory (ChatEngine) | ✅ done |
| [backend/002-code-indexing.md](features/backend/002-code-indexing.md) | Source code indexing from ZIP | ✅ done |
| [backend/003-document-indexing.md](features/backend/003-document-indexing.md) | PDF document indexing | ✅ done |
| [backend/004-provider-system.md](features/backend/004-provider-system.md) | LLM/embeddings provider system | ✅ done |

---

## Features — Frontend

| File | Description | Status |
|------|-------------|--------|
| [frontend/001-chat-interface.md](features/frontend/001-chat-interface.md) | Chat interface with history and source citations | ✅ done |
| [frontend/002-index-form.md](features/frontend/002-index-form.md) | File upload and index management form | ✅ done |
| [frontend/003-session-management.md](features/frontend/003-session-management.md) | Multi-session management in sidebar | ✅ done |

---

## Configuration and Workflow

| File | Description | Status |
|------|-------------|--------|
| [features/git-workflow.md](features/git-workflow.md) | Git Flow + Conventional Commits + git-cliff | ✅ done |

---

## How to use this folder

1. **New feature:** create `spec/features/<area>/<id>-<name>.md` with the plan before implementing.
2. **Important technical decision:** create `spec/decisions/<id>-<name>.md` with context and alternatives.
3. **When done:** update the status in this README to `✅ done`.
4. **For Claude:** at the start of a session include "read spec/features/..." to provide context.

### Status legend

| Icon | Meaning |
|------|---------|
| `📋 planned` | Spec created, not yet implemented |
| `🔄 in progress` | In active development |
| `✅ done` | Implemented and committed |
| `🚫 cancelled` | Discarded (reason in the file) |

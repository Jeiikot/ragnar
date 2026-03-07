---
name: ddd-architecture
description: >
  Consult DDD-lite Ports & Adapters architecture rules for the Ragnar project.
  Use when discussing system architecture, planning a new feature, reviewing
  layer boundaries, adding domain ports, or evaluating design decisions.
---

## DDD-Lite (Ports & Adapters)

### Layer Rules

| Layer | May import from | MUST NOT import from |
|-------|----------------|----------------------|
| `domain/` | stdlib only | api, application, infrastructure |
| `application/` | domain, shared | api, infrastructure |
| `infrastructure/` | domain, shared | api |
| `api/` | all layers | — |
| `shared/` | stdlib, pydantic | all other layers |

### Domain Layer
- Defines `typing.Protocol`-based ports — NEVER `abc.ABC`.
- Pure entities: `@dataclass(frozen=True)` with `field(default_factory=...)` for mutable defaults.
- Port grouping via frozen dataclasses (e.g., `IndexingPorts`, `DocumentIndexingPorts`).

### Application Layer
- Services are plain functions receiving port bundles and settings as arguments.
- They never construct ports themselves and never import from infrastructure.
- Reference patterns: `index_directory()`, `index_zip_bytes()`, `index_documents()`.

### Infrastructure Layer
- Concrete adapters implementing domain Protocols.
- Wiring via `build_*_ports()` functions using `functools.partial` to bind settings.
- Example: `build_indexing_ports(settings, session_id)` → `IndexingPorts`.

### API Layer (Composition Root)
- `api/dependencies.py` is the SOLE composition root.
- Wires `Settings`, `ChatEngine`, `IndexingPorts`, `DocumentIndexingPorts` via FastAPI `Depends()`.
- Tests override dependencies with `app.dependency_overrides`.
- All routes under `/api/v1/` prefix.

## Provider Strategy Pattern

Three LLM/embedding providers (OpenAI, Ollama, HuggingFace) with identical `ProviderBuilders` contracts:
- `build_chat_model(settings) -> BaseChatModel`
- `build_embeddings(settings) -> Embeddings`

Auto-selection priority: **Ollama (if reachable) > OpenAI (if key set) > HuggingFace (if key set)**

The `selector.py` uses dependency injection for the Ollama probe to enable testing.

## Session-Scoped Collections

Each `session_id` maps to a separate ChromaDB collection → multi-tenant isolation at the vector store level.

## Key Data Flows

### Indexing Flow (Code)
1. `POST /api/v1/index/code` receives ZIP upload; `session_id` from `Form` data.
2. `get_indexing_ports` dependency calls `build_indexing_ports(settings, session_id)` → `IndexingPorts` bundle.
3. Router calls `index_zip_bytes(zip_bytes, ports, settings)` via `asyncio.to_thread()`.
4. `index_zip_bytes()` validates, extracts to temp dir, then calls `index_directory(path, ports, settings)`.
5. `index_directory()` calls `ports.collect_files()`, `ports.split_file()` per file, `ports.write_documents()` to Chroma.

### Chat Flow
1. `POST /api/v1/chat` receives `{message, session_id}`.
2. Router calls `engine.aask(question, session_id)`.
3. `ChatEngine.aask()` builds retriever for session's collection, retrieves docs, invokes LCEL chain with history.
4. Returns `ChatResponse(answer, sources)`.

### Provider Selection Flow
1. `build_chat_engine()` calls `build_chat_model(settings)`.
2. `resolve_chat_provider(settings)` checks `settings.chat_provider`:
   - If not "auto": use that provider directly.
   - If "auto": probe Ollama → check OpenAI key → check HuggingFace key.
3. Look up `_PROVIDER_BUILDERS[provider].build_chat_model(settings)`.

## Dependency Direction Enforcement

- Domain never imports from infrastructure/api.
- Application imports only from domain and shared (port bundles are injected, never constructed inside services).
- Infrastructure implements domain protocols.
- API layer (`dependencies.py`) is the sole composition root that calls `build_indexing_ports`/`build_document_ports`.
- Request schemas use `model_config = ConfigDict(extra="forbid")`.
- Schemas mirror between backend (`api/schemas/`) and frontend (`api/client.ts`).

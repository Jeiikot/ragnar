# ADR-001: DDD-lite / Ports & Adapters Architecture

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend

---

## Context

Ragnar needs to:
- Swap LLM providers (Ollama, OpenAI, HuggingFace) without breaking the application core.
- Unit-test indexing and chat logic without spinning up ChromaDB or real models.
- Maintain clear separation between business rules and infrastructure details.

## Decision

Use **DDD-lite with Ports & Adapters** (simplified Hexagonal Architecture):

```
API (composition root)
  └─ Application (use cases: plain functions)
       └─ Domain (protocols + frozen dataclasses)
  └─ Infrastructure (concrete adapters)
  └─ Shared (config: pydantic-settings)
```

### Layers and responsibilities

| Layer | Responsibility | May import from |
|-------|---------------|-----------------|
| `domain/` | Port protocols + entities | stdlib only |
| `application/` | Use cases (pure functions) | `domain`, `shared` |
| `infrastructure/` | Concrete adapters (ChromaDB, LangChain) | `domain`, `shared` |
| `api/` | Composition root: DI, routers, schemas | all layers |
| `shared/` | Configuration (pydantic-settings) | stdlib, pydantic |

### Key patterns

- **Ports as `typing.Protocol`**: never ABC. Protocols live in `domain/` and define contracts without coupling to implementations.
- **Entities as `@dataclass(frozen=True)`**: immutable, no persistence logic.
- **Services as plain functions**: `application/indexing/service.py` are functions that receive `ports` as arguments. They never construct ports internally.
- **Port bundles**: `IndexingPorts` and `DocumentIndexingPorts` are frozen dataclasses grouping the ports needed for a use case.
- **Composition root in `api/dependencies.py`**: `build_indexing_ports()` / `build_document_ports()` assemble concrete adapters; FastAPI `Depends()` injects them per request.
- **`functools.partial` for adapter configuration**: instead of creating wrapper classes, `partial(append_documents_impl, settings, collection_name=...)` is used.

## Alternatives considered

| Alternative | Reason for rejection |
|-------------|---------------------|
| Simple MVC (routers + flat services without layer separation) | Makes provider swapping hard; does not scale well with multiple adapters |
| Full DDD (aggregates, repositories, domain events) | Over-engineering for the current project scale |
| Django REST Framework | Does not fit the LangChain/ChromaDB stack; FastAPI is lighter and async-first |

## Consequences

- **Positive:** Unit tests for `application/` are fast (trivial protocol mocks).
- **Positive:** Adding a new provider (e.g., Cohere) only requires a file in `infrastructure/providers/`.
- **Negative:** More files and folders than a simple MVC approach.
- **Negative:** Requires discipline to respect import rules between layers.

## Key files

- `backend/domain/indexing/ports/protocols.py` — port protocols
- `backend/domain/indexing/ports/bundles.py` — port bundles
- `backend/application/indexing/service.py` — use cases
- `backend/infrastructure/indexing/adapters.py` — adapter wiring
- `backend/api/dependencies.py` — composition root (FastAPI DI)

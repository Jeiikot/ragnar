---
name: python-conventions
description: >
  Apply Python coding conventions for the Ragnar project. Use when writing,
  reviewing, or discussing Python or backend code: new endpoints, domain entities,
  infrastructure adapters, services, tests, or any .py file.
---

## Python Style

- **Every file starts with:** `from __future__ import annotations`
- **Line length:** 99 characters max (ruff, target Python 3.11)
- **Import ordering:** stdlib → third-party → local (ruff handles this)
- **Type hints:** Use everywhere. Union syntax: `str | None`, not `Optional[str]`. Use `list[str]`, not `List[str]`.
- **Logging:** `logger = logging.getLogger(__name__)` at module level; structured JSON logging in production.

## Domain Layer Rules

- Entities are `@dataclass(frozen=True)` with `field(default_factory=...)` for mutable defaults.
- Ports use `typing.Protocol`, NEVER `abc.ABC`.
- Domain MUST NOT import from `api/`, `application/`, or `infrastructure/`.
- Port grouping via frozen dataclasses (e.g., `IndexingPorts`, `DocumentIndexingPorts`).

## Application Layer Rules

- Services are plain functions (not classes) receiving ports and settings as arguments — they never construct ports themselves.
- Application MUST NOT import from any infrastructure module (`zip_utils`, `storage`, `chunking`, `adapters`, etc.).
- Application imports only from `domain` and `shared`.
- Port bundles are built by `api/dependencies.py` (composition root) and injected via FastAPI `Depends()`.

## API Layer Rules

- All routes under `/api/v1/` prefix. Routers use `APIRouter(prefix="/api/v1", tags=[...])`.
- Request schemas: `ConfigDict(extra="forbid")` to reject unknown fields.
- Response schemas: plain `BaseModel` without extra constraints.
- Error responses use the `ErrorResponse` schema.
- `api/dependencies.py` is the sole composition root.
- Async route handlers that delegate to services/engine.
- Use `asyncio.to_thread()` for CPU-bound work (indexing).

## Infrastructure Layer Rules

- Implements domain Protocols with concrete functions.
- Adapters wired via `build_*_ports()` functions using `functools.partial` to bind settings.
- Provider implementations follow `ProviderBuilders` contract: `build_chat_model(settings) -> BaseChatModel`, `build_embeddings(settings) -> Embeddings`.
- `ChatEngine` is a class (the only one) because it holds LCEL chain + history state.

## Testing Patterns

- **Fixtures** in `conftest.py`: `test_settings`, `mock_retriever`, `mock_chat_engine`, `client`.
- **Unit tests:** Use `unittest.mock.MagicMock`, `AsyncMock`, `patch`. Group in classes (e.g., `TestFormatDocs`).
- **Router tests:** Use `SyncASGIClient` with `dependency_overrides` to mock engines/settings.
- **Markers:** `@pytest.mark.integration` for tests needing real Chroma, `@pytest.mark.e2e` for tests needing API keys.
- **Async:** `asyncio_mode = "auto"` — async test functions just work.
- Run: `uv run pytest -q` (unit), `uv run pytest -m integration`, `uv run pytest -m e2e`.
- Lint: `uv run ruff check .` and `uv run ruff format .`.

## Configuration Pattern

- All settings in `shared/config.py` `Settings` class (pydantic-settings BaseSettings).
- Env vars map directly to field names (uppercase in `.env`, lowercase in code).
- Validators use `@field_validator` with `@classmethod`.
- Access via `get_settings()` lazy singleton; reset with `reset_settings()` for tests.

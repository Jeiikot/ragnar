# ADR-004: Per-session isolation (multi-tenancy with session_id)

**Date:** 2026-01-15
**Area:** Backend — API + Infrastructure

---

## Planning

- [x] Research — context and alternatives documented
- [x] Decision — rationale written
- [x] Applied — decision implemented in the codebase
- [x] Validated — consequences confirmed in practice

---

## Context

A single Ragnar deployment may be used by:
- A single user with multiple projects (project A code vs. project B).
- Multiple concurrent users whose indexes must not mix.

A simple mechanism is needed to isolate both vector indexes and conversation history.

## Decision

Use **`session_id` as the isolation key** throughout the entire stack:

### In the vector index (ChromaDB)

Each `session_id` maps to a separate ChromaDB collection:

```python
collection_name = session_id or settings.chroma_collection_name
```

- Indexing with `session_id="project-backend"` → collection `"project-backend"`.
- Indexing with `session_id="project-frontend"` → collection `"project-frontend"`.
- Clearing `session_id="project-backend"` does not affect `"project-frontend"`.

### In conversation history (ChatEngine)

```python
_session_store: dict[str, ChatMessageHistory] = {}
# _session_store["project-backend"] → history for that session
```

`RunnableWithMessageHistory` automatically injects the correct history using `session_id`.

### In the API

All endpoints accept `session_id` as a parameter:
- **Form data** (POST indexing endpoints): `session_id: str = Form("default")`
- **Query param** (GET status): `session_id: str = Query("default")`
- **JSON body** (chat): `session_id: str = "default"`

### In the frontend

The frontend generates a UUID per session when creating it:
```typescript
{ id: crypto.randomUUID(), name: `Session ${sessions.length + 1}` }
```

That UUID is passed to all requests for that session.

## Alternatives considered

| Alternative | Reason for rejection |
|-------------|---------------------|
| Authentication (JWT/OAuth) | Over-engineering for the current version; no registered users |
| Metadata filters in a shared collection | More complex; deletion requires filtering and removing individual docs |
| API keys per user | Requires user management; out of scope |

## Consequences

- **Positive:** Multi-tenancy without authentication; simple to implement and test.
- **Positive:** The frontend can manage multiple projects simultaneously.
- **Positive:** Deleting a session is atomic (`clear_collection`).
- **Negative:** If the user loses the `session_id` (e.g., page reload), they lose access to the index.
- **Negative:** No usage limits per session (anyone can create unlimited collections).
- **Negative:** Sessions do not expire automatically; cleanup is manual.

## Key files

- `backend/api/routers/index.py` — `session_id` parameter in all endpoints
- `backend/api/routers/chat.py` — `session_id` in ChatRequest
- `backend/infrastructure/indexing/storage.py` — `collection_name` as parameter
- `backend/infrastructure/retriever.py` — retriever per `session_id`
- `backend/infrastructure/chat/engine.py` — `_session_store` per `session_id`
- `frontend/src/App.tsx` — UUID generation per session

# Feature: Standardized API Response Envelope and Error Handler

**Status:** ✅ done
**Date:** 2026-02-26
**Area:** Backend + Frontend

---

## Description

Introduced a uniform response envelope for all domain endpoints and a structured error response with machine-readable error codes. The health endpoint is excluded (liveness primitive).

See [ADR-006](../../decisions/006-api-response-envelope.md) for the architectural rationale.

## Involved files

| File | Role |
|------|------|
| `backend/api/schemas/response.py` | New — `ApiResponse[T]`, `ApiListResponse[T, M]`, `ApiErrorResponse`, `ErrorCode` |
| `backend/api/schemas/index.py` | Updated — removed `status` from `IndexResponse`, added `ClearResponse`, moved `total_chunks` to meta |
| `backend/api/schemas/__init__.py` | Updated — exports new types |
| `backend/api/main.py` | Updated — three exception handlers return `ApiErrorResponse` |
| `backend/api/routers/index.py` | Updated — all endpoints wrapped in `ApiResponse` / `ApiListResponse` |
| `backend/api/routers/chat.py` | Updated — chat endpoint wrapped in `ApiResponse` |
| `frontend/src/api/client.ts` | Updated — envelope interfaces, `requestEnveloped<T>()`, updated API functions |

## Implemented behavior

### Response shapes per endpoint

| Endpoint | Success shape | Error responses |
|---|---|---|
| `POST /api/v1/index/code` | `ApiResponse[IndexResponse]` | 400 `INVALID_FILE_TYPE`, 500 `INDEXING_FAILED` |
| `POST /api/v1/index/documents` | `ApiResponse[IndexResponse]` | 400 `INVALID_FILE_TYPE`, 500 `INDEXING_FAILED` |
| `GET /api/v1/index/status` | `ApiListResponse[IndexSourceInfo, IndexStatusMeta]` | — |
| `POST /api/v1/index/clear` | `ApiResponse[ClearResponse]` | — |
| `POST /api/v1/chat` | `ApiResponse[ChatResponse]` | 500 `CHAT_FAILED` |
| `GET /api/v1/health` | `HealthResponse` (unwrapped) | — |

### Success envelope

```python
# Single object
class ApiResponse(BaseModel, Generic[T]):
    data: T

# List with metadata
class ApiListResponse(BaseModel, Generic[T]):
    data: list[T]
    meta: IndexStatusMeta

class IndexStatusMeta(BaseModel):
    total_items: int
    total_chunks: int
```

Example — `POST /api/v1/index/code`:
```json
{
  "data": {
    "documents_indexed": 42
  }
}
```

Example — `GET /api/v1/index/status`:
```json
{
  "data": [
    { "name": "repo.zip", "chunks": 15 },
    { "name": "docs.pdf", "chunks": 8 }
  ],
  "meta": {
    "total_items": 2,
    "total_chunks": 23
  }
}
```

### Error envelope

```python
class ErrorCode(StrEnum):
    INVALID_FILE_TYPE = "INVALID_FILE_TYPE"
    INDEXING_FAILED   = "INDEXING_FAILED"
    CHAT_FAILED       = "CHAT_FAILED"
    VALIDATION_ERROR  = "VALIDATION_ERROR"
    INTERNAL_ERROR    = "INTERNAL_ERROR"

class ApiErrorResponse(BaseModel):
    detail: str
    error_code: ErrorCode
    details: dict[str, Any] | None = None
```

Example — invalid file type:
```json
{
  "detail": "Uploaded file must be a .zip archive",
  "error_code": "INVALID_FILE_TYPE",
  "details": null
}
```

Example — validation error:
```json
{
  "detail": "Validation error",
  "error_code": "VALIDATION_ERROR",
  "details": {
    "message": ["String should have at least 1 character"]
  }
}
```

### Exception handler flow

Routers raise `HTTPException` with a structured dict detail:
```python
raise HTTPException(
    status_code=400,
    detail={"error_code": ErrorCode.INVALID_FILE_TYPE, "detail": "Uploaded file must be a .zip archive"},
)
```

The global handler reads `exc.detail` dict if available; otherwise falls back to HTTP status → generic code mapping:
```
400 → INVALID_FILE_TYPE
422 → VALIDATION_ERROR
500 → INTERNAL_ERROR
```

`RequestValidationError` handler populates `details` from Pydantic's `exc.errors()`.

### Frontend client pattern

```typescript
// Private helper — unwraps envelope.data
async function requestEnveloped<T>(path: string, options?: RequestInit): Promise<T> {
  const envelope = await request<ApiResponse<T>>(path, options);
  return envelope.data;
}

// Error parsing — backward compatible
const message = body?.detail ?? response.statusText;
```

Components and hooks receive the same flat domain types as before — envelope is transparent above the API client layer.

## Related tests

- `backend/tests/unit/api/test_schemas.py` — `TestApiResponse`, `TestApiErrorResponse`, `TestClearResponse`
- `backend/tests/unit/api/routers/test_index.py` — updated assertions for new envelope
- `backend/tests/unit/api/routers/test_chat.py` — updated assertions for new envelope
- `frontend/tests/unit/components/IndexForm.test.tsx` — mock returns updated (no `status` field)
- `frontend/tests/integration/hooks/useChat.test.ts` — fetch mocks updated to envelope shape

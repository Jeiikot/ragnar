# ADR-006: Standardized API Response Envelope

**Status:** ✅ done
**Date:** 2026-02-26
**Area:** API (Backend + Frontend)

---

## Context

Each endpoint returned its own schema directly (`IndexResponse`, `ChatResponse`, etc.) with no shared structure. Error responses used a flat `{"detail": "message"}` (FastAPI default) with no machine-readable error code. `/index/clear` returned a raw Python `dict` with no declared `response_model`. This made the OpenAPI schema inconsistent and offered no way to distinguish error categories programmatically.

## Decision

Adopt a two-shape convention:

**Success — single object:**
```json
{ "data": { ...domain payload... } }
```

**Success — list with metadata:**
```json
{
  "data": [ ...items... ],
  "meta": { "total_items": N, ...context... }
}
```

**Error:**
```json
{
  "detail": "Human readable description",
  "error_code": "MACHINE_READABLE_CODE",
  "details": { "field": ["reason"] }
}
```

`detail` is the human-readable message. `error_code` is machine-readable for programmatic branching. `details` carries field-level information (most useful for validation errors).

### Error codes

| Code | HTTP status | When |
|------|-------------|------|
| `INVALID_FILE_TYPE` | 400 | Uploaded file is not the expected type |
| `INDEXING_FAILED` | 500 | Unhandled error during indexing |
| `CHAT_FAILED` | 500 | Unhandled error during chat |
| `VALIDATION_ERROR` | 422 | Pydantic request validation failure |
| `INTERNAL_ERROR` | 500 | Any other unhandled exception |

### Health endpoint exception

`GET /api/v1/health` is **excluded** from the envelope. It is a liveness primitive used by Docker healthchecks and load balancers that expect a stable flat shape. It keeps returning `{"status": "ok", "version": "0.1.0"}` directly.

### OpenAPI compatibility

`ApiResponse[T]` and `ApiListResponse[T, M]` are `Generic` Pydantic v2 models. FastAPI generates a concrete parameterized schema per endpoint in the OpenAPI spec, so Swagger shows the exact `data` shape for each endpoint.

### Frontend contract

The API client (`frontend/src/api/client.ts`) unwraps the envelope internally. Components and hooks continue to receive flat domain types unchanged.

## Alternatives considered

| Alternative | Reason for rejection |
|---|---|
| Keep flat success, only standardize errors | Success and error responses stay incompatible in shape; consumers branch only on HTTP status |
| `{"success": bool, "data": T, "error": ...}` single union | Ambiguous in OpenAPI; both `data` and `error` show as optional; less idiomatic than HTTP status already provides |
| Full RFC 7807 Problem Details | More fields than needed (`type`, `title`, `instance`); `error_code` + `detail` + `details` covers Ragnar's needs with less verbosity |
| JSON:API full spec | Too much structure for a small API (relationships, links, included resources) |

## Consequences

- **Positive:** Uniform shape; machine-readable error codes; OpenAPI docs accurately reflect each endpoint's payload; `details` enables field-level error messages for validation failures.
- **Negative:** Frontend API client must unwrap `envelope.data`; existing tests must be updated to the new shape.

## Key files

- `backend/api/schemas/response.py` — `ApiResponse[T]`, `ApiListResponse[T, M]`, `ApiErrorResponse`, `ErrorCode`
- `backend/api/schemas/index.py` — `ClearResponse` added; `status` field removed from `IndexResponse`; `total_chunks` moves to `meta`
- `backend/api/main.py` — three exception handlers updated
- `backend/api/routers/index.py` — all four endpoints wrapped
- `backend/api/routers/chat.py` — chat endpoint wrapped
- `frontend/src/api/client.ts` — envelope interfaces, `requestEnveloped<T>()` helper

## Related specs

- [005-api-response-envelope (feature)](../features/backend/005-api-response-envelope.md)

# Feature: File upload and index management form

**Date:** 2026-01-15
**Area:** Frontend — Components

---

## Planning

- [x] Design — indexing modes, component state machine, and UI sections defined
- [x] Implementation — IndexForm component with progress timer built
- [x] Tests — unit tests and e2e upload flow passing
- [x] Integration — mounted in sidebar of App.tsx, calls all index API endpoints

---

## Description

Sidebar panel that allows the user to:
1. Select the indexing mode: **Code** (ZIP) or **Documents** (PDF / ZIP of PDFs).
2. Upload a file and trigger indexing.
3. View progress with contextual messages and elapsed time.
4. Check the current index status (indexed sources + chunk counts).
5. Clear the session index with confirmation.

## Involved files

| File | Role |
|------|------|
| `frontend/src/components/IndexForm.tsx` | Main form component |
| `frontend/src/api/client.ts` | `indexCodebaseZip()`, `indexDocuments()`, `getIndexStatus()`, `clearIndex()` |

## Implemented behavior

### Indexing modes

```typescript
type Mode = "code" | "documents";
```

- **Code:** accepts `.zip` → calls `indexCodebaseZip()` → `POST /api/v1/index/code`
- **Documents:** accepts `.pdf` or `.zip` → calls `indexDocuments()` → `POST /api/v1/index/documents`

Switching modes resets the selected file and status.

### Component state

```typescript
type Status =
  | { kind: "idle" }
  | { kind: "loading" }
  | { kind: "success"; count: number }
  | { kind: "error"; message: string };
```

### UI sections

1. **Mode toggle:** Two radio-like buttons (Código / Documentos).
2. **File picker:** Hidden native input + styled "Examinar" button + filename display.
3. **Submit button:** Dynamic label based on mode and state:
   - Code: "Indexar código" / "Indexando..."
   - Documents: "Indexar documentos" / "Procesando..."
4. **Progress hint:** Contextual messages during loading:
   - "Extrayendo archivos del ZIP..."
   - "Generando embeddings..."
   - etc.
5. **Status feedback:** Badge with icon:
   - `✓ N documentos indexados` (emerald)
   - `✗ Error message` (red)
6. **Index status panel:**
   - Source list with name and chunk count.
   - Total chunk count.
   - "Limpiar índice" button with native confirmation (`window.confirm`).
   - Auto-refreshes after indexing or clearing.

### Progress timer

`useElapsedSeconds(active)` — same as ChatWindow:
- Shows `"3s - Generando embeddings..."` during indexing.
- Resets when done.

## Backend contract

```typescript
// Index code
POST /api/v1/index/code  (FormData: file, session_id)
→ { status: "ok", documents_indexed: number }

// Index documents
POST /api/v1/index/documents  (FormData: file, session_id)
→ { status: "ok", documents_indexed: number }

// Index status
GET /api/v1/index/status?session_id=...
→ { sources: [{ name: string, chunks: number }], total_chunks: number }

// Clear index
POST /api/v1/index/clear  (FormData: session_id)
→ { status: "ok" }
```

## Related tests

- `frontend/tests/unit/components/IndexForm.test.tsx`
- `frontend/tests/e2e/` — upload and verification flow with Playwright

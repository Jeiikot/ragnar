# Feature: PDF document indexing

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend — Application + Infrastructure + API

---

## Description

Allows the user to upload a single PDF or a ZIP of PDFs. The system:
1. Detects whether the file is a PDF or ZIP.
2. Extracts and processes each PDF (text by pages).
3. Splits the content into chunks.
4. Stores the embeddings in ChromaDB under the session's collection.

## Involved files

| File | Role |
|------|------|
| `backend/api/routers/index.py` | Endpoint `POST /api/v1/index/documents` |
| `backend/api/schemas/index.py` | `IndexResponse` schema |
| `backend/api/dependencies.py` | `get_document_ports()` — per-request wiring |
| `backend/application/indexing/service.py` | `index_documents()`, `_index_pdf_zip()` |
| `backend/domain/indexing/ports/protocols.py` | `PdfChunkerProtocol`, `VectorStoreWriterProtocol`, `ZipExtractorProtocol` |
| `backend/domain/indexing/ports/bundles.py` | `DocumentIndexingPorts` bundle |
| `backend/infrastructure/indexing/adapters.py` | `build_document_ports()` — adapter wiring |
| `backend/infrastructure/indexing/pdf_reader.py` | `read_and_chunk_pdf()` — PDF reading and chunking |
| `backend/infrastructure/indexing/storage.py` | `append_documents()` — write to ChromaDB |

## Data flow

```
POST /api/v1/index/documents (multipart: file, session_id)
  │
  ├─ [API] reads bytes + filename
  ├─ [API] asyncio.to_thread(index_documents, bytes, filename, ports, settings)
  │
  ├─ [App] index_documents()
  │     ├─ if filename.endswith(".pdf"):
  │     │     docs = ports.read_pdf(bytes, filename, chunk_size, overlap)
  │     └─ elif filename.endswith(".zip"):
  │           docs = _index_pdf_zip(bytes, ports, settings)
  │                 ├─ extracts ZIP to temp dir
  │                 ├─ finds all .pdf files
  │                 └─ for each pdf: docs += ports.read_pdf(bytes, name, ...)
  │
  ├─ [App] ports.write_documents(docs)
  └─ [API] IndexResponse(status="ok", documents_indexed=N)
```

## Differences from code indexing

| Aspect | Code (ZIP) | Documents (PDF) |
|--------|-----------|-----------------|
| Input format | ZIP of source code files | PDF or ZIP of PDFs |
| Splitter | Language-aware (`RecursiveCharacterTextSplitter.from_language`) | Generic character splitter |
| Filtering | `.ignore` policy | No filtering (all PDFs in ZIP) |
| Source metadata | Relative file path | PDF filename |

## Relevant configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `chunk_size` | `1500` | Maximum size per chunk |
| `chunk_overlap` | `200` | Overlap between chunks |

## Related tests

- `backend/tests/unit/core/` — `index_documents` tests with mocks
- `backend/tests/integration/core/` — tests with real PDFs and ChromaDB

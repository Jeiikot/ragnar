# Feature: Source code indexing from ZIP

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend — Application + Infrastructure + API

---

## Description

Allows the user to upload a ZIP file with source code. The system:
1. Validates and extracts the ZIP safely.
2. Discovers indexable files according to `.ignore` rules.
3. Splits each file into language-aware chunks.
4. Stores the embeddings in ChromaDB under the session's collection.

## Involved files

| File | Role |
|------|------|
| `backend/api/routers/index.py` | Endpoint `POST /api/v1/index/code` |
| `backend/api/schemas/index.py` | `IndexResponse` schema |
| `backend/api/dependencies.py` | `get_indexing_ports()` — per-request wiring |
| `backend/application/indexing/service.py` | `index_zip_bytes()`, `index_directory()` — use cases |
| `backend/domain/indexing/ports/protocols.py` | `FileCollectorProtocol`, `FileChunkerProtocol`, `VectorStoreWriterProtocol`, `ZipExtractorProtocol` |
| `backend/domain/indexing/ports/bundles.py` | `IndexingPorts` bundle |
| `backend/infrastructure/indexing/adapters.py` | `build_indexing_ports()` — adapter wiring |
| `backend/infrastructure/indexing/zip_utils.py` | `extract_zip_safely()` — safe extraction |
| `backend/infrastructure/indexing/file_discovery.py` | `collect_all_files()` — discovery with `.ignore` |
| `backend/infrastructure/indexing/chunking.py` | `load_and_split()` — language-aware chunking |
| `backend/infrastructure/indexing/storage.py` | `append_documents()` — write to ChromaDB |
| `backend/infrastructure/indexing/constants.py` | `EXTENSION_LANGUAGE_MAP` |

## Data flow

```
POST /api/v1/index/code (multipart: file, session_id)
  │
  ├─ [API] validates ZIP, reads bytes
  ├─ [API] asyncio.to_thread(index_zip_bytes, bytes, ports, settings)
  │
  ├─ [App] index_zip_bytes()
  │     ├─ validates paths (no absolute, no "..")    ← security
  │     ├─ ports.extract_zip(zip_archive, temp_dir)
  │     └─ index_directory(temp_dir, ports, settings)
  │
  ├─ [App] index_directory()
  │     ├─ ports.collect_files(root)
  │     │     └─ applies .ignore, filters out no-extension files, sorts
  │     ├─ for each (path, extension):
  │     │     └─ docs += ports.split_file(path, ext, root, chunk_size, overlap)
  │     └─ ports.write_documents(docs)
  │
  └─ [API] IndexResponse(status="ok", documents_indexed=N)
```

## Security

- **Path traversal:** `index_zip_bytes()` rejects entries with absolute paths or `..`.
- **`.ignore` policy:** `collect_all_files()` applies the `backend/.ignore` file (gitignore-style) to all uploads. The file is mandatory; if missing, the app fails on startup.

## Relevant configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `chunk_size` | `1500` | Maximum size per chunk |
| `chunk_overlap` | `200` | Overlap between chunks |

## Related tests

- `backend/tests/unit/core/test_indexer.py` — use case tests with port mocks
- `backend/tests/integration/core/` — tests with real ChromaDB

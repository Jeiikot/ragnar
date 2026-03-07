# ADR-005: Language-aware chunking for source code

**Date:** 2026-01-15
**Area:** Backend — Infrastructure

---

## Planning

- [x] Research — context and alternatives documented
- [x] Decision — rationale written
- [x] Applied — decision implemented in the codebase
- [x] Validated — consequences confirmed in practice

---

## Context

Chunking strategy is critical for RAG quality:
- Cutting in the middle of a function destroys semantic context.
- A generic character splitter ignores the structure of the programming language.
- Plain text files (README, configs) should be split differently than code.

## Decision

Use **`RecursiveCharacterTextSplitter` with language mode** for code files, and the generic splitter for everything else.

### Implementation (`infrastructure/indexing/chunking.py`)

```python
# For code: language-aware splitter
splitter = RecursiveCharacterTextSplitter.from_language(
    language=Language.PYTHON,  # or JS, TS, SQL, etc.
    chunk_size=settings.chunk_size,
    chunk_overlap=settings.chunk_overlap,
)

# For plain text (README, configs, etc.)
splitter = RecursiveCharacterTextSplitter(
    chunk_size=settings.chunk_size,
    chunk_overlap=settings.chunk_overlap,
)
```

### Extension-to-language map (`infrastructure/indexing/constants.py`)

```python
EXTENSION_LANGUAGE_MAP = {
    ".py": Language.PYTHON,
    ".js": Language.JS,
    ".ts": Language.JS,     # TypeScript uses JS separators
    ".tsx": Language.JS,
    ".sql": Language.SQL,
    # ... more languages supported by LangChain
}
```

### Metadata added per chunk

Each produced `Document` includes:
```python
metadata = {
    "source": "relative/path/to/file.py",
    "language": "python",
    "file_extension": ".py",
    "chunk_index": 3,
    "start_line": 145,  # calculated from char offset
}
```

`start_line` allows the frontend to display precise source citations (`file:line`).

### Configuration parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `chunk_size` | 1500 | Maximum chunk size in characters |
| `chunk_overlap` | 200 | Overlap between consecutive chunks |

Rule: `chunk_overlap < chunk_size` (validated in `Settings`).

## Alternatives considered

| Alternative | Reason for rejection |
|-------------|---------------------|
| Generic splitter for everything | Cuts functions in half; degrades RAG quality |
| AST-based chunking | More precise but far more complex; per-language dependencies |
| Chunk by function/class | Requires AST parsing; variable chunk size can blow up the context window |

## Consequences

- **Positive:** Better retrieval quality by respecting natural code boundaries.
- **Positive:** Line metadata enables precise source citations in chat.
- **Positive:** Fallback to generic splitter for unknown file types (no crash).
- **Negative:** Unmapped extensions (e.g., `.dart`, `.rs`) use the generic splitter.
- **Negative:** `start_line` is approximate (calculated from character offset, not true AST parsing).

## Key files

- `backend/infrastructure/indexing/chunking.py` — chunking logic
- `backend/infrastructure/indexing/constants.py` — extension → language map
- `backend/shared/config.py` — `chunk_size`, `chunk_overlap`, validation

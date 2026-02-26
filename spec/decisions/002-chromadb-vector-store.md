# ADR-002: ChromaDB as vector store

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend — Infrastructure

---

## Context

The RAG pipeline needs a vector store to:
- Store embeddings for code and document chunks.
- Search by semantic similarity at query time.
- Support multiple isolated sessions (collections).
- Work locally without external services (self-hosted via Docker Compose).

## Decision

Use **ChromaDB** with disk persistence (`./chroma_data`), accessed through the LangChain integration (`langchain-chroma`).

### Configuration

```python
# shared/config.py
chroma_persist_dir: str = "./chroma_data"
chroma_collection_name: str = "ragnar"

# Per session: collection_name = session_id
```

### Multi-tenancy strategy

Each `session_id` maps to an **independent ChromaDB collection**. This allows:
- Isolated indexes per project/user.
- Clean deletion of a session without affecting others (`clear_collection(settings, collection_name)`).
- No need for metadata filtering within a shared collection.

### Search type

`mmr` (Maximal Marginal Relevance) as default:
- Reduces redundancy in retrieved chunks.
- Parameters: `retriever_k=6` (final results), `retriever_fetch_k=20` (initial candidates).

## Alternatives considered

| Alternative | Reason for rejection |
|-------------|---------------------|
| Weaviate | Requires an external server; higher deployment complexity |
| Pinecone | Cloud service (cost, not self-hosted) |
| pgvector (Postgres) | Extra Postgres dependency; ChromaDB is lighter for the current scope |
| FAISS (in-memory) | No persistence; index is lost on restart |
| Qdrant | More powerful but higher overhead for the current scope |

## Consequences

- **Positive:** Zero external dependencies in local development.
- **Positive:** Native integration with LangChain.
- **Positive:** Per-session collections = simple multi-tenancy.
- **Negative:** Not suited for horizontal scaling (single-node file-based storage).
- **Negative:** No built-in administration UI.

## Key files

- `backend/infrastructure/indexing/storage.py` — ChromaDB collection CRUD
- `backend/infrastructure/retriever.py` — per-session retriever construction
- `backend/shared/config.py` — Chroma and retriever configuration parameters

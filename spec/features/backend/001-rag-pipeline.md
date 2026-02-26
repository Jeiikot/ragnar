# Feature: RAG pipeline with conversation memory (ChatEngine)

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend — Infrastructure + API

---

## Description

Chat system based on RAG (Retrieval-Augmented Generation) that:
1. Retrieves relevant chunks from the vector index based on the user's question.
2. Builds a prompt with the retrieved context + conversation history.
3. Generates a response using the configured LLM.
4. Returns the response along with cited sources (`file:line`).

## Involved files

| File | Role |
|------|------|
| `backend/infrastructure/chat/engine.py` | `ChatEngine` — main RAG pipeline class |
| `backend/infrastructure/retriever.py` | `get_retriever()` — retrieves relevant chunks from ChromaDB |
| `backend/infrastructure/providers/__init__.py` | `build_chat_model()` — builds the LLM for the configured provider |
| `backend/domain/chat/entities.py` | `ChatResponse` — domain response entity |
| `backend/api/routers/chat.py` | Endpoint `POST /api/v1/chat` |
| `backend/api/schemas/chat.py` | `ChatRequest` / `ChatResponse` schemas |
| `backend/api/dependencies.py` | `get_chat_engine_dep()` — ChatEngine singleton |

## Implemented behavior

### Endpoint
```
POST /api/v1/chat
Body: { "message": string, "session_id": string }
Response: { "answer": string, "sources": string[] }
```

### LCEL pipeline (LangChain Expression Language)

```
User question
  → get_retriever(session_id).invoke(question)    # semantic search in ChromaDB
  → format docs as context + extract sources
  → PROMPT (system + chat_history + context + question)
  → LLM (Ollama / OpenAI / HuggingFace)
  → StrOutputParser()
  → RunnableWithMessageHistory (injects history per session_id)
  → ChatResponse(answer, sources)
```

### System prompt

The model is instructed to:
- Answer using **only** information from the provided context.
- Be concise (2-4 sentences by default).
- Suggest alternative questions if context is insufficient.
- Cite sources in `file:line` format.

### Conversation memory

- `_session_store: dict[str, ChatMessageHistory]` — in-memory dict, no TTL.
- `RunnableWithMessageHistory` injects the correct history per `session_id`.
- History persists while the process is running (lost on restart).

### ChatEngine singleton

The `ChatEngine` is instantiated once per process and shared across requests (expensive to build due to provider calls). Accessed via `get_chat_engine_dep()` in `api/dependencies.py`.

## Relevant configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `retriever_search_type` | `"mmr"` | Search type: `similarity` or `mmr` |
| `retriever_k` | `6` | Number of chunks retrieved |
| `retriever_fetch_k` | `20` | Initial candidates for MMR |
| `chat_temperature` | `0.1` | LLM temperature (0.0–2.0) |

## Related tests

- `backend/tests/unit/core/test_chat_engine.py` — ChatEngine tests with mocks
- `backend/tests/e2e/` — full end-to-end chat flows

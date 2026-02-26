# Feature: LLM/Embeddings provider system

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Backend — Infrastructure

---

## Description

Abstraction layer that allows swapping the LLM and embeddings provider without modifying the RAG pipeline code. Supports Ollama, OpenAI, and HuggingFace with automatic priority-based selection.

See also: [ADR-003](../../decisions/003-multi-provider-llm.md) for the decision rationale.

## Involved files

| File | Role |
|------|------|
| `backend/infrastructure/providers/__init__.py` | Facade: `build_chat_model()`, `build_embeddings()` |
| `backend/infrastructure/providers/selector.py` | `resolve_chat_provider()`, `resolve_embeddings_provider()` |
| `backend/infrastructure/providers/types.py` | `ProviderName = Literal["openai", "ollama", "huggingface"]` |
| `backend/infrastructure/providers/contracts.py` | `ChatModelBuilder`, `EmbeddingsBuilder` protocols; `ProviderBuilders` dataclass |
| `backend/infrastructure/providers/openai.py` | `ChatOpenAI`, `OpenAIEmbeddings` |
| `backend/infrastructure/providers/ollama.py` | `ChatOllama`, `OllamaEmbeddings` |
| `backend/infrastructure/providers/huggingface.py` | `ChatHuggingFace`, `HuggingFaceEmbeddings` |
| `backend/shared/config.py` | Per-provider environment variables |

## Environment variables

### Provider selection
```bash
CHAT_PROVIDER=auto          # auto | openai | ollama | huggingface
EMBEDDINGS_PROVIDER=auto    # auto | openai | ollama | huggingface
```

### OpenAI
```bash
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1  # override for proxies
CHAT_MODEL=gpt-4o-mini
EMBEDDING_MODEL=text-embedding-3-small
```

### Ollama
```bash
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_CHAT_MODEL=qwen2.5:7b-instruct
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
```

### HuggingFace
```bash
HUGGINGFACE_API_KEY=hf_...
HUGGINGFACE_CHAT_MODEL=HuggingFaceH4/zephyr-7b-beta
HUGGINGFACE_EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
HUGGINGFACE_MAX_NEW_TOKENS=512
```

### Temperature
```bash
CHAT_TEMPERATURE=0.1    # 0.0 – 2.0
```

## How to add a new provider

1. Create `backend/infrastructure/providers/new_provider.py`:
   ```python
   def build_chat_model_new(settings: Settings) -> BaseChatModel: ...
   def build_embeddings_new(settings: Settings) -> Embeddings: ...
   ```

2. Extend `ProviderName` in `types.py`:
   ```python
   ProviderName = Literal["openai", "ollama", "huggingface", "new_provider"]
   ```

3. Register in `__init__.py`:
   ```python
   _PROVIDER_BUILDERS["new_provider"] = ProviderBuilders(
       build_chat_model=build_chat_model_new,
       build_embeddings=build_embeddings_new,
   )
   ```

4. Add detection logic in `selector.py` if it should participate in auto-selection.

## Related tests

- `backend/tests/unit/core/test_providers.py` — provider selection and construction tests

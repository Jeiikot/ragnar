# ADR-003: Multi-provider LLM system with auto-selection

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

Ragnar must work across different environments:
- **Local development:** Ollama running in Docker (no cost, no internet required).
- **GPU server:** Ollama with large models.
- **Cloud:** OpenAI as fallback when Ollama is unavailable.
- **CI / no-GPU environments:** HuggingFace Inference API.

A mechanism is needed that automatically selects the right provider but also allows forcing a specific one via configuration.

## Decision

Implement a **provider system with priority-based auto-selection**:

```
Ollama  →  OpenAI  →  HuggingFace
```

### Auto-selection logic (`infrastructure/providers/selector.py`)

1. If `chat_provider != "auto"`: use the explicit provider.
2. If `"auto"`, probe in order:
   - **Ollama:** HTTP GET to `{ollama_base_url}/api/tags` → available if responds 2xx.
   - **OpenAI:** available if `openai_api_key` is set.
   - **HuggingFace:** available if `huggingface_api_key` is set.
3. If none available: `RuntimeError`.

### Provider module structure

```
infrastructure/providers/
├── __init__.py        # Facade: build_chat_model(), build_embeddings()
├── selector.py        # Auto-selection + provider resolution
├── types.py           # ProviderName = Literal["openai", "ollama", "huggingface"]
├── contracts.py       # Protocols: ChatModelBuilder, EmbeddingsBuilder
├── openai.py          # OpenAI implementations
├── ollama.py          # Ollama implementations
└── huggingface.py     # HuggingFace implementations
```

### Provider registry (Open/Closed)

```python
_PROVIDER_BUILDERS: dict[ProviderName, ProviderBuilders] = {
    "openai": ProviderBuilders(...),
    "ollama": ProviderBuilders(...),
    "huggingface": ProviderBuilders(...),
}
```

Adding a new provider only requires:
1. Creating `infrastructure/providers/new_provider.py`.
2. Adding an entry in `_PROVIDER_BUILDERS`.
3. Extending `ProviderName` in `types.py`.

### Default models

| Provider | Chat | Embeddings |
|----------|------|------------|
| Ollama | `qwen2.5:7b-instruct` | `nomic-embed-text` |
| OpenAI | `gpt-4o-mini` | `text-embedding-3-small` |
| HuggingFace | `zephyr-7b-beta` | `all-MiniLM-L6-v2` |

## Alternatives considered

| Alternative | Reason for rejection |
|-------------|---------------------|
| Single fixed provider | Not portable across environments |
| LangChain RouterChain | Too coupled to LangChain internals; hard to test |
| Mandatory manual configuration | Poor DX; requires extra setup steps |

## Consequences

- **Positive:** Zero-config in environments with Ollama running (Docker Compose).
- **Positive:** Easy extension for new providers.
- **Negative:** Auto-selection makes an HTTP call to Ollama on startup (minimal latency).
- **Negative:** If Ollama is running but the model is not downloaded, it fails late (on the first query).

## Key files

- `backend/infrastructure/providers/__init__.py` — facade
- `backend/infrastructure/providers/selector.py` — auto-selection logic
- `backend/infrastructure/providers/contracts.py` — builder protocols
- `backend/shared/config.py` — per-provider environment variables

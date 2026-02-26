# Feature: Git Flow + Conventional Commits + git-cliff

**Status:** ✅ done
**Date:** 2026-02-26
**Area:** Backend + Frontend — Workflow

---

## Description

Complete version control workflow setup for both sub-projects (`backend/` and `frontend/`), including:
- Independent git repositories with Git Flow.
- Semantic commits (Conventional Commits).
- Automatic CHANGELOG generation with git-cliff.
- Claude sub-agents to assist with commits.

## Involved files

### Backend (`backend/`)
| File | Role |
|------|------|
| `.gitignore` | Excludes `.venv/`, `.env`, caches, `chroma_data/` |
| `cliff.toml` | git-cliff configuration for CHANGELOG generation |
| `CHANGELOG.md` | Auto-generated change history |
| `pyproject.toml` | `version` field (SemVer, updated on release) |

### Frontend (`frontend/`)
| File | Role |
|------|------|
| `.gitignore` | Excludes `node_modules/`, build artifacts, Playwright artifacts |
| `cliff.toml` | git-cliff configuration for CHANGELOG generation |
| `CHANGELOG.md` | Auto-generated change history |
| `package.json` | `version` field (SemVer, updated with `npm version`) |

### Claude sub-agents
| File | Role |
|------|------|
| `.claude/agents/commit-backend.md` | Agent that runs ruff+mypy and creates backend commits |
| `.claude/agents/commit-frontend.md` | Agent that runs ESLint+tsc and creates frontend commits |

## Git Flow setup

Both repos have Git Flow initialized with:
- `main` — production (no direct commits)
- `develop` — integration

```bash
# Tag prefix: empty (git flow creates "v0.1.0", not "vv0.1.0")
git config gitflow.prefix.versiontag ""
git config gitflow.branch.master main
```

## Conventional Commits

Format: `type(scope): description`

### Types
| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Restructuring without behavior change |
| `test` | Tests |
| `docs` | Documentation |
| `chore` | Build, deps, config |
| `style` | Formatting/linting with no logic change |
| `perf` | Performance improvement |
| `ci` | CI/CD |

### Backend scopes
`api`, `domain`, `app`, `infra`, `chat`, `indexing`, `providers`, `config`, `tests`, `deps`, `docker`

### Frontend scopes
`ui`, `components`, `hooks`, `api`, `pages`, `tests`, `deps`, `config`, `docker`

## Release workflow

### Backend
```bash
git flow release start X.Y.Z
# Edit: version = "X.Y.Z" in pyproject.toml
git-cliff --config cliff.toml -o CHANGELOG.md
git add pyproject.toml CHANGELOG.md
git commit -m "chore(release): bump version to X.Y.Z and update changelog"
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release vX.Y.Z" X.Y.Z
```

### Frontend
```bash
git flow release start X.Y.Z
npm version X.Y.Z --no-git-tag-version
git-cliff --config cliff.toml -o CHANGELOG.md
git add package.json CHANGELOG.md
git commit -m "chore(release): bump version to X.Y.Z and update changelog"
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release vX.Y.Z" X.Y.Z
```

### Versioning rule (SemVer)
| Change type | Bump |
|-------------|------|
| `feat` | minor (0.1.0 → 0.2.0) |
| `fix` | patch (0.1.0 → 0.1.1) |
| Breaking change | major (0.x.x → 1.0.0) |

## git-cliff (`cliff.toml`)

Changelog groups:
- Features (`^feat`)
- Bug Fixes (`^fix`)
- Refactoring (`^refactor`)
- Performance (`^perf`)
- Tests (`^test`)
- Documentation (`^docs`)
- Styling (`^style`)
- Dependencies (`^chore\(deps\)`)
- CI (`^chore\(ci\)` and `^ci`)
- Docker (`^chore\(docker\)`)
- Generic chores (skip = true — not shown in CHANGELOG)

Tag pattern: `v[0-9].*`

## Current versions

| Sub-project | Version |
|-------------|---------|
| `backend/` | `0.1.0` |
| `frontend/` | `0.1.0` |

## Initial commits in `backend/` (v0.1.0)

1. `chore(deps): add backend project dependencies and uv setup`
2. `chore(docker): add Dockerfile and docker-compose for backend`
3. `chore(ci): add Makefile and .ignore policy`
4. `docs(readme): add README`
5. `feat(api): implement FastAPI app factory with exception handlers and CORS`
6. `feat(config): add pydantic-settings configuration with provider support`
7. `feat(domain): define indexing ports (protocols and bundles)`
8. `feat(infra): add LLM and embeddings provider system (openai, ollama, huggingface)`
9. `feat(infra): add code indexing adapters (chunking, file discovery, storage, zip)`
10. `feat(infra): add PDF document indexing adapter`
11. `feat(infra): add vector store retriever with MMR support`
12. `feat(infra): add ChatEngine with LCEL pipeline and conversation memory`
13. `feat(app): add indexing use cases (index_zip_bytes, index_documents)`
14. `feat(api): add index router (code, documents, status, clear)`
15. `feat(api): add chat router and dependency injection`
16. `test(api): add unit tests for routers and schemas`
17. `test(core): add unit and integration tests for chat, indexing, and providers`

## Initial commits in `frontend/` (v0.1.0)

1. `chore(deps): initialize Vite + React 19 + TypeScript project`
2. `chore(config): add Tailwind CSS 4, Vitest, and Playwright setup`
3. `chore(docker): add Dockerfile for frontend`
4. `feat(api): add API client with typed request/response interfaces`
5. `feat(hooks): add useChat hook for chat state management`
6. `feat(components): add MessageBubble with collapsible source citations`
7. `feat(components): add IndexForm with file upload and index status panel`
8. `feat(components): add ChatWindow with auto-scroll and elapsed timer`
9. `feat(pages): add App with session management and sidebar layout`
10. `test(components): add unit tests for ChatWindow, IndexForm, MessageBubble`
11. `test(hooks): add integration tests for useChat hook`

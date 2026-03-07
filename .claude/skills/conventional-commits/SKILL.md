---
name: conventional-commits
description: >
  Write Conventional Commit messages for the Ragnar project. Use when creating
  commits, reviewing commit messages, drafting a commit, or discussing git history
  and changelog generation.
---

## Format

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

## Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or fixing tests |
| `docs` | Documentation only (CLAUDE.md, agent files, docstrings) |
| `chore` | Build system, deps, config (no source change) |
| `style` | Formatting / linting auto-fixes with no logic change |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |

## Scopes — Backend

| Scope | Covers |
|-------|--------|
| `api` | Routers, schemas, dependencies (`backend/api/`) |
| `domain` | Entities, ports (`backend/domain/`) |
| `app` | Application services (`backend/application/`) |
| `infra` | Infrastructure adapters (`backend/infrastructure/`) |
| `chat` | Chat engine or chat router |
| `indexing` | Indexing service, adapters, or router |
| `providers` | LLM / embeddings provider modules |
| `config` | `shared/config.py` or settings |
| `tests` | Test files only |
| `deps` | Dependency updates (`pyproject.toml`) |
| `docker` | Dockerfile or docker-compose |

## Scopes — Frontend

| Scope | Covers |
|-------|--------|
| `ui` | Visual-only component changes, Tailwind styles |
| `components` | React component logic (`frontend/src/components/`) |
| `hooks` | Custom hooks (`frontend/src/hooks/`) |
| `api` | API client / types (`frontend/src/api/`) |
| `pages` | Page-level components or routing |
| `tests` | Test files only |
| `deps` | Dependency updates (`package.json`) |
| `config` | Vite, TypeScript, ESLint, Tailwind config files |
| `docker` | Dockerfile or docker-compose |

## Scopes — Root Repository

| Scope | Covers |
|-------|--------|
| `backend` | Submodule pointer update for `backend/` |
| `frontend` | Submodule pointer update for `frontend/` |
| `docker` | `docker-compose.yml` changes |
| `docs` | `README.md`, `CLAUDE.md`, `AGENTS.md`, `spec/**`, `.claude/agents/**` |
| `release` | Coordinated release commits |

## Short Description Rules

- Imperative mood, lowercase, no trailing period.
- Subject line max 72 characters (type + scope + description combined).
- For submodule pointer updates in root repo, include the short SHA.

## Examples

```
feat(indexing): add session_id support to clear endpoint
fix(chat): handle empty retriever result gracefully
refactor(domain): split ports.py into protocols and bundles packages
test(api): add router tests for index_documents endpoint
chore(deps): bump ruff to v0.14.10
style(api): apply ruff formatting fixes
feat(components): add file upload progress indicator
fix(hooks): prevent double submission on slow connections
chore(backend): update submodule to abc1234
chore(release): pin backend v1.1.0 + frontend v0.3.0
docs(docs): update CLAUDE.md with new agent descriptions
```

## Body

Include when:
- The **why** is not obvious from the short description.
- The change spans multiple layers.
- A breaking change needs explanation (`BREAKING CHANGE: <description>` in footer).

## Creating the Commit

```bash
git commit -m "$(cat <<'EOF'
type(scope): short description

Optional body explaining the why.
EOF
)"
```

## Changelog Generation

`git-cliff` reads the full git history and groups commits by type using `cliff.toml`. Version bump rules (SemVer):
- `feat` → minor (0.1.0 → 0.2.0)
- `fix` → patch (0.1.0 → 0.1.1)
- breaking → major (0.x.x → 1.0.0)

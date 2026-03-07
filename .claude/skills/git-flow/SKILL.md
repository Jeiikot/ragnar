---
name: git-flow
description: >
  Follow the Git Flow branching strategy for the Ragnar project. Use when
  creating branches, merging, starting or finishing releases, or discussing
  version control workflow and branching decisions.
---

## Branch Conventions — Backend & Frontend

| Branch pattern | Purpose | Allowed commit types |
|----------------|---------|----------------------|
| `main` | Production — **no direct commits** | — |
| `develop` | Integration branch | all types |
| `feature/<name>` | New features | `feat`, `refactor`, `test`, `docs`, `style`, `perf` |
| `fix/<name>` | Bug fixes | `fix`, `test`, `refactor` |
| `hotfix/<name>` | Critical prod fixes | `fix` |
| `release/<version>` | Release stabilization | `fix`, `docs`, `chore` |
| `chore/<name>` | Tooling, deps, config | `chore`, `ci`, `docs` |

### Branch Warnings (warn but do not block, except main)

- On `feature/*` with a `fix` type → suggest a `fix/*` branch.
- On `release/*` with a `feat` type → warn that features belong on `develop`.

## Branch Conventions — Root Repository

| Branch pattern | Purpose | Allowed commit types |
|----------------|---------|----------------------|
| `main` | Production — **no direct commits** | — |
| `develop` | Integration branch | all types |
| `release/<version>` | Release stabilization and submodule pinning | `chore`, `docs`, `fix` |

> The root repository does **not** use `feature/`, `fix/`, `hotfix/`, or `chore/` branches. All feature and fix work is done inside the submodule repositories.

## Release Workflow — Backend

```bash
# 1. Start release
git flow release start X.Y.Z

# 2. Bump version in pyproject.toml: version = "X.Y.Z"

# 3. Generate changelog
git-cliff --config cliff.toml -o CHANGELOG.md

# 4. Commit version bump + changelog
git add pyproject.toml CHANGELOG.md
git commit -m "chore(release): bump version to X.Y.Z and update changelog"

# 5. Finish release
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release vX.Y.Z" X.Y.Z
```

## Release Workflow — Frontend

```bash
# 1. Start release
git flow release start X.Y.Z

# 2. Bump version (no git tag — git flow handles the tag)
npm version X.Y.Z --no-git-tag-version

# 3. Generate changelog
git-cliff --config cliff.toml -o CHANGELOG.md

# 4. Commit version bump + changelog
git add package.json CHANGELOG.md
git commit -m "chore(release): bump version to X.Y.Z and update changelog"

# 5. Finish release
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release vX.Y.Z" X.Y.Z
```

## Release Workflow — Root Repository

```bash
# 1. Start release
git flow release start X.Y.Z

# 2. Pin submodule pointers to their tagged versions
git submodule update --remote backend   # or: git -C backend checkout vX.Y.Z
git submodule update --remote frontend  # or: git -C frontend checkout vX.Y.Z
git add backend frontend
git commit -m "chore(release): pin backend vX.Y.Z + frontend vX.Y.Z"

# 3. Generate changelog
git-cliff --config cliff.toml -o CHANGELOG.md
git add CHANGELOG.md
git commit -m "docs: update changelog for vX.Y.Z"

# 4. Finish release
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release vX.Y.Z" X.Y.Z
```

## Hard Rules (all repos)

- NEVER use `--no-verify` to skip hooks.
- NEVER amend a previous commit — always create a new one.
- NEVER commit `.env`, credentials, or secrets.
- NEVER push to remote unless the user explicitly asks.
- NEVER commit files unrelated to the described change.
- Root repo: NEVER commit root-level changes if a submodule has uncommitted work.

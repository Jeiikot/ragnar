# Ragnar — Specs

Conventions for documenting technical decisions and feature specifications.

- `decisions/` — Architecture Decision Records (ADR): explain **why** each approach was chosen.
- `features/` — Feature specs: describe **what** is implemented or planned, with execution phases.

---

## When to create each type

| Situation | Type |
|-----------|------|
| Choosing between technical alternatives and want to leave a record | ADR |
| Implementing a new feature and need a plan | Feature spec |
| A feature spans multiple layers (backend + frontend) | Feature spec under `features/` root or `features/<area>/` |

---

## Naming convention

```
decisions/NNN-kebab-name.md    # e.g. 006-api-response-envelope.md
features/<area>/NNN-name.md    # e.g. features/backend/005-api-response-envelope.md
```

The numeric prefix is sequential within its folder. Use three digits (`001`, `002`, …).

---

## Phase status

Each phase inside `## Planning` uses a Markdown checkbox:

| Symbol | Meaning |
|--------|---------|
| `- [ ]` | Pending |
| `- [x]` | Done |

If a spec is discarded before implementation, add at the top of the file:

```
> **Cancelled:** <reason>
```

---

## Template — ADR

```markdown
# ADR-NNN: Decision title

**Date:** YYYY-MM-DD
**Area:** Backend | Frontend | Backend + Frontend

---

## Planning

- [ ] Research — context and alternatives documented
- [ ] Decision — rationale written
- [ ] Applied — decision implemented in the codebase
- [ ] Validated — consequences confirmed in practice

---

## Context

<Why this decision arose. What problem it solves.>

## Decision

<What was decided and why.>

## Alternatives considered

| Option | Reason rejected |
|--------|----------------|
| ...    | ...            |

## Consequences

**Positive:** …
**Negative / trade-offs:** …

## Key files

| File | Role |
|------|------|
| ...  | ...  |
```

---

## Template — Feature spec

```markdown
# Feature: Feature name

**Date:** YYYY-MM-DD
**Area:** Backend | Frontend | Backend + Frontend

---

## Planning

- [ ] Design — spec written and acceptance criteria defined
- [ ] Implementation — code written
- [ ] Tests — unit + integration passing
- [ ] Integration — wired into the rest of the system

---

## Description

<What this feature does. Capabilities it enables.>

## Involved files

| File | Role |
|------|------|
| ...  | ...  |

## Implemented behavior

<Architecture details, data flows, API contracts, schemas, etc.>

## Related tests

- `tests/unit/...`
- `tests/integration/...`
```

---

## Referencing a spec

From commits or PRs:

```
feat(api): add response envelope

See spec/features/backend/005-api-response-envelope.md
See spec/decisions/006-api-response-envelope.md
```

From other specs (relative Markdown link):

```markdown
See [ADR-006](../../decisions/006-api-response-envelope.md) for the architectural rationale.
```

---

## For Claude

At the start of an implementation session, provide the relevant spec:

```
Read spec/features/backend/005-api-response-envelope.md
```

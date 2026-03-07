---
name: react-conventions
description: >
  Apply React and TypeScript coding conventions for the Ragnar frontend.
  Use when writing, reviewing, or discussing frontend code: new components,
  hooks, API client updates, TypeScript types, or test writing.
---

## TypeScript / React

- **Function components only** — never use class components.
- **Named exports** for components: `export function ChatWindow(...)`, not default exports (exception: `App.tsx` uses `export default`).
- **Props as inline interfaces** or nearby type definitions, not separate files.
- **Hooks in `src/hooks/`** — custom hooks prefixed with `use`.
- **No CSS modules** — Tailwind utility classes directly in JSX `className`.
- **`import type`** for type-only imports: `import type { Message } from "../hooks/useChat"`.
- **State management:** React built-in hooks only (`useState`, `useCallback`, `useEffect`, `useRef`). No Redux, Zustand, or context providers.

## API Client Pattern (`src/api/client.ts`)

- Base URL from `import.meta.env.VITE_API_URL ?? ""`.
- Generic `request<T>(path, options)` helper for JSON endpoints.
- `parseJsonResponse<T>(response)` handles error extraction from backend `ErrorResponse`.
- File uploads use `FormData` directly (no JSON content-type header).
- **Interfaces mirror backend schemas exactly:** `IndexResponse`, `ChatRequest`, `ChatResponse`, `HealthResponse`, `IndexStatusResponse`, `IndexSourceInfo`.
- When backend adds a new schema, `client.ts` interfaces MUST be updated to match.

## Component Patterns

**App.tsx:**
- Manages `ChatSession[]` and active session ID.
- Sessions created with `crypto.randomUUID()`.
- Health check on mount via `useEffect` calling `checkHealth()`.
- `ChatWindow` uses `key={activeSession.id}` to force remount on session change.

**ChatWindow.tsx:**
- Uses `useChat(sessionId)` hook for all state.
- Custom `useElapsedSeconds(active)` hook for loading timer.
- Textarea with Enter-to-submit (Shift+Enter for newline).
- Auto-scrolls to bottom via `useRef` + `scrollIntoView`.

**IndexForm.tsx:**
- Toggle between "code" and "documents" mode.
- Discriminated union for status: `{ kind: "idle" | "loading" | "success" | "error" }`.

**MessageBubble.tsx:**
- Presentational component only.
- User: indigo background, right-aligned. Assistant: zinc background, left-aligned.
- Source citations as `font-mono` rounded badges.

**useChat.ts hook:**
- Returns `{ messages, isLoading, error, sendMessage, clearChat }`.
- `Message`: `{ id: string, role: "user" | "assistant", content: string, sources?: string[], timestamp: number }`.
- Appends user message immediately (optimistic), then appends assistant response.

## Testing Patterns

**Unit tests (vitest + @testing-library/react):**
- Located in `tests/unit/components/`.
- Import: `import { describe, it, expect, vi } from "vitest"`.
- Mock hooks with `vi.mock("../../../src/hooks/useChat", () => ({ useChat: () => ({...}) }))`.
- Use `render()`, `screen.getByText()`, `screen.getByPlaceholderText()`.
- Group tests in `describe` blocks.

**E2E tests (Playwright):**
- Located in `tests/e2e/`.
- Run with: `npm run test:e2e`.

**Running:**
- Unit + integration: `npm test`
- Lint: `npm run lint`
- Build: `npm run build` (runs `tsc -b && vite build`)

## When Writing Code

1. Run `npm run lint` after changes to check ESLint compliance.
2. Run `npm test` after changes to verify tests pass.
3. Run `npm run build` to verify TypeScript compilation succeeds.
4. New component → `src/components/`, named export, test in `tests/unit/components/`.
5. New hook → `src/hooks/`, test in `tests/integration/hooks/`.
6. Backend API changes → update `src/api/client.ts` interfaces first.
7. All user-facing text must be in **Spanish**.
8. Keep components focused and small. Extract reusable logic into hooks.

# Feature: Chat interface with history and source citations

**Status:** ✅ done
**Date:** 2026-01-15
**Area:** Frontend — Components + Hooks

---

## Description

Conversational chat interface that allows the user to:
1. Send questions about indexed code/documents.
2. View assistant responses with conversation history.
3. Inspect model-cited sources (`file:line`) via a collapsible panel.
4. See a loading indicator with elapsed time.
5. Clear the chat history for the current session.

## Involved files

| File | Role |
|------|------|
| `frontend/src/hooks/useChat.ts` | Chat state (messages, loading, error) + send logic |
| `frontend/src/components/ChatWindow.tsx` | Chat UI: messages, input, loading indicator |
| `frontend/src/components/MessageBubble.tsx` | Individual message bubble with collapsible sources |
| `frontend/src/api/client.ts` | `sendMessage()` — calls `POST /api/v1/chat` |

## Implemented behavior

### Hook `useChat(sessionId: string)`

```typescript
interface Message {
  id: string;         // UUID
  role: "user" | "assistant";
  content: string;
  sources?: string[];
  timestamp: number;
}

// Returns:
{ messages, isLoading, error, sendMessage, clearChat }
```

**`sendMessage(text)` flow:**
1. Appends user message with UUID + timestamp (optimistic).
2. Calls `apiSendMessage({ message, session_id })`.
3. Appends assistant response with sources.
4. On error: sets error state, clears loading.

### `ChatWindow` component

**Sections:**
- **Header:** Session name + "Clear chat" button.
- **Messages area:** List of `MessageBubble` components, auto-scrolls to latest.
- **Loading indicator:** Animated dots + elapsed seconds counter.
- **Error banner:** Red message if request fails.
- **Input form:** Textarea + send button.
  - `Enter` → sends, `Shift+Enter` → new line.
  - Disabled while `isLoading` or input is empty.

### `MessageBubble` component

| Aspect | User | Assistant |
|--------|------|-----------|
| Alignment | Right | Left |
| Color | `indigo-600` | `zinc-800` |
| Corners | `rounded-2xl rounded-br-sm` | `rounded-2xl rounded-bl-sm` |

**Collapsible sources:**
- Button with count: "3 sources ▾"
- Click expands list of `font-mono` badges (`zinc-700`)
- Accessible: `aria-expanded`, `aria-label`

### Loading timer

`useElapsedSeconds(active: boolean)` — local hook:
- Increments every second while `isLoading`.
- Resets to 0 when `active = false`.
- Shown as "3s..." next to the spinner.

## Backend contract

```typescript
// Request
POST /api/v1/chat
{ "message": string, "session_id": string }

// Response
{ "answer": string, "sources": string[] }
// sources: ["relative/path/file.py:42", ...]
```

## Related tests

- `frontend/tests/unit/components/ChatWindow.test.tsx`
- `frontend/tests/unit/components/MessageBubble.test.tsx`
- `frontend/tests/integration/hooks/useChat.test.ts`
- `frontend/tests/e2e/` — end-to-end chat flow with Playwright

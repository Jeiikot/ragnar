# Feature: Multi-session management in sidebar

**Date:** 2026-01-15
**Area:** Frontend — App

---

## Planning

- [x] Design — session structure, state management, and sidebar layout defined
- [x] Implementation — App.tsx session state, switch, create, and health check built
- [x] Tests — App component unit tests and e2e session flow passing
- [x] Integration — wraps ChatWindow and IndexForm with active session context

---

## Description

Multi-session system that allows the user to:
1. Create independent sessions (each with its own index and chat history).
2. Navigate between sessions from the sidebar.
3. View the backend health status (version / "offline").
4. Manage the index of each session in isolation.

## Involved files

| File | Role |
|------|------|
| `frontend/src/App.tsx` | Global session state + main layout |
| `frontend/src/api/client.ts` | `checkHealth()` — health check on mount |

## Implemented behavior

### Session structure

```typescript
interface ChatSession {
  id: string;   // UUID generated with crypto.randomUUID()
  name: string; // "Sesión 1", "Sesión 2", etc.
}
```

### State in `App.tsx`

```typescript
const [sessions, setSessions] = useState<ChatSession[]>([initialSession]);
const [activeId, setActiveId] = useState<string>(initialSession.id);
```

### Create new session

```typescript
const handleNewSession = () => {
  const session = { id: crypto.randomUUID(), name: `Sesión ${sessions.length + 1}` };
  setSessions(prev => [...prev, session]);
  setActiveId(session.id);
};
```

### Switch session

- Click on sidebar session → `setActiveId(id)`.
- `ChatWindow` receives `key={activeSession.id}` → React unmounts and remounts the component, clearing local state (messages, input) without affecting the backend.
- `IndexForm` receives the new `sessionId` → refreshes the corresponding index status.

### Health check

```typescript
useEffect(() => {
  checkHealth()
    .then(h => setVersion(h.version))
    .catch(() => setVersion("offline"));
}, []);
```

- Shows `v0.1.0` in the sidebar footer when the backend responds.
- Shows `"offline"` if the backend is unavailable.

### Layout

```
┌─ Sidebar (320px) ─────────────────────┐  ┌─ Main (flex) ────────────────────────┐
│ Ragnar                                 │  │                                       │
│ [+ Nueva sesión]                       │  │  ChatWindow                           │
│                                        │  │  (key=activeId → remount on switch)   │
│ ● Sesión 1  ← active                   │  │                                       │
│   Sesión 2                             │  │                                       │
│                                        │  │                                       │
│ ─────────────────                      │  │                                       │
│ IndexForm (active session)             │  │                                       │
│                                        │  │                                       │
│ v0.1.0                                 │  │                                       │
└────────────────────────────────────────┘  └───────────────────────────────────────┘
```

## Session isolation

Each session has a unique `session_id` (UUID) passed to:
- `ChatWindow` → `useChat(sessionId)` → `sendMessage({ session_id })`
- `IndexForm` → `indexCodebaseZip(file, sessionId)`, `getIndexStatus(sessionId)`, etc.

The backend isolates per `session_id` at the ChromaDB collection level and in-memory conversation history.

See also: [ADR-004](../../decisions/004-per-session-isolation.md)

## Known limitations

- Sessions **do not persist** on page reload (React in-memory state).
- Names are auto-generated ("Sesión N"); no name editing.
- No session limit per instance.

## Related tests

- `frontend/tests/unit/` — App component tests
- `frontend/tests/e2e/` — session creation and switching flow with Playwright

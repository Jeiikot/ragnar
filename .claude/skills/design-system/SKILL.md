---
name: design-system
description: >
  Apply the Ragnar UI design system. Use when designing, reviewing, or discussing
  UI components, layouts, visual design, accessibility, or any task focused on
  the look, feel, or usability of the interface.
---

## Color Palette

| Role | Token |
|------|-------|
| Page background | `bg-zinc-900` |
| Surface / card | `bg-zinc-800` |
| Elevated surface | `bg-zinc-700` |
| Border subtle | `border-zinc-800` |
| Border default | `border-zinc-700` |
| Text primary | `text-zinc-100` |
| Text secondary | `text-zinc-400` |
| Text muted | `text-zinc-500` |
| Accent primary | `indigo-600` |
| Accent hover | `indigo-500` |
| Success | `emerald-*` |
| Error | `red-*` |

## Border Radius Scale

- Containers / panels: `rounded-lg`
- Inputs / buttons: `rounded-xl`
- Message bubbles: `rounded-2xl`
- Badges / chips: `rounded-full`

## Typography Scale

- Base body: `text-sm`
- Labels / metadata: `text-xs`
- Monospace (code, sources): `font-mono text-xs`

## Interaction States

- Hover: `hover:bg-zinc-800` or `hover:bg-indigo-500`
- Active / pressed: `active:scale-95` or `active:opacity-80`
- Disabled: `disabled:opacity-40 disabled:cursor-not-allowed`
- Transitions: `transition-colors duration-150` (always on interactive elements)

## Layout Conventions

- Sidebar width: `w-72` (fixed)
- Content area: `flex-1 overflow-hidden`
- Common padding: `px-4 py-3`
- Common gaps: `gap-2`, `gap-3`, `gap-4`

## UX Principles

1. **Clarity over cleverness** — Every action self-evident. Labels, placeholders, tooltips in Spanish.
2. **Status always visible** — Loading spinners, progress feedback, error messages, and success confirmations must be immediate and descriptive.
3. **Non-destructive defaults** — Confirm before clearing indexed data or deleting sessions.
4. **Keyboard accessible** — All interactive elements reachable via Tab; Enter/Space activates buttons; Escape closes modals.
5. **Responsive but desktop-first** — Primary use is desktop; mobile is secondary.
6. **Consistent spacing rhythm** — Follow Tailwind's 4-point grid. Never invent arbitrary spacing values.
7. **Error recovery** — Error states must explain what went wrong and offer a recovery path.

## Accessibility (a11y) Requirements

- All `<img>` and icon-only buttons must have `aria-label` or `alt` text.
- Form inputs must have associated `<label>` (or `aria-label`).
- Color alone must never convey meaning — always pair with text or icon.
- Focus rings must be visible — never `outline-none` without a custom focus style.
- Semantic HTML: `<button>`, `<nav>`, `<main>`, `<section>`, `<header>` appropriately.
- ARIA live regions (`aria-live="polite"`) for dynamic status messages.
- Minimum touch target: `min-h-[44px] min-w-[44px]` for interactive elements.

## Language

All user-facing text is in **Spanish**:
- Buttons: "Enviar", "Indexar", "Limpiar", "Nueva conversación", "Cancelar"
- States: "Cargando…", "Pensando…", "Indexando…", "Error al indexar"
- Placeholders: "Escribe tu pregunta…", "Selecciona un archivo ZIP…"
- Confirmations: "¿Estás seguro de que quieres limpiar los datos indexados?"

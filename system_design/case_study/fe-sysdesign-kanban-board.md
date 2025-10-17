# Front-End System Design Deep Dive — Kanban Board / Drag-and-Drop

> Build a Trello-like **Kanban board** with drag-and-drop, optimistic updates, and large-list performance using React + TypeScript.

---

## 1. Requirements
- Columns (To Do, Doing, Done) with cards.
- Drag cards within/between columns.
- Persist order; optimistic UI with rollback.
- Filters, search, and assignees.

---

## 2. Architecture
```
KanbanApp
 ├─ Board
 │  ├─ Column[]
 │  │  └─ Card[] (virtualized for large lists)
 └─ DnD layer (dnd-kit / react-beautiful-dnd)
```

---

## 3. Tech Choices
| Area | Choice | Why | Alt |
|---|---|---|---|
| DnD | **dnd-kit** | Modern, good perf | react-beautiful-dnd (archived) |
| State | **React Query** | Sync with server order | Redux |
| Virtualization | **react-window** | Large columns perf | Plain list |

---

## 4. Implementation

```tsx
import { DndContext, useDraggable, useDroppable } from "@dnd-kit/core";

function Card({ card }: { card: any }) {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({ id: card.id });
  const style = transform ? { transform: `translate3d(${transform.x}px, ${transform.y}px, 0)` } : undefined;
  return <div ref={setNodeRef} {...listeners} {...attributes} style={style} className="card">{card.title}</div>;
}
```

Optimistic reorder:
```ts
// onDragEnd: update local order immediately, POST to server; on failure, rollback
```

---

## 5. Performance & A11y
- Virtualize cards; avoid heavy shadows during drag.
- Keyboard DnD fallback; ARIA live updates on reorder.

---

## 6. Testing
- Reorder scenarios; cross-column moves.
- Failure rollback; large dataset scroll.

---

## 7. Interview Summary
> “I use dnd‑kit with optimistic reordering and server reconciliation. Virtualized columns keep scrolling smooth at scale.”

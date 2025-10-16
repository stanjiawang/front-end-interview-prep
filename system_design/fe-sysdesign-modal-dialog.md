# Front-End System Design Deep Dive — Modal / Dialog Component

> Design a reusable, accessible **Modal/Dialog system** with focus management, stacking, and animations in **React + TypeScript**.

---

## 1. Requirements
- Programmatic open/close; ESC and overlay close.
- Focus trap; restore focus to opener.
- Scroll lock; portal to body; z-index stacking.
- Sizes/variants; nested modals (with caution).

---

## 2. Architecture
```
ModalProvider (Context)
 ├─ Portal root (e.g., <div id="modal-root" />)
 ├─ ModalManager (stack)
 └─ <Modal /> component
```

---

## 3. Implementation
```tsx
import { createPortal } from "react-dom";
import { useEffect, useRef } from "react";

export function Modal({ open, onClose, labelledBy, children }: { open: boolean; onClose: () => void; labelledBy: string; children: React.ReactNode; }) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!open) return;
    const prev = document.activeElement as HTMLElement | null;
    const onKey = (e: KeyboardEvent) => { if (e.key === "Escape") onClose(); };
    document.body.style.overflow = "hidden";
    document.addEventListener("keydown", onKey);
    ref.current?.focus();
    return () => { document.body.style.overflow = ""; document.removeEventListener("keydown", onKey); prev?.focus(); };
  }, [open, onClose]);

  if (!open) return null;
  return createPortal(
    <div className="fixed inset-0 z-50" role="dialog" aria-modal="true" aria-labelledby={labelledBy}>
      <div className="absolute inset-0 bg-black/40" onClick={onClose} />
      <div ref={ref} tabIndex={-1} className="absolute inset-0 grid place-items-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-6 max-w-lg w-full">{children}</div>
      </div>
    </div>,
    document.body
  );
}
```

**Note**: for bulletproof focus trap & a11y, prefer mature libs (Radix UI, Headless UI).

---

## 4. Trade-offs
| Topic | DIY | Library |
|---|---|---|
| A11y coverage | Requires diligence | ✅ Built-in |
| Animations | Custom CSS/JS | Components provided |
| Stacking | Custom manager | Often included |
| Bundle size | Minimal | Larger |

---

## 5. Testing
- RTL: focus advances inside modal; ESC closes; overlay click closes.
- E2E: ensure scroll lock; nested dialogs behavior.

---

## 6. Interview Soundbite
> “Portal + focus management + scroll lock are the essentials. I’ll use a proven a11y library unless product needs full custom control.”

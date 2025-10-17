# Front-End System Design Deep Dive — Settings / Preferences (Autosave)

> Design a **settings page** with inline edits, optimistic autosave, error recovery, and audit-friendly UX. React + TypeScript.

---

## 1. Requirements
- Edit profile/settings; autosave after change.
- Visual status (saving… / saved / error with retry).
- Field-level validation; undo.
- Dirty-state navigation guard.

---

## 2. Architecture
```
SettingsPage
 ├─ Section (Profile, Notifications, Security)
 ├─ Field components (Text, Toggle, Select)
 ├─ useAutosave() (debounce + mutation + optimistic UI)
 └─ Toast/inline status
```

---

## 3. Tech Choices
| Area | Choice | Why |
|---|---|---|
| Autosave | Debounce + React Query mutation | Reduce QPS; retries |
| Undo | Local history stack | Quick revert UX |
| Validation | Zod/Yup | Consistent rules |

---

## 4. Implementation

```tsx
function useAutosave<T>(initial: T) {
  const [value, setValue] = React.useState<T>(initial);
  const [status, setStatus] = React.useState<"idle"|"saving"|"saved"|"error">("idle");
  React.useEffect(() => {
    const id = setTimeout(async () => {
      setStatus("saving");
      try { await fetch("/api/settings", { method: "POST", body: JSON.stringify(value) }); setStatus("saved"); }
      catch { setStatus("error"); }
    }, 500);
    return () => clearTimeout(id);
  }, [value]);
  return { value, setValue, status };
}
```

---

## 5. A11y & UX
- Announce status changes; visible focus; keyboard nav.
- Prevent loss with “unsaved changes” guard on navigation when saving.

---

## 6. Testing
- Autosave debounce; server errors; undo/redo.
- E2E: rapid toggles, network flake.

---

## 7. Interview Summary
> “Debounced autosave with optimistic UI and undo; validation with Zod; clear status feedback and navigation guards protect user trust.”

# Front-End System Design Deep Dive — Autocomplete / Typeahead

> Build an accessible, resilient, and fast **autocomplete** (a.k.a. typeahead / combobox) using **React + TypeScript**. Includes architecture, tech choices, trade-offs, and production-ready code sketches.

---

## 1. Problem Framing (Interview Checklist)
- **Data source**: remote index (REST/GraphQL), latency target (P95 ≤ 300ms), max result count.
- **Behavior**: min characters before query, debounce, highlight, grouping, recent searches.
- **UX**: keyboard (↑/↓/Enter/Esc), mouse, touch, IME composition, RTL.
- **A11y**: WAI-ARIA combobox pattern; screen reader announcements.
- **Perf**: avoid overfetching; cache & cancel stale requests; virtualize long lists.
- **Edge**: offline/spotty network, rate limiting, API errors, partial results.

---

## 2. Architecture (Front End)
```
Autocomplete
 ├─ Input (controlled) + tokens/pills (optional)
 ├─ SuggestionList (virtualized if large)
 ├─ SuggestionItem (highlight + metadata)
 └─ useSuggestions()  ← debounce, cache, cancel, error handling
Global: small LRU for recent queries; feature flags; telemetry
```

**Data flow**: input → debounce → fetch (cancel previous) → cache → render → selection callback → consumer updates state/URL.

---

## 3. Tech Choices & Rationale
| Concern | Option | Why |
|---|---|---|
| Debounce/cancel | `setTimeout` + `AbortController` | Simple, no deps; precise cancellation. |
| Cache | In-memory LRU (Map) or React Query | LRU if widget-level; React Query if app-wide cache/devtools desired. |
| Render list | Plain list vs `react-window` | Use virtualization for 100+ results to avoid jank. |
| Highlight | Lightweight client highlight | Send match ranges from server when possible. |
| Transport | REST | Lower overhead; GraphQL fine if schema already used. |

---

## 4. Implementation (React + TS)

### 4.1 Utilities
```ts
import { useEffect, useMemo, useRef, useState } from "react";

export function useDebounce<T>(value: T, delay = 300): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => { const id = setTimeout(() => setDebounced(value), delay); return () => clearTimeout(id); }, [value, delay]);
  return debounced;
}

class LRU<K, V> {
  private map = new Map<K, V>();
  constructor(private cap = 100) {}
  get(k: K) { const v = this.map.get(k); if (v !== undefined) { this.map.delete(k); this.map.set(k, v); } return v; }
  set(k: K, v: V) { if (this.map.has(k)) this.map.delete(k); this.map.set(k, v); if (this.map.size > this.cap) this.map.delete(this.map.keys().next().value); }
}
```

### 4.2 Data Hook with Debounce + Cancel + Cache
```ts
type Suggestion = { id: string; label: string; subtitle?: string };

const lru = new LRU<string, Suggestion[]>(200);

export function useSuggestions(query: string) {
  const [items, setItems] = useState<Suggestion[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const debounced = useDebounce(query, 250);

  useEffect(() => {
    setError(null);
    if (!debounced || debounced.length < 2) { setItems([]); return; }

    const cached = lru.get(debounced);
    if (cached) { setItems(cached); return; }

    const ac = new AbortController();
    setLoading(true);
    fetch(`/api/search?q=${encodeURIComponent(debounced)}&limit=8`, { signal: ac.signal, headers: { "Accept": "application/json" } })
      .then(r => r.ok ? r.json() : Promise.reject(`${r.status} ${r.statusText}`))
      .then((data: { items: Suggestion[] }) => { lru.set(debounced, data.items); setItems(data.items); })
      .catch(e => { if ((e as any).name !== "AbortError") setError(String(e)); })
      .finally(() => setLoading(false));
    return () => ac.abort();
  }, [debounced]);

  return { items, loading, error };
}
```

### 4.3 Accessible Combobox
```tsx
export function Autocomplete({ onSelect }: { onSelect: (item: { id: string; label: string }) => void }) {
  const [q, setQ] = useState("");
  const [active, setActive] = useState(-1);
  const listId = "ac-list";
  const { items, loading, error } = useSuggestions(q);

  const onKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.nativeEvent.isComposing) return;
    if (e.key === "ArrowDown") { e.preventDefault(); setActive(a => Math.min(a + 1, items.length - 1)); }
    else if (e.key === "ArrowUp") { e.preventDefault(); setActive(a => Math.max(a - 1, -1)); }
    else if (e.key === "Enter" && active >= 0) { onSelect(items[active]); }
    else if (e.key === "Escape") { setQ(""); setActive(-1); }
  };

  return (
    <div role="combobox" aria-expanded={items.length > 0} aria-owns={listId} aria-haspopup="listbox" className="relative w-80">
      <input
        aria-controls={listId}
        aria-activedescendant={active >= 0 ? `${listId}-opt-${active}` : undefined}
        value={q}
        onChange={(e) => setQ(e.target.value)}
        onKeyDown={onKeyDown}
        className="w-full border rounded px-3 py-2"
        placeholder="Search..."
      />
      {loading && <div className="absolute right-2 top-2 text-sm">Loading…</div>}
      {error && <div className="absolute right-2 bottom-2 text-xs text-red-600">{error}</div>}
      {items.length > 0 && (
        <ul id={listId} role="listbox" className="absolute z-10 mt-1 max-h-72 overflow-auto w-full border bg-white rounded shadow">
          {items.map((it, i) => (
            <li
              id={`${listId}-opt-${i}`}
              role="option"
              aria-selected={i === active}
              key={it.id}
              className={`px-3 py-2 cursor-pointer ${i === active ? "bg-gray-100" : ""}`}
              onMouseEnter={() => setActive(i)}
              onMouseDown={() => onSelect(it)}
            >
              <div className="font-medium">{it.label}</div>
              {it.subtitle && <div className="text-sm opacity-70">{it.subtitle}</div>}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## 5. Performance, A11y, Security
- **Perf**: small overscan, memoize rows, throttle hover highlights; avoid layout trashing.
- **A11y**: visible focus, proper roles/aria, announce “N suggestions” in a live region.
- **Security**: escape suggestion labels; never inject HTML from untrusted sources.

---

## 6. Testing Strategy
- Unit: debounce timing, abort on rapid changes, cache hits.
- Integration: keyboard focus flow, aria attributes.
- E2E: slow network, IME composition, RTL.

---

## 7. Interview Soundbite
> “I balance responsiveness and QPS via debounce+cache+cancel, deliver a11y combobox semantics, and virtualize only when necessary.”

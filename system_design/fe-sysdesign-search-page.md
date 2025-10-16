# Front-End System Design Deep Dive — Search Results Page (Faceted Search)

> Build a **faceted search** UI with filters, sort, pagination, debounced inputs, and URL synchronization. React + TypeScript.

---

## 1. Requirements
- Query input; filters (brand, price, tags), sort.
- Server-side pagination; debounced apply.
- URL reflects state; shareable.
- Empty state & spell suggestions.

---

## 2. Architecture
```
SearchPage
 ├─ SearchBar
 ├─ FacetPanel
 ├─ ResultList (virtualized)
 └─ useSearch() → builds querystring, fetches results, syncs URL
```

---

## 3. Tech Choices
| Area | Choice | Why |
|---|---|---|
| Cache | React Query | Keep previous data while fetching |
| URL | History API | Shareable filters |
| Debounce | Custom hook | Reduce QPS |

---

## 4. Implementation

```tsx
function useSearchParamsState() {
  const [state, setState] = React.useState({ q: "", sort: "relevance", facets: {} as Record<string,string[]> });
  React.useEffect(() => {
    const qs = new URLSearchParams();
    if (state.q) qs.set("q", state.q);
    qs.set("sort", state.sort);
    Object.entries(state.facets).forEach(([k, arr]) => arr.forEach(v => qs.append(k, v)));
    history.replaceState(null, "", `?${qs.toString()}`);
  }, [state]);
  return [state, setState] as const;
}
```

---

## 5. Performance & A11y
- Keep-previous-data to avoid flicker.
- Virtualize large lists; skeletons.
- Keyboard-accessible facets; ARIA for selected filters.

---

## 6. Interview Summary
> “Debounced filters with URL sync; React Query caches pages; virtualized results for performance and accessible facets for usability.”

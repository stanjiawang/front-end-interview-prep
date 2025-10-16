# Front-End System Design Deep Dive — Infinite Scroll Feed (with Ads)

> Design a modern **infinite scroll** content feed with sponsored units (ads), image/video lazy loading, and analytics.

---

## 1. Requirements
- Cursor-based pagination; bidirectional scroll.
- Mixed units: content + ad cards.
- Prefetch next page; lazy render below viewport.
- Track impressions, clicks, dwell time.

---

## 2. Architecture
```
FeedPage
 ├─ VirtualizedList
 ├─ useFeed() → fetch page, merge, dedupe by id
 └─ AdInjector → every N items or based on rules
```

---

## 3. Tech Choices
| Area | Choice | Why |
|---|---|---|
| Virtualization | react-window | Avoid DOM bloat |
| Data | React Query | Cache pages; background refresh |
| Media | IntersectionObserver | Lazy images/video |
| Ads | Client injector + server eligibility | Control cadence |

---

## 4. Implementation

```tsx
// useFeed.ts
export function useFeed() {
  const [items, setItems] = React.useState<any[]>([]);
  const [cursor, setCursor] = React.useState<string | null>(null);
  const loadMore = async () => {
    const res = await fetch(`/api/feed?cursor=${cursor ?? ""}`).then(r => r.json());
    setItems(prev => mergeDedupe(prev, res.items));
    setCursor(res.nextCursor ?? null);
  };
  return { items, loadMore };
}
```

Ad injection strategy: after merge, insert `{type:"ad", id: "ad-"+n}` at slots (e.g., after every 8 items) respecting frequency caps.

---

## 5. Performance & Measurement
- Reserve media aspect ratio to avoid CLS.
- Virtualize; small overscan; prefetch threshold (IO rootMargin).
- Analytics: emit on first visible intersection (once).

---

## 6. Interview Summary
> “Cursor pagination + virtualization + IO-based lazy loading keeps the feed smooth; ad injection follows deterministic slots with impression tracking on first visibility.”

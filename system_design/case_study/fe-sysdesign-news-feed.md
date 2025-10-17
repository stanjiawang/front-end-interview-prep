
# Front‑End System Design Deep Dive — **News Feed (Social/Activity Feed)**

> Goal: Design and implement a **high‑scale news feed** UI (Instagram/Twitter/LinkedIn style) with **infinite scroll**, **rich media**, **reactions**, **save/share**, **personalization hooks**, and **realtime updates**. Tech focus: **React + TypeScript**, with comparisons (RTK Query vs React Query, react‑window vs react‑virtualized, SSR vs SPA) and concrete code.

---

## 1) Problem Framing & Requirements (Interview Flow)
**Clarify**:
- **Content**: text, images, video, link previews; ads or sponsored items? NSFW filters?
- **Scale**: Max feed length in client? P95 update latency target? Peak QPS for fetch?
- **Realtime**: “New posts” banner or auto‑append? WebSocket vs polling?
- **Personalization**: server‑ranked? client‑side reorder? sticky “seen” cursor?
- **Offline**: browse cached items? like/bookmark offline → sync later?
- **A11y/Intl**: keyboard nav, screen readers, RTL.
- **SEO**: Public feeds need SSR? Authenticated feeds: SPA OK.

**Functional**: infinite scroll (bidirectional optional), media previews, reactions, comments, share, save, report, “new items” anchor, inline ads.
**Non‑Functional**: 60fps scroll, LCP < 2.5s, avoid layout shift with skeletons, safe rendering (XSS), telemetry & error tracking.

---

## 2) Architecture (Front End)
```
App Shell
 └─ FeedPage
     ├─ FeedHeader (filters, tabs)
     ├─ FeedList (Virtualized)
     │   ├─ FeedItem (text/media/reactions)
     │   └─ DateDividers / AdCard
     └─ NewItemsToast
Global
 ├─ Query cache (RTK Query / React Query)
 ├─ WS client + EventBus
 ├─ ImageLoader / VideoController
 ├─ Service Worker (offline cache, background sync)
 └─ Telemetry (RUM + logs)
```

**Why this stack**  
- **React + TS**: type‑safe props & DTOs.  
- **Virtualization** (`react-window`) for stable perf with long lists; **dynamic measuring** cache for mixed media.  
- **RTK Query vs React Query**:  
  - RTKQ integrates with Redux easily for **normalized entities**; good for global cache + websocket patches.  
  - React Query is great standalone; powerful cache & devtools.  
  - **Pick**: If app already uses Redux slices or needs global reducers → **RTK Query**. Otherwise **React Query**.
- **SPA**: Auth feed usually not SEO‑critical; persistent WS; faster client nav. If public/SEO heavy → SSR feed shell.

---

## 3) Data Contracts (simplified)
```ts
export interface FeedItem {
  id: string;
  author: { id: string; name: string; avatarUrl?: string };
  body: string; // sanitized markdown-ish
  media?: Array<{ kind: "image" | "video" | "link"; url: string; previewUrl?: string; aspect?: number }>;
  stats: { likes: number; comments: number; shares: number };
  viewer: { liked: boolean; saved: boolean };
  createdAt: string;
}

export interface Page<T> {
  items: T[];
  nextCursor?: string;
}
```

---

## 4) Pagination, Realtime, and Caching
- **Initial page**: `GET /feed?limit=30`.  
- **Infinite scroll**: `?cursor=<serverCursor>`; use **intersection observer** on sentinel.  
- **Realtime**: WS event `feed.new` inserts to top; if user scrolled up, show **NewItemsToast** instead of auto‑scroll.  
- **Cache**: query cache is the source of truth; **idempotent merge** by `id`; **optimistic** reactions.

**Scroll anchoring** for prepend: record first visible item + offset, re‑measure after prepend, restore offset.

---

## 5) Rendering Strategy & Performance
- **Virtualize** mixed heights; **reserve media slots** with aspect ratio boxes to avoid CLS.  
- **Lazy load** images (`loading="lazy"`), **defer video** until near viewport; use preconnect to CDN.  
- **Memoization**: `React.memo` `FeedItem`; keep props stable.  
- **Code‑split**: comments drawer, share sheet, video player.

---

## 6) Security & A11y
- **Sanitize** any rich text on server; escape on client; CSP to disallow inline script.  
- **A11y**: list semantics; keyboard like/expand; alt text for images; visible focus; announce “N new posts”.

---

## 7) Sample Implementation (React + TS)
(See code blocks for FeedList, optimistic reactions, etc.)
```tsx
// FeedList.tsx
import { useEffect, useRef } from "react";
import { FixedSizeList as List } from "react-window";
import AutoSizer from "react-virtualized-auto-sizer";
import { useFeedQuery, useFetchMore } from "../state/feedApi";
import { FeedItemRow } from "./FeedItemRow";

export function FeedList() {
  const { data } = useFeedQuery();
  const fetchMore = useFetchMore();
  const items = data?.items ?? [];

  const sentinelRef = useRef<HTMLDivElement>(null);
  useEffect(() => {
    if (!sentinelRef.current) return;
    const io = new IntersectionObserver((entries) => {
      for (const e of entries) if (e.isIntersecting) fetchMore();
    }, { rootMargin: "600px" });
    io.observe(sentinelRef.current);
    return () => io.disconnect();
  }, [fetchMore]);

  return (
    <div className="h-full">
      <AutoSizer>
        {({ height, width }) => (
          <List height={height} width={width} itemCount={items.length} itemSize={96}>
            {({ index, style }) => <FeedItemRow key={items[index].id} style={style} item={items[index]} />}
          </List>
        )}
      </AutoSizer>
      <div ref={sentinelRef} aria-hidden />
    </div>
  );
}
```

---

## 8) Testing
- E2E: verify infinite scroll, new items banner, optimistic reaction reconciliation.  
- Perf CI: budget CLS/LCP; synthetic scroll jank checks.


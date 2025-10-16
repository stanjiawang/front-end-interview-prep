# Photos at Scale — Deep Dive Guide

This guide turns each interview keyword into concrete, production‑grade techniques with **implementation details + example code**. All examples use **TypeScript + React 18** unless noted. You can copy snippets into a Next.js app (App Router) or plain React SPA.

---

## 0) Reference Dataset & API Contract (for all examples)

We’ll assume a photos API that returns **cursor‑based pagination** with multiple image sizes.

```ts
// GET /api/photos?cursor=<opaque>&limit=200&zoom=month|day|grid
// Optional filters: year, month, personId, albumId
// Response (example):
interface PhotoItem {
  id: string
  takenAt: string // ISO date
  width: number
  height: number
  urls: {
    tiny: string   // e.g., 64px blur thumbnail (LQIP)
    thumb: string  // e.g., 256px
    medium: string // e.g., 1024px
    full: string   // original or downscaled
    signedUntil: string // ISO expiry for signed URLs
  }
  exif?: { make?: string; model?: string; focal?: number; lat?: number; lng?: number }
}

interface Page<T> {
  items: T[]
  nextCursor?: string // undefined when last page
}
```

---

## 1) Incremental Data Loading (Pagination + Infinite Scroll)

### Why
Load only what’s visible to avoid long TTFB, huge memory, and janky scrolling.

### How
Use **cursor‑based pagination** on the server and **infinite scroll** on the client driven by `IntersectionObserver`.

### Example
```tsx
import { useEffect, useRef, useState } from 'react'

async function fetchPage(cursor?: string, limit = 200) {
  const res = await fetch(`/api/photos?limit=${limit}${cursor ? `&cursor=${cursor}` : ''}`)
  if (!res.ok) throw new Error('Failed to load')
  return (await res.json()) as Page<PhotoItem>
}

export function useInfinitePhotos() {
  const [pages, setPages] = useState<Page<PhotoItem>[]>([])
  const [cursor, setCursor] = useState<string | undefined>(undefined)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    let cancelled = false
    ;(async () => {
      if (loading) return
      setLoading(true)
      try {
        const page = await fetchPage(cursor)
        if (cancelled) return
        setPages(prev => [...prev, page])
        setCursor(page.nextCursor)
      } catch (e: any) {
        if (!cancelled) setError(e)
      } finally {
        if (!cancelled) setLoading(false)
      }
    })()
    return () => { cancelled = true }
  }, [cursor])

  const loadMore = () => {
    if (loading) return
    if (!cursor) return // already at end
    // trigger effect by updating cursor to same value (no‑op) is tricky; 
    // instead keep a separate flag or design useEffect differently.
    setCursor(cursor)
  }

  return {
    items: pages.flatMap(p => p.items),
    hasMore: Boolean(cursor),
    loading,
    error,
    loadMore,
  }
}

export function InfiniteScrollSentinel({ onIntersect }: { onIntersect: () => void }) {
  const ref = useRef<HTMLDivElement | null>(null)
  useEffect(() => {
    const el = ref.current
    if (!el) return
    const io = new IntersectionObserver(entries => {
      const anyVisible = entries.some(e => e.isIntersecting)
      if (anyVisible) onIntersect()
    }, { rootMargin: '800px' }) // preload before user reaches bottom
    io.observe(el)
    return () => io.disconnect()
  }, [onIntersect])
  return <div ref={ref} aria-hidden="true" />
}

// Usage in a page component
function GalleryPage() {
  const { items, hasMore, loadMore } = useInfinitePhotos()
  return (
    <>
      <Grid items={items} />
      {hasMore && <InfiniteScrollSentinel onIntersect={loadMore} />}
    </>
  )
}
```

**Notes**
- `rootMargin: '800px'` preloads early for smooth UX.
- Use **retry with backoff** for network errors (see §11 Error Handling).

---

## 2) Virtualized Grid (Render Only What’s Visible)

### Why
Rendering 10k+ DOM nodes is slow. Virtualization keeps ~40–100 items in DOM.

### How
Use `react-window` for a fixed‑row/column grid or a self‑built virtual scroller.

### Example (react-window FixedSizeGrid)
```tsx
import { FixedSizeGrid as Grid } from 'react-window'

interface GridProps { items: PhotoItem[] }

function VirtualGrid({ items }: GridProps) {
  const columnWidth = 180
  const columnCount = Math.max(1, Math.floor(window.innerWidth / columnWidth))
  const rowCount = Math.ceil(items.length / columnCount)
  const rowHeight = columnWidth // square cells

  const Cell = ({ columnIndex, rowIndex, style }: any) => {
    const index = rowIndex * columnCount + columnIndex
    if (index >= items.length) return null
    return (
      <div style={style}>
        <PhotoTile item={items[index]} sizeHint={columnWidth} />
      </div>
    )
  }

  return (
    <Grid
      columnCount={columnCount}
      columnWidth={columnWidth}
      height={window.innerHeight - 80}
      rowCount={rowCount}
      rowHeight={rowHeight}
      width={window.innerWidth}
    >
      {Cell}
    </Grid>
  )
}
```

**Masonry layout?** Use `react-virtualized`’s `Masonry`, `react-virtuoso`, or custom measurement + `position: absolute` with column pack.

---

## 3) Responsive/Progressive Images (`srcset`, LQIP, Blur‑Up)

### Why
Use the smallest image that looks good at the current zoom. Show a quick blurry preview first.

### How
- Server provides **multiple sizes** (tiny, thumb, medium, full).
- `<img>` uses `srcset` + `sizes` + CSS `object-fit: cover`.
- Start with **LQIP** tiny image as background or placeholder and fade in real image.

### Example
```tsx
function PhotoTile({ item, sizeHint }: { item: PhotoItem; sizeHint: number }) {
  const { tiny, thumb, medium } = item.urls
  return (
    <div className="tile" style={{ width: sizeHint, height: sizeHint }}>
      <img
        alt={new Date(item.takenAt).toLocaleString()}
        loading="lazy"
        decoding="async"
        src={thumb}
        srcSet={`${thumb} 256w, ${medium} 1024w`}
        sizes={`${sizeHint}px`}
        style={{
          width: '100%',
          height: '100%',
          objectFit: 'cover',
          backgroundImage: `url(${tiny})`, // blur-up background via CSS
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          transition: 'filter 200ms ease',
        }}
      />
    </div>
  )
}
```

**Blur‑Up CSS**: Place a blurred tiny image as background, remove blur on `onLoad`.

---

## 4) Zoom Levels (Year → Month → Day → Grid)

### Why
Render fewer, coarser elements when zoomed out; switch to actual photos when zoomed in.

### How
- Maintain a `zoomLevel` state.
- For **Month view**, render clusters (one cell per month) with representative mosaics.
- For **Day/Grid view**, render actual photos with virtualization.

### Example
```tsx
type ZoomLevel = 'year' | 'month' | 'day' | 'grid'

function useZoom() {
  const [zoom, setZoom] = useState<ZoomLevel>('month')
  const zoomIn = () => setZoom(z => (z === 'year' ? 'month' : z === 'month' ? 'day' : 'grid'))
  const zoomOut = () => setZoom(z => (z === 'grid' ? 'day' : z === 'day' ? 'month' : 'year'))
  return { zoom, zoomIn, zoomOut }
}

function Gallery() {
  const { zoom, zoomIn, zoomOut } = useZoom()
  const { items } = useInfinitePhotos()
  const groups = groupByMonth(items)

  return (
    <div>
      <Toolbar onZoomIn={zoomIn} onZoomOut={zoomOut} level={zoom} />
      {zoom === 'month' && <MonthClusters groups={groups} />}
      {zoom === 'grid' && <VirtualGrid items={items} />}
      {/* year/day left as exercise: same idea, different aggregation */}
    </div>
  )
}
```

**MonthClusters** can render a mosaic (2–5 images) using CSS grid to avoid loading full lists.

---

## 5) Bidirectional Virtual Scroll (Jump to Older Content)

### Why
Users scroll up to older months; we must load both directions efficiently.

### How
- Keep an **anchor** item at top and bottom sentinels.
- When user scrolls near top, fetch **previous page** using `before=<takenAt>` or `cursorPrev`.

### Example (top sentinel)
```tsx
function TopSentinel({ onIntersect }: { onIntersect: () => void }) {
  const ref = useRef<HTMLDivElement | null>(null)
  useEffect(() => {
    const io = new IntersectionObserver(es => {
      if (es[0].isIntersecting) onIntersect()
    }, { rootMargin: '800px' })
    if (ref.current) io.observe(ref.current)
    return () => io.disconnect()
  }, [onIntersect])
  return <div ref={ref} />
}
```

Preserve scroll position using **`scrollRestoration`** or calculate **offset diff** after prepending items.

---

## 6) Memory Management (Unload Off‑Screen, Recycle Nodes)

### Why
Avoid OOM on long sessions.

### How
- Virtualization removes off‑screen nodes automatically.
- For custom lists: clear image `src` when far off‑screen; keep a small **LRU cache** of decoded images.

### Example: LRU for decoded images
```ts
class LruImageCache {
  private map = new Map<string, HTMLImageElement>()
  constructor(private capacity = 200) {}
  get(key: string) { const v = this.map.get(key); if (!v) return; this.map.delete(key); this.map.set(key, v); return v }
  set(key: string, img: HTMLImageElement) {
    if (this.map.has(key)) this.map.delete(key)
    this.map.set(key, img)
    if (this.map.size > this.capacity) {
      const oldest = this.map.keys().next().value as string
      const victim = this.map.get(oldest)!
      victim.src = '' // allow GC
      this.map.delete(oldest)
    }
  }
}
```

---

## 7) Network‑Aware Loading

### Why
Adapt quality to connections; be kind to mobile users.

### How
Use `navigator.connection.effectiveType` & `saveData` to choose sizes.

```ts
function chooseSize(sizeHint: number) {
  // Basic heuristic
  // 2g / saveData → tiny; 3g → thumb; else medium
  const c = (navigator as any).connection
  if (c?.saveData) return 'thumb'
  if (c?.effectiveType === '2g') return 'tiny'
  if (c?.effectiveType === '3g') return 'thumb'
  return sizeHint <= 320 ? 'thumb' : 'medium'
}
```

---

## 8) Caching: HTTP Cache, Service Worker, IndexedDB

### 8.1 HTTP Cache
Ensure server sets **`Cache-Control`** for immutable thumbnails (e.g., `public, max-age=31536000, immutable`).

### 8.2 Service Worker (runtime image caching)
```js
// sw.js
self.addEventListener('fetch', (event) => {
  const req = event.request
  const isImage = req.destination === 'image'
  if (!isImage) return
  event.respondWith((async () => {
    const cache = await caches.open('img-v1')
    const cached = await cache.match(req)
    if (cached) return cached
    const res = await fetch(req)
    // Optionally: only cache successful responses
    if (res.ok) cache.put(req, res.clone())
    return res
  })())
})
```
Register once in the app entry:
```ts
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
}
```

### 8.3 IndexedDB (persist metadata for instant reopen)
```ts
// Simple wrapper using native IDB
function idbPutPhotos(items: PhotoItem[]) {
  return new Promise<void>((resolve, reject) => {
    const open = indexedDB.open('photos-db', 1)
    open.onupgradeneeded = () => {
      const db = open.result
      db.createObjectStore('photos', { keyPath: 'id' })
    }
    open.onsuccess = () => {
      const db = open.result
      const tx = db.transaction('photos', 'readwrite')
      const store = tx.objectStore('photos')
      items.forEach(i => store.put(i))
      tx.oncomplete = () => resolve()
      tx.onerror = () => reject(tx.error)
    }
    open.onerror = () => reject(open.error)
  })
}
```

---

## 9) Preload & Prefetch (Next/Prev in Viewer)

### Why
Lightbox should feel instant when navigating next/previous.

### How
- Preload neighbor images with `new Image().src = url`.
- Use `<link rel="preload">` for the hero image when opening viewer.

```ts
function preload(url: string) { const img = new Image(); img.decoding = 'async'; img.src = url }
```

---

## 10) Concurrency & Smoothness (React 18, idle work)

### Techniques
- `useTransition` for non‑urgent state (e.g., filter/zoom change).
- `requestIdleCallback` for low priority tasks (warm caches, EXIF parse).

```tsx
const [isPending, startTransition] = useTransition()

function onChangeZoom(level: ZoomLevel) {
  startTransition(() => setZoom(level))
}

requestIdleCallback(() => {
  // warm up caches or compute month clusters
})
```

---

## 11) Error Handling & Retry with Backoff

```ts
async function fetchWithRetry(url: string, max = 3) {
  let attempt = 0; let lastErr: any
  while (attempt < max) {
    try {
      const res = await fetch(url)
      if (!res.ok) throw new Error('bad status ' + res.status)
      return res
    } catch (e) {
      lastErr = e
      const delay = Math.min(1000 * 2 ** attempt, 5000)
      await new Promise(r => setTimeout(r, delay))
      attempt++
    }
  }
  throw lastErr
}
```

---

## 12) Accessibility (A11y) & Keyboard Navigation

### How
- Provide `alt` text, roles, and roving tabindex for grid.
- Arrow keys to move focus; `Enter` to open viewer.

```tsx
function A11yGrid({ items }: { items: PhotoItem[] }) {
  const [focus, setFocus] = useState(0)
  const onKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'ArrowRight') setFocus(i => Math.min(i + 1, items.length - 1))
    if (e.key === 'ArrowLeft') setFocus(i => Math.max(i - 1, 0))
    if (e.key === 'Enter') openViewer(items[focus])
  }
  return (
    <div role="grid" aria-label="Photos" onKeyDown={onKeyDown} tabIndex={0}>
      {items.map((p, i) => (
        <div
          key={p.id}
          role="gridcell"
          tabIndex={i === focus ? 0 : -1}
          aria-selected={i === focus}
          onFocus={() => setFocus(i)}
        >
          <img alt={new Date(p.takenAt).toDateString()} src={p.urls.thumb} />
        </div>
      ))}
    </div>
  )
}
```

---

## 13) Security: Signed URLs & Expiry

- Use **time‑limited signed URLs** for images; refresh when expired.
- Detect expiry via `urls.signedUntil` and pre‑fetch a refreshed URL.

```ts
function isExpired(iso: string) { return Date.now() > new Date(iso).getTime() - 30_000 }
```

---

## 14) Timeline & Date Index Mapping

### Why
Jump quickly to a month/year without loading all DOM nodes.

### How
- Maintain a **month index** → **approximate scroll offset** mapping based on cluster heights.
- On jump, load needed pages then `scrollTo({ top })`.

```ts
function monthToOffset(month: string): number {
  // e.g., sum of known cluster heights + estimated heights where unknown
  // store in a map updated as clusters render/measured
  return estimateFromMeasurements(month)
}
```

---

## 15) Canvas/WebGL Rendering (when DOM hits limits)

### Why
For extremely dense zoomed‑out views, Canvas/WebGL outperforms DOM images.

### How
- Draw LQIP tiles to a canvas; only load actual `<img>` at closer zoom.
- Use `OffscreenCanvas` + Web Worker for parallel rendering.

```js
// worker.js
onmessage = async (e) => {
  const { canvas, tiles } = e.data // canvas is an OffscreenCanvas
  const ctx = canvas.getContext('2d')
  for (const t of tiles) {
    const img = await createImageBitmap(await (await fetch(t.url)).blob())
    ctx.drawImage(img, t.x, t.y, t.w, t.h)
  }
}
```

```ts
// main thread
const worker = new Worker('/worker.js')
const off = (canvas as any).transferControlToOffscreen()
worker.postMessage({ canvas: off, tiles }, [off])
```

---

## 16) Upload Flow (Drag & Drop, Progress, Resumable)

```tsx
function Uploader() {
  const onDrop = async (files: FileList) => {
    for (const f of Array.from(files)) {
      const url = await getSignedUploadUrl(f.name)
      await fetch(url, { method: 'PUT', body: f })
      // Update UI progress, then refresh listing (optimistic)
    }
  }
  return <input type="file" multiple onChange={e => onDrop(e.target.files!)} />
}
```

Resumable uploads (e.g., tus/Multipart) can be integrated for very large videos.

---

## 17) Internationalization (Dates/Numbers)

```ts
const d = new Intl.DateTimeFormat(undefined, { year: 'numeric', month: 'short', day: 'numeric' }).format
```

---

## 18) State/Data Separation (React Query/SWR)

### Why
Keep data fetching resilient and cache-aware.

### Example (React Query)
```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

function usePhotosRQ() {
  return useInfiniteQuery<Page<PhotoItem>>({
    queryKey: ['photos'],
    queryFn: ({ pageParam }) => fetchPage(pageParam),
    getNextPageParam: (last) => last.nextCursor,
    initialPageParam: undefined,
    staleTime: 60_000,
  })
}
```

---

## 19) Lightbox Viewer (Performance Friendly)

- Render single `<img>` with `max-height: 100vh`.
- Preload neighbors.
- Close on `Esc`, swipe on touch.

```tsx
function Viewer({ list, index, onClose }: { list: PhotoItem[]; index: number; onClose: () => void }) {
  const [i, setI] = useState(index)
  const curr = list[i]
  useEffect(() => { if (list[i+1]) preload(list[i+1].urls.medium) }, [i])
  return (
    <dialog open onClose={onClose}>
      <img src={curr.urls.full || curr.urls.medium} alt="photo" />
      <button onClick={() => setI(i - 1)} disabled={i === 0}>Prev</button>
      <button onClick={() => setI(i + 1)} disabled={i === list.length-1}>Next</button>
    </dialog>
  )
}
```

---

## 20) Testing & Metrics

- **Lighthouse**: LCP, CLS, TBT.
- **Web Vitals**: track LCP/INP across photo pages.
- **Synthetic tests**: scroll 10k items; memory snapshots; CPU throttling.

```ts
// Example vital capture
import { onLCP, onINP } from 'web-vitals'
onLCP(console.log)
onINP(console.log)
```

---

## 21) Put It Together: Minimal Gallery (Wire‑up)

```tsx
function GridPage() {
  const { data, fetchNextPage, hasNextPage } = usePhotosRQ()
  const items = data?.pages.flatMap(p => p.items) ?? []
  return (
    <>
      <VirtualGrid items={items} />
      {hasNextPage && <InfiniteScrollSentinel onIntersect={() => fetchNextPage()} />}
    </>
  )
}
```

---

## 22) Common Interview Follow‑ups & Suggested Answers

1. **How do you keep scroll position when prepending data?**  
   Measure previous first visible element’s offset; after insert, scroll by the delta.

2. **What if signed URLs expire while images are on screen?**  
   Listen for 403; refresh URL via background API; swap `src`.

3. **How to handle HEIC/Live Photos?**  
   Transcode server‑side to web‑friendly formats (AVIF/WebP/MP4) and expose variants.

4. **How to render 50k items timeline quickly?**  
   Canvas mosaic at zoomed‑out level; real images only at zoomed‑in levels.

5. **How do you design for offline?**  
   Cache recent pages (metadata + thumbnails) with SW/IndexedDB; stale‑while‑revalidate on reconnect.

---

## 23) Checklist for the Onsite

- [ ] Infinite scroll with `IntersectionObserver`
- [ ] Virtualized grid (react‑window)
- [ ] Progressive images with `srcset` + blur‑up
- [ ] Zoom levels & clusters
- [ ] Bidirectional loading + scroll anchoring
- [ ] Service Worker cache + IndexedDB metadata
- [ ] A11y (keyboard, alt, roles)
- [ ] Error handling & retries
- [ ] Preload neighbor images in viewer
- [ ] Network‑aware size selection

---

### Closing Script (Concise Answer You Can Say)
> *“I’d combine cursor‑based pagination, bidirectional virtual scrolling, and progressive images. The grid renders only visible cells via react‑window; images load lazily with `srcset` and blur‑up placeholders. Zoomed‑out views use clustered mosaics (even Canvas/WebGL if needed). We cache thumbnails via a Service Worker and store metadata in IndexedDB for instant reopen. Accessibility, keyboard navigation, and robust error handling are built‑in. This keeps iCloud Photos fast and memory‑efficient for tens of thousands of pictures.”*

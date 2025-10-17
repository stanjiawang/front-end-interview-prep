# 04 — Web Caching Layers & Performance Engineering (Part 1 of 3)
*(Sections 1–3 | Bilingual Concepts Version)*

---

## 1. Caching Fundamentals

Caching is the process of **storing reusable data closer to the user** to reduce latency, save bandwidth, and minimize origin load.
Efficient caching converts repeated network trips into quick local lookups, creating a massive performance difference in real-world web apps.

> 💡 **中文解释：** 缓存的本质是“就近取数据”。通过在客户端、本地磁盘或 CDN 节点存储常用资源，可以显著减少延迟和服务器压力。

### Key Concepts

| Term | Definition |
|------|-------------|
| **Cache Hit / Miss** | A *hit* serves directly from cache; a *miss* fetches from origin. |
| **Freshness Lifetime** | Duration where cached data is considered valid. |
| **Revalidation** | Checking with server if the cache is still fresh. |
| **Staleness** | Cached data used after expiration. |

### Benefits of Caching

1. **Latency Reduction** – The closer the data, the faster the load.  
2. **Bandwidth Savings** – Shared assets fetched once and reused.  
3. **Scalability** – Reduces origin requests, allowing higher throughput.

> 💡 **中文解释：** 缓存的命中率越高，页面响应越快，同时减轻服务器负载，提高系统可扩展性。

---

## 2. Browser Caching Hierarchy

Modern browsers employ **multiple caching layers**, each optimized for different trade-offs between speed, persistence, and control.

| Layer | Scope | Characteristics | Eviction Policy |
|--------|--------|------------------|-----------------|
| **Memory Cache** | Per-tab | Fastest (RAM); volatile | LRU (Least Recently Used) |
| **Disk Cache (HTTP Cache)** | Cross-session | Persistent, governed by HTTP headers | LRU + storage quota |
| **Service Worker Cache** | Origin scope | Programmable via Cache API | Controlled by developer |
| **Prefetch / Push Cache** | Temporary | Used for preloading / HTTP2 push | Auto-expiry |

> 💡 **中文解释：** 浏览器缓存体系分为内存缓存、磁盘缓存、Service Worker 缓存和预取缓存。每种缓存层在“速度 vs 持久性”之间平衡。

### Cache Key Composition

Browsers derive cache identity using:
```
{URL} + {Request Method} + {Vary Headers}
```
The `Vary` header defines which request headers influence the cache key. Example: `Vary: Accept-Encoding` means responses differ by compression type (gzip vs br).

> 💡 **中文解释：** Vary 头告诉缓存系统哪些请求头会影响响应内容，从而决定缓存键的唯一性。

### Eviction Rules

- **Memory Cache:** cleared when tab closes or memory pressure increases.  
- **Disk Cache:** managed by size quota and LRU algorithm.  
- **Service Worker Cache:** manually controlled by developers via `caches.delete()` or version bumping.

---

## 3. HTTP Caching (Headers & Validation)

HTTP caching defines **how responses are stored, shared, and validated** using response headers. Mastering these headers is essential to fine-tune cache behavior.

### 3.1 Expiration Headers

| Header | Description | Example |
|---------|--------------|---------|
| `Cache-Control` | Primary cache directive | `Cache-Control: public, max-age=86400` |
| `Expires` | Absolute expiry time (legacy) | `Expires: Fri, 18 Oct 2024 07:28:00 GMT` |

Common `Cache-Control` Directives

| Directive | Meaning | 中文说明 |
|------------|----------|-----------|
| `max-age=N` | Valid for N seconds | 缓存有效期 |
| `s-maxage=N` | CDN-specific override | 用于共享缓存 |
| `public` | Allow any cache | 可由代理缓存 |
| `private` | Browser-only | 禁止共享缓存 |
| `no-store` | Never store | 敏感内容 |
| `no-cache` | Must revalidate before use | 使用前需验证 |
| `must-revalidate` | Must obey expiry strictly | 不可使用过期数据 |

> 💡 **中文解释：** Cache-Control 是最核心的缓存控制头，用来定义资源的新鲜度策略。

### 3.2 Validation Headers

Conditional requests avoid full re-downloads when the cache might be stale.

| Header | Description | Example |
|---------|-------------|----------|
| `ETag` | Entity tag; unique resource hash | `ETag: "v1.2.3"` |
| `If-None-Match` | Client sends previous ETag | `If-None-Match: "v1.2.3"` |
| `Last-Modified` | Timestamp of last update | `Wed, 16 Oct 2024 07:28:00 GMT` |
| `If-Modified-Since` | Client’s timestamp validation | `If-Modified-Since: Wed, 16 Oct 2024` |

**Example Validation Flow**
```http
Client → If-None-Match: "v1.2.3"
Server → 304 Not Modified
Client → Serves cached copy
```

> 💡 **中文解释：** ETag 表示资源版本；当浏览器发现资源可能过期，会带上 ETag 进行条件请求。如果版本一致，服务器返回 304 表示可复用缓存。

### Strong vs Weak ETags

Strong ETag compares byte-for-byte; weak ETag (`W/`) compares semantic equivalence.
```http
ETag: W/"v1.2.3"
```

> 💡 **中文解释：** 弱 ETag 允许轻微差异（如换行符不同），在动态页面中更常见。

---

**End of Part 1 (Sections 1–3)**

# 04 — Web Caching Layers & Performance Engineering (Part 2 of 3)
*(Sections 4–6 | Bilingual Concepts Version)*

---

## 4. Service Worker Caching

Service Workers provide **programmable caching** — they intercept network requests and decide whether to serve from cache or network dynamically.

| Strategy | Description |
|-----------|-------------|
| **Cache First** | Use cache, fallback to network |
| **Network First** | Try network first, fallback to cache |
| **Stale-While-Revalidate** | Serve cached, refresh in background |
| **Network Only** | Always fetch new data |

```js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      return cached || fetch(event.request);
    })
  );
});
```

> 💡 **中文解释：** Service Worker 缓存可自定义策略，例如缓存优先、网络优先或过期后更新，从而平衡“速度”和“实时性”。

**Lifecycle Phases**
1. **Install** – pre-cache static assets.
2. **Activate** – clear outdated caches.
3. **Fetch** – intercept network requests.
4. **Update** – replace old version when new SW is available.

> 💡 **中文解释：** Service Worker 有独立的生命周期，更新后不会立即生效，需等待旧实例关闭才能激活新版本。

---

## 5. CDN Architecture

A **Content Delivery Network (CDN)** replicates static assets across globally distributed edge nodes.
When users request content, the CDN routes them to the nearest node, dramatically reducing latency.

| Component | Function |
|------------|-----------|
| **Edge Node** | Nearest server serving cached assets |
| **Origin Server** | Main content source |
| **TTL (Time To Live)** | Duration cached before revalidation |
| **Edge Logic** | Functions like redirects or A/B testing run at the edge |

**Edge Caching Flow**
```
Browser → Edge Node (cache hit?)
       ↳ If miss → Fetch from Origin
       ↳ Store and serve to future users
```

> 💡 **中文解释：** CDN 将资源缓存到全球节点，用户请求时自动定向到最近服务器，大幅降低延迟。

**Advanced Features**
- **Edge Compute:** Run code (e.g., personalization) near users.
- **Origin Shield:** Extra cache layer protecting origin.
- **Tiered Caching:** One edge forwards misses to mid-tier cache.
- **Dynamic Caching:** Cache API responses and JSON payloads when possible.

> 💡 **中文解释：** 现代 CDN 不仅能缓存静态资源，还可通过边缘计算执行逻辑，如定向、重定向和实时 A/B 测试。

---

## 6. Comparing All Caching Layers

| Layer | Scope | Control | Use Case | Example |
|--------|--------|----------|-----------|----------|
| **Browser Memory Cache** | Per-tab | Automatic | Fast reuse in same session | Inline scripts |
| **Disk Cache (HTTP Cache)** | Cross-session | HTTP headers | Long-lived static files | Images, CSS |
| **Service Worker Cache** | Per-origin | Custom logic | Offline or PWA | App shell |
| **CDN Cache** | Global | CDN rules | Edge delivery | Static bundles |

> 💡 **中文解释：** 各缓存层的作用域和控制权不同：浏览器缓存面向客户端会话，Service Worker 提供离线能力，而 CDN 在全球边缘节点提供加速。

**Design Trade-offs**
| Factor | Browser Cache | HTTP Cache | Service Worker | CDN |
|--------|----------------|-------------|----------------|------|
| **Speed** | ⚡️ Fastest | Fast | Medium | Medium (depends on edge) |
| **Control** | Low | Medium | High | High |
| **Persistence** | Volatile | Persistent | Persistent | Distributed |
| **Use Cases** | Page reloads | Static files | PWA, offline | Global acceleration |

> 💡 **中文解释：** 设计缓存策略时需考虑速度、可控性与一致性之间的平衡。CDN 提供最广的分布式缓存，而 Service Worker 在本地控制最灵活。

---

**End of Part 2 (Sections 4–6)**

# 04 — Web Caching Layers & Performance Engineering (Part 3 of 3)
*(Sections 7–9 + Interview Summary | Bilingual Concepts Version)*

---

## 7. Core Web Vitals (CWV)

Core Web Vitals are **Google’s standardized performance metrics** that reflect real user experience.
They measure loading speed, interactivity, and visual stability.

| Metric | Description | Ideal Threshold |
|---------|--------------|-----------------|
| **FCP (First Contentful Paint)** | Time until first content is rendered | < 1.8s |
| **LCP (Largest Contentful Paint)** | Time to render the largest visible element | < 2.5s |
| **CLS (Cumulative Layout Shift)** | Measures layout stability | < 0.1 |
| **TBT (Total Blocking Time)** | Sum of all long tasks blocking main thread | < 200ms |
| **TTI (Time To Interactive)** | Time until page becomes interactive | < 3.8s |

> 💡 **中文解释：** Core Web Vitals 是衡量网页用户体验的关键指标，涵盖加载速度（LCP）、交互延迟（TBT）、视觉稳定性（CLS）等方面。

### How They’re Calculated
- **LCP:** triggered when the largest text or image is painted.  
- **CLS:** calculated as `impact fraction × distance fraction`.  
- **TBT:** total main-thread blocking time between FCP and TTI.  

### Optimization Tips
- **LCP:** preconnect CDNs, optimize images, and use server-side rendering.  
- **CLS:** reserve layout space, avoid late-loading ads.  
- **TBT/TTI:** split bundles, use async/defer for scripts, minimize long tasks.

> 💡 **中文解释：** 优化 LCP 需减少关键资源加载延迟；优化 CLS 需避免布局抖动；优化 TBT 则需降低主线程阻塞，如代码拆分与懒加载。

---

## 8. Performance Optimization Strategies

Modern web performance engineering combines **network-level, render-level, and runtime-level** techniques.

### 8.1 Resource Loading
| Technique | Description |
|------------|-------------|
| **Preload** | Instruct browser to fetch critical resources early. |
| **Prefetch** | Fetch likely-needed resources in background. |
| **HTTP/2 Multiplexing** | Multiple requests per TCP connection. |
| **HTTP/3 (QUIC)** | Uses UDP; faster connection setup. |

> 💡 **中文解释：** preload 用于提前加载关键资源；prefetch 用于预测性加载；HTTP/2 和 HTTP/3 优化传输效率。

**Example**
```html
<link rel="preload" href="/fonts/roboto.woff2" as="font" type="font/woff2" crossorigin>
<link rel="prefetch" href="/next-page.html">
```

### 8.2 Compression and Minification
- Use **Brotli** or **Gzip** compression.  
- Minify JS/CSS/HTML.  
- Remove unused code (tree-shaking).

> 💡 **中文解释：** 压缩和最小化减少传输体积；Brotli 对文本资源的压缩率优于 Gzip。

### 8.3 JavaScript Optimization
- Split large bundles (code splitting).  
- Use `async` or `defer` attributes for non-blocking scripts.  
- Reduce long-running tasks to < 50ms.  
- Memoize heavy calculations.

### 8.4 Rendering Performance
- Use **requestAnimationFrame** for animations.  
- Avoid forced reflows (layout thrashing).  
- Use **will-change** CSS for hardware acceleration.  

> 💡 **中文解释：** 优化渲染性能需减少回流与重绘，使用硬件加速能提升动画流畅度。

---

## 9. Interview Summary

### 9.1 Common System Design Questions
| Question | What It Tests | Hint |
|-----------|----------------|------|
| “Explain browser caching hierarchy.” | Architecture depth | Mention memory, disk, SW, CDN |
| “How do ETag and Cache-Control differ?” | HTTP caching logic | ETag = validation, Cache-Control = policy |
| “What is stale-while-revalidate?” | Async cache pattern | Serve old data, update in background |
| “How to reduce LCP and TBT?” | Performance tuning | Preload, split JS, minimize blocking |
| “Explain how CDN reduces latency.” | Networking layer understanding | Edge nodes + routing |

> 💡 **中文解释：** 面试问题通常考察候选人能否解释多层缓存体系、性能瓶颈与取舍逻辑。

### 9.2 Design Checklist for Answers
1. Start with **Caching Layers** – Browser → HTTP → SW → CDN.  
2. Add **Performance Metrics** – quantify improvement (LCP, CLS).  
3. Discuss **Trade-offs** – freshness vs performance.  
4. Mention **Monitoring** – use RUM + Lighthouse.  
5. End with **Scalability** – global edge strategy.

### 9.3 Key Takeaways
- Combine caching and performance insights for scalable systems.  
- Quantify answers with metrics, not adjectives.  
- Understand browser internals; explain “why,” not just “what.”  

> 💡 **中文解释：** 高级前端系统设计面试中，考官更关注你的推理与权衡能力，而非工具名称。能从“架构层”解释性能，是区分高级工程师的关键。



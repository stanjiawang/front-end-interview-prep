# Web Performance Engineering – Principles, Measurement, and Optimization
---

## Part 1 – Overview & Goals

Modern web performance engineering aims to deliver **fast, smooth, and efficient** user experiences across devices and networks.  
It combines **measurement**, **diagnosis**, and **systematic optimization**.

### 1.1 Why Performance Matters
- **User retention**: every +1s delay reduces conversion by 7–20%.
- **SEO**: Google ranks by Core Web Vitals (LCP, CLS, INP).
- **Accessibility**: fast pages work better on low-end devices.
- **Business metrics**: performance directly correlates with engagement and revenue.

### 1.2 Key Questions
1. How fast does the first meaningful paint occur?
2. How long until the page is interactive?
3. How smooth are animations and scrolling?
4. What consumes the most CPU or network time?
5. How consistent is performance across real users?

---

## Part 2 – How Browsers Work (Foundations)

Understanding the **rendering pipeline** is critical to knowing where optimizations apply.

### 2.1 The Critical Rendering Path

1. **HTML Parsing → DOM Tree**
   - The browser parses HTML sequentially into nodes (Document Object Model).
   - `<script>` tags **block** parsing unless marked `async` or `defer`.

2. **CSS Parsing → CSSOM Tree**
   - CSS rules are parsed and matched to selectors.
   - Until all CSS is loaded, rendering is **blocked** (critical CSS).

3. **DOM + CSSOM → Render Tree**
   - Merges style and structure into a render tree of visible elements.

4. **Layout (Reflow)**
   - Computes geometry (x, y, width, height) for each node.

5. **Paint**
   - Draws each element into layers (color, text, images).

6. **Composite**
   - GPU or compositor thread merges layers for display.

### 2.2 Reflow vs Repaint
| Operation | Trigger | Cost |
|------------|----------|------|
| **Reflow (Layout)** | DOM structure or geometry change (`width`, `display`, font size) | High |
| **Repaint** | Visual change (color, background) without layout | Medium |
| **Composite only** | `transform`, `opacity` | Low (GPU accelerated) |

### 2.3 Threads & Pipelines
- **Main Thread**: executes JS, handles style/layout/paint scheduling.
- **Compositor Thread**: merges GPU layers and handles smooth scrolling.
- **Worker Threads**: run background scripts (Web Workers).

### 2.4 Blocking Scenarios
- JS blocks parsing → use `defer` or `async`.
- CSS blocks first paint → extract **critical CSS** inline.
- Fonts can block text → use `font-display: swap|optional`.

---

## Part 3 – Network & Resource Delivery

### 3.1 The Journey of a Request
1. **DNS lookup** → domain → IP.
2. **TCP handshake** → 3 packets.
3. **TLS negotiation** (HTTPS) → encryption setup.
4. **HTTP request** → send headers/body.
5. **TTFB (Time to First Byte)** → server response latency.

### 3.2 HTTP/2 and HTTP/3 Principles
| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|----------|-----------|--------|--------|
| Connection | 1 per resource | Multiplexed over one | Multiplexed over QUIC (UDP) |
| Header Compression | None | HPACK | QPACK |
| Prioritization | Manual | Weighted streams | Dynamic |
| Latency | High | Low | Lowest (0-RTT TLS) |

HTTP/2 and 3 enable **parallel requests** on a single connection, removing the need for domain sharding.

### 3.3 CDN & Edge Delivery
- **Concept**: serve content from geographically closer servers.
- **Mechanism**:
  - **Edge caching** (HTML, JS, images)
  - **Origin Shielding**
  - **Smart routing**
- **Benefit**: reduced TTFB, fewer round trips, resilience under load.

### 3.4 Compression
- **Gzip**: DEFLATE algorithm (~60–70% savings).
- **Brotli**: newer, context-based compression (15–20% smaller than gzip).
- Enable via server headers:
  ```http
  Content-Encoding: br
  ```

### 3.5 Preload, Preconnect, Prefetch
| Hint | Purpose | Example |
|------|----------|----------|
| **preload** | Force early fetch of critical resource | `<link rel="preload" as="script" href="main.js">` |
| **preconnect** | Warm up TCP/TLS | `<link rel="preconnect" href="https://cdn.example.com">` |
| **prefetch** | Fetch low-priority future resources | `<link rel="prefetch" href="/next-page.js">` |

These hints reduce blocking during the critical rendering path.

---

## Part 4 – Measuring Performance

### 4.1 Lab Tools
1. **Chrome DevTools → Performance Tab**
   - Record, interact, inspect flame chart.
   - Identify long tasks, layout thrashing, paint bottlenecks.

2. **Lighthouse (DevTools or CLI)**
   ```bash
   npx lighthouse https://your-site.com --view
   ```
   Reports FCP, LCP, CLS, TBT, TTI, and improvement hints.

3. **WebPageTest**
   - Simulate different devices, networks.
   - Provides filmstrip and request waterfall.

4. **PageSpeed Insights**
   - Combines Lighthouse (lab) + CrUX (real-world field data).

### 4.2 Real User Monitoring (RUM)
Use `PerformanceObserver` and `web-vitals` to track Core Web Vitals in production:

```js
import { onLCP, onCLS, onINP } from 'web-vitals';

onLCP(console.log);
onCLS(console.log);
onINP(console.log);
```

Collect data by route and device to observe true P75 (75th percentile) metrics.

---

## Part 5 – Front-End Delivery Optimization

### 5.1 Reduce Critical Path
- Inline **critical CSS**.
- Defer or async all non-critical JS.
- Remove unused CSS and JS (tree-shaking).
- Use **code-splitting** by route.

### 5.2 Image Optimization
- Prefer **WebP/AVIF** for better compression.
- Set explicit `width`/`height` to prevent CLS.
- Use `loading="lazy"` for below-the-fold.
- Serve responsive variants via `srcset`/`sizes`.

### 5.3 Font Optimization
- Subset fonts → smaller WOFF2 files.
- Use `font-display: swap|optional`.
- Preload critical fonts:
  ```html
  <link rel="preload" as="font" href="/fonts/inter.woff2" type="font/woff2" crossorigin>
  ```

### 5.4 Server & Cache
- Long-term cache hashed assets (`max-age=31536000, immutable`).
- Use CDN with edge cache.
- Enable Brotli for text resources.

### 5.5 Bundling & Delivery
- **Tree Shaking**: remove dead code using ES module import graph.
- **Lazy Loading**: import modules dynamically.
- **HTTP/2 Push** (deprecated but conceptually replaced by `preload`).

---

## Part 6 – Runtime & Rendering Optimization

### 6.1 JavaScript Execution
- JS runs on the **main thread** — avoid long blocking tasks.
- Break large loops using `setTimeout(fn, 0)` or `requestIdleCallback`.
- Move heavy computation to **Web Workers**.

### 6.2 Virtual DOM & Reconciliation
In React:
- Diffing compares old/new virtual DOM.
- Minimize re-renders: use `memo`, `useCallback`, `useMemo`.
- Batch state updates to reduce layout recalculations.

### 6.3 Animation & Scrolling
- Animate **transform** and **opacity** (compositor-friendly).
- Avoid triggering layout (`width`, `top`, `left`, etc.).
- Use `will-change` for upcoming animations.

### 6.4 Layout Containment
Use CSS `contain` to limit reflow scope:
```css
.card {
  contain: layout paint;
}
```

### 6.5 Virtualization for Large Lists
Render only visible items:
```js
const visibleItems = items.slice(start, end);
```
Libraries: `react-window`, `react-virtualized`.

---

## Part 7 – Monitoring & Continuous Improvement

### 7.1 Lighthouse CI (Automated Budgets)
```json
{
  "ci": {
    "collect": { "staticDistDir": "./dist" },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "total-blocking-time": ["error", { "maxNumericValue": 300 }]
      }
    }
  }
}
```

### 7.2 RUM Dashboard
Aggregate Web Vitals via analytics:
- Report LCP, INP, CLS by route and device.
- Alert when metrics regress by >10% week-over-week.

### 7.3 Performance Budget Examples
| Metric | Target | Notes |
|--------|--------|-------|
| LCP | < 2.5s | Critical content visible |
| INP | < 200ms | Input responsiveness |
| CLS | < 0.1 | Layout stability |
| TBT | < 300ms | Script execution |
| JS Bundle | < 250KB | gzip size |

---

## Part 8 – Summary: Systematic Performance Strategy

| Phase | Tools | Outcome |
|-------|--------|----------|
| **Measurement** | DevTools, Lighthouse, RUM | Baseline metrics |
| **Diagnosis** | Flame charts, request waterfalls | Identify root causes |
| **Optimization** | Code splitting, image optimization, caching | Reduce bottlenecks |
| **Verification** | Repeat tests, field data validation | Confirm improvement |
| **Automation** | Lighthouse CI + RUM dashboards | Prevent regressions |

---

## Appendix – Mental Model of the Rendering Pipeline

```text
HTML → DOM
CSS → CSSOM
DOM + CSSOM → Render Tree
Render Tree → Layout
Layout → Paint
Paint → Composite → Display
```

**Goal**: shorten and streamline this pipeline.  
Every extra render-blocking script, stylesheet, or reflow slows the path.

---

**Final takeaway**  
Performance is not a one-time fix — it’s an engineering discipline.  
The winning formula is:

> **Measure → Understand → Optimize → Automate → Repeat**

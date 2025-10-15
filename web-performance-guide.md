# Web Performance Engineering – Principles, Measurement, and Optimization

> Audience: Front-end engineers, performance specialists, and web architects  
> Purpose: Understand not only how to **test** performance but also how to **improve** it — with a solid grasp of **browser internals**, **network behavior**, and **runtime optimization principles**.

---

## 📘 Table of Contents
1. [Part 1 – Overview & Goals](#part-1--overview--goals)
2. [Part 2 – How Browsers Work (Foundations)](#part-2--how-browsers-work-foundations)
3. [Part 3 – Network & Resource Delivery](#part-3--network--resource-delivery)
4. [Part 4 – Measuring Performance](#part-4--measuring-performance)
5. [Part 5 – Front-End Delivery Optimization](#part-5--front-end-delivery-optimization)
6. [Part 6 – Runtime & Rendering Optimization](#part-6--runtime--rendering-optimization)
7. [Part 7 – Monitoring & Continuous Improvement](#part-7--monitoring--continuous-improvement)
8. [Part 8 – Core Web Vitals Explained](#part-8--core-web-vitals-explained)
9. [Part 9 – Summary: Systematic Performance Strategy](#part-9--summary-systematic-performance-strategy)
10. [Appendix – Rendering Pipeline](#appendix--rendering-pipeline)

---

## Part 1 – Overview & Goals

Modern web performance engineering aims to deliver **fast, smooth, and efficient** user experiences across devices and networks.  
It combines **measurement**, **diagnosis**, and **systematic optimization**.

### Why Performance Matters
- **User retention**: every +1s delay reduces conversion by 7–20%.
- **SEO**: Google ranks by Core Web Vitals (LCP, CLS, INP).
- **Accessibility**: fast pages work better on low-end devices.
- **Business metrics**: performance directly correlates with engagement and revenue.

---

## Part 2 – How Browsers Work (Foundations)

Understanding the **rendering pipeline** is critical to knowing where optimizations apply.

### The Critical Rendering Path
1. **HTML Parsing → DOM Tree**
2. **CSS Parsing → CSSOM Tree**
3. **DOM + CSSOM → Render Tree**
4. **Layout (Reflow)**
5. **Paint → Composite → Display**

### Threads & Pipelines
- **Main Thread**: executes JS, handles style/layout/paint scheduling.
- **Compositor Thread**: merges GPU layers and handles smooth scrolling.
- **Worker Threads**: run background scripts (Web Workers).

---

## Part 3 – Network & Resource Delivery

### The Journey of a Request
1. **DNS lookup → TCP handshake → TLS negotiation → HTTP request → TTFB**

### HTTP/2 and HTTP/3
| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|----------|-----------|--------|--------|
| Multiplexing | ❌ | ✅ | ✅ |
| Header Compression | ❌ | HPACK | QPACK |
| Transport | TCP | TCP | QUIC (UDP) |

### CDN & Compression
- **Edge caching** reduces latency.
- **Brotli** compression provides 15–20% smaller text payloads than Gzip.

---

## Part 4 – Measuring Performance

### Lab Tools
- Chrome DevTools Performance tab
- Lighthouse / PageSpeed Insights
- WebPageTest

### Field Tools (RUM)
```js
import { onLCP, onCLS, onINP } from 'web-vitals';
onLCP(console.log);
onCLS(console.log);
onINP(console.log);
```

---

## Part 5 – Front-End Delivery Optimization

### Reduce Critical Path
- Inline critical CSS.
- Async/defer non-critical JS.
- Tree-shake and code-split.

### Image Optimization
- Use WebP/AVIF.
- Add width/height to prevent CLS.
- Lazy-load below-the-fold.

### Font Optimization
- Subset fonts, preload critical ones.

---

## Part 6 – Runtime & Rendering Optimization

- Break large JS tasks using `setTimeout` or `requestIdleCallback`.
- Use Web Workers for CPU-bound work.
- Animate transform/opacity only.
- Use `contain` and virtualization for large lists.

---

## Part 7 – Monitoring & Continuous Improvement

### Lighthouse CI Example
```json
{
  "ci": {
    "collect": { "staticDistDir": "./dist" },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }]
      }
    }
  }
}
```

### RUM Dashboards
Track LCP, INP, CLS by route and device.

---

## Part 8 – Core Web Vitals Explained

| Metric | Full Name | What It Measures | Ideal Value | User Experience |
|---------|------------|------------------|--------------|-----------------|
| **FCP** | First Contentful Paint | Time until first visible content (text/image) appears | < 1.8s | "White screen ends" |
| **LCP** | Largest Contentful Paint | Time until main visible element (hero image, headline) finishes rendering | < 2.5s | "Main content appears" |
| **CLS** | Cumulative Layout Shift | Measures unexpected layout shifts during load | < 0.1 | "Page doesn’t jump" |
| **TBT** | Total Blocking Time | Total time main thread is blocked by JS >50ms | < 300ms | "No lag or freeze" |
| **TTI** | Time To Interactive | Time until page fully responds to input | < 5s | "Page is usable" |

### How They Relate
```text
Navigation Start
│
├── TTFB (Server response)
├── FCP (First visible paint)
├── LCP (Main content visible)
├── TBT (JS blocking period)
└── TTI (Page fully interactive)
```

CLS is independent and tracks throughout the entire loading process.

### How to Improve
- **FCP** → Inline critical CSS, defer JS, reduce TTFB.
- **LCP** → Preload hero images, use WebP, minimize render-blocking CSS.
- **CLS** → Reserve element space, use font-display: swap, avoid DOM insertions.
- **TBT** → Split long tasks, lazy-load scripts, reduce JS bundle size.
- **TTI** → SSR/ISR, delay analytics, chunk non-critical logic.

---

## Part 9 – Summary: Systematic Performance Strategy

| Phase | Tools | Outcome |
|-------|--------|----------|
| Measurement | DevTools, Lighthouse, RUM | Baseline metrics |
| Diagnosis | Flame charts, waterfalls | Root causes |
| Optimization | Code-splitting, caching, image tuning | Faster load |
| Verification | Repeat tests, CrUX validation | Confirm wins |
| Automation | Lighthouse CI + Dashboards | Guardrails |

---

## Appendix – Rendering Pipeline

```text
HTML → DOM
CSS → CSSOM
DOM + CSSOM → Render Tree
Render Tree → Layout
Layout → Paint
Paint → Composite → Display
```

**Goal:** Shorten and streamline this pipeline.  
Each blocking script or reflow extends load time.

---

**Final takeaway**  
Performance is an ongoing engineering discipline:

> **Measure → Understand → Optimize → Automate → Repeat**

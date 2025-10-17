# 14 — Resilient Front-End Systems (学习级详细版 / Full Learning Guide)

> Resilience = Detect → Isolate → Recover → Resume (and Observe).
> 💡 中文要点：高可用前端的核心目标是“发现问题、隔离影响、恢复功能、持续运行，并且可观测”。

---

## 1. Error Boundaries & Recovery Mechanisms（错误边界与恢复机制）

### 1.1 What is an Error Boundary?
**Definition:** A React component that catches errors in its child tree during render, lifecycle methods, and constructors, preventing the whole app from crashing.

> 💡 中文解释：Error Boundary（错误边界）能拦截子树中的渲染错误，展示降级 UI，避免整页崩溃。

**Minimal Implementation:**
```tsx
import React from "react";

type Props = { fallback: React.ReactNode; children: React.ReactNode };
type State = { hasError: boolean; error?: Error };

export class ErrorBoundary extends React.Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // log to monitoring service
    fetch("/log", { method: "POST", body: JSON.stringify({ error, info }) });
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}
```

**Usage:**
```tsx
<ErrorBoundary fallback={<p>Something went wrong. Try refresh.</p>}>
  <CriticalPanel />
</ErrorBoundary>
```

### 1.2 Error Categories & Handling Surface
| Category | Examples | Handling |
|---|---|---|
| Render-time | `throw new Error()` in render | Error Boundary |
| Async/Promise | `fetch().then(() => { throw ... })` | `window.onunhandledrejection` + retry |
| Event handler | `onClick={() => { throw ... }}` | Not caught by Error Boundary → wrap try/catch |
| Global | runtime errors, CSP violations | `window.onerror`, `ErrorEvent`, `Reporting-API` |

**Global Handlers:**
```ts
window.onerror = (msg, src, line, col, err) => {
  report({ type: "onerror", msg, src, line, col, stack: err?.stack });
};

window.onunhandledrejection = (e) => {
  report({ type: "unhandledrejection", reason: String(e.reason) });
};
```

### 1.3 Next.js / React Router Integration
- **Next.js 13+**: `app/segment/error.tsx` 自动捕获子树异常；`reset()` 可一键重置边界状态。
- **React Router v6**: `errorElement` + `useRouteError()` 实现路由级降级 UI。

```tsx
// app/dashboard/error.tsx (Next.js)
"use client";
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <h2>Dashboard failed</h2>
      <pre>{error.message}</pre>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
```

### 1.4 Recovery Patterns（恢复模式）
| Pattern | When to use | Example |
|---|---|---|
| Soft Reload | temporary glitch | re-mount component / `reset()` |
| State Reset | corrupted local state | clear store / `queryClient.resetQueries()` |
| Retry with Backoff | transient network error | exponential backoff |
| Fallback UI | persistent failure | skeleton / cached snapshot |
| Partial Degradation | isolate failing widget | hide module, keep shell |

---

## 2. Network Resilience（网络容错与重试策略）

### 2.1 Exponential Backoff（指数退避）
Avoid hammering servers and improve success probability.

```ts
async function retry<T>(fn: () => Promise<T>, times = 5, base = 300) {
  let attempt = 0, lastErr;
  while (attempt < times) {
    try { return await fn(); } catch (e) {
      lastErr = e;
      const delay = Math.min(5000, base * 2 ** attempt) + Math.random() * 100;
      await new Promise(r => setTimeout(r, delay));
      attempt++;
    }
  }
  throw lastErr;
}
```

> 💡 中文：指数退避通过指数级增加等待时间，叠加抖动（jitter）避免雪崩。

### 2.2 Circuit Breaker（断路器）
Stop calling an unhealthy endpoint temporarily.

```ts
type State = "CLOSED" | "OPEN" | "HALF";
class Circuit {
  private failures = 0; private state: State = "CLOSED"; private openedAt = 0;
  constructor(private threshold = 5, private timeoutMs = 10000) {}
  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === "OPEN") {
      if (Date.now() - this.openedAt > this.timeoutMs) this.state = "HALF";
      else throw new Error("CircuitOpen");
    }
    try {
      const res = await fn();
      this.failures = 0; this.state = "CLOSED"; return res;
    } catch (e) {
      this.failures++;
      if (this.failures >= this.threshold) { this.state = "OPEN"; this.openedAt = Date.now(); }
      throw e;
    }
  }
}
```

> 💡 中文：断路器在失败达到阈值后“打开”，短期内快速失败，保护后端与客户端体验。

### 2.3 Caching & SW Fallback（缓存与离线回退）
**Workbox-style strategy:**
- **Cache First** for static assets (icons, fonts).  
- **Stale-While-Revalidate** for content feeds.  
- **Network First** for dynamic API with offline fallback.

```js
self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);
  if (url.pathname.startsWith("/api/")) {
    event.respondWith(networkFirst(event.request));
  } else if (url.pathname.endsWith(".woff2")) {
    event.respondWith(cacheFirst(event.request));
  }
});

async function cacheFirst(req) {
  const cache = await caches.open("v1");
  return (await cache.match(req)) || fetch(req).then((resp) => (cache.put(req, resp.clone()), resp));
}

async function networkFirst(req) {
  const cache = await caches.open("v1");
  try {
    const resp = await fetch(req);
    cache.put(req, resp.clone());
    return resp;
  } catch {
    return (await cache.match(req)) || new Response(JSON.stringify({ offline: true }), { status: 503 });
  }
}
```

### 2.4 Background Sync（离线队列）
Queue failed POSTs and replay when online.

```js
// pseudo-queue using IndexedDB (conceptual)
async function enqueue(key, payload) { /* save to IDB */ }
self.addEventListener("sync", async (event) => {
  if (event.tag === "retry-queue") event.waitUntil(flushQueue());
});
async function flushQueue() { /* read from IDB and retry */ }
```

> 💡 中文：离线队列将失败写操作记录到本地，网络恢复后批量重放，保证数据最终一致性。

---

## 3. Fault Isolation & Graceful Degradation（故障隔离与优雅降级）

### 3.1 Isolate → Contain → Continue
- **Isolate:** sandbox failing widget (iframe / shadow root / worker).  
- **Contain:** cut integration links (disable cross-app events).  
- **Continue:** keep core features functional.

**Sandbox iframe example:**
```html
<iframe src="/promo-widget" sandbox="allow-scripts allow-same-origin"></iframe>
```
> 💡 中文：使用 sandbox iframe 可限制脚本能力与 DOM 影响范围。

### 3.2 Feature Flags & Canary
Progressively roll out risky features and quickly turn off when metrics degrade.

```ts
type Flags = { newCheckout?: boolean; };
export const flags: Flags = JSON.parse(localStorage.getItem("flags") || "{}");
export const on = (k: keyof Flags) => !!flags[k];
// usage
if (on("newCheckout")) renderNewFlow(); else renderLegacy();
```

**Remote-controlled flags** (LaunchDarkly/Unleash) allow instant kill-switch.

### 3.3 Micro-Frontends Resilience
- Host owns routing, auth, and **shell health checks**.  
- Each remote can be **lazy-loaded** with timeout and fallback.  
- Circuit breaker per remote: disable failing remote and show static fallback.

```tsx
function Remote({ url, scope, module }: { url: string; scope: string; module: string }) {
  const [Comp, setComp] = React.useState<React.ComponentType | null>(null);
  React.useEffect(() => {
    let timeout = setTimeout(() => setComp(() => () => <Fallback />), 3000);
    loadRemote(url, scope, module).then((m) => { clearTimeout(timeout); setComp(() => m.default); })
      .catch(() => setComp(() => () => <Fallback />));
  }, [url, scope, module]);
  return Comp ? <Comp /> : <Skeleton />;
}
```

### 3.4 Graceful Degradation Patterns
| Pattern | UX | Implementation |
|---|---|---|
| Skeleton | perceived fast | lightweight skeleton components |
| Static Fallback | reliability | cached SSR HTML / last good snapshot |
| Read-Only Mode | availability | disable write actions, show banner |
| Reduced Quality | performance | lower-res images, fewer animations |

---

## 4. Observability & Self-Healing（可观测性与自愈）

### 4.1 Observability Pillars
- **Logs**: error/warn/info with context (user, page, version).  
- **Metrics**: counters (error rate), timers (latency), gauges (queue size).  
- **Traces**: request spans across browser → API → DB.

**Client log schema:**
```json
{
  "ts": 1699999999999,
  "level": "error",
  "name": "CheckoutError",
  "message": "POST /api/pay failed",
  "userId": "u_123",
  "release": "web@2.4.1",
  "context": { "route": "/checkout", "status": 503 }
}
```

### 4.2 Web Vitals + Error Correlation
Send **LCP/INP/CLS** together with error spans to correlate performance and errors.

```ts
import { onLCP, onINP, onCLS } from "web-vitals";
const send = (metric: any) => navigator.sendBeacon("/vitals", JSON.stringify(metric));
onLCP(send); onINP(send); onCLS(send);
```

### 4.3 Self-Healing Playbook
| Trigger | Self-Heal Action |
|---|---|
| Spike in 5xx | Enable circuit breaker + switch to cached API |
| High INP | Reduce expensive JS / pause non-critical tasks |
| Token expired | Silent refresh token / re-login prompt |
| Memory leak | Force unmount subtree / hard reload prompt |

**Token refresh example:**
```ts
async function fetchWithRefresh(input: RequestInfo, init?: RequestInit) {
  let res = await fetch(input, addAuth(init));
  if (res.status === 401) {
    await refreshToken(); // silent refresh
    res = await fetch(input, addAuth(init));
  }
  return res;
}
```

---

## 5. Interview-Oriented Section（面试导向）

### 5.1 “Design a Resilient Frontend for Checkout”
**Answer Outline:**
- Error boundaries for payment widgets, fallback to static QR.  
- Network resilience: retry with backoff, circuit breaker to legacy API.  
- Graceful degradation: read-only order history when API slow.  
- Observability: vitals + error logs + alerting.  
- Self-healing: auto token refresh, re-mount failing widget.

### 5.2 Common Q&A
**Q: How do you handle intermittent API failures?**  
A: Backoff retries + idempotent endpoints + circuit breaker + SW cache fallback.

**Q: What is graceful degradation vs progressive enhancement?**  
A: Degradation keeps core features working under failure; enhancement adds advanced features when capabilities allow.

**Q: How to isolate faults in micro-frontends?**  
A: Lazy-load remotes with timeouts, sandbox, per-remote circuit breaker, and disable broadcast events.

---

## 6. Checklists（核对清单）

### 6.1 Engineering Checklist
- [ ] Error boundaries wrap risky components
- [ ] `window.onerror` / `onunhandledrejection` wired
- [ ] Retry with exponential backoff + jitter
- [ ] Circuit breaker around critical APIs
- [ ] Service Worker cache + offline queue
- [ ] Feature flags for kill-switch
- [ ] Observability (logs/metrics/traces) shipped
- [ ] Web Vitals + error correlation
- [ ] Self-healing flows (refresh, remount, reset)
- [ ] Runbooks & alerts documented

### 6.2 Security & Privacy
- [ ] Do not log PII in client logs
- [ ] Redact tokens/cookies
- [ ] Honor CSP and Reporting-API

---

## 7. Architecture Diagrams（架构图）

### 7.1 Resilience Flow (ASCII)
```
   Exception/Failure
          │
          ▼
   Detect & Capture (EB, onerror, RUM)
          │
          ├─ Isolate (sandbox/flag off)
          │
          ├─ Recover (retry/backoff/SW cache)
          │
          ├─ Resume (fallback UI / read-only mode)
          │
          └─ Observe (logs, metrics, traces, alerts)
```

### 7.2 Observability Pipeline
```
Browser SDK → Ingestion API → Queue(Kafka) → Store → Dashboard(Grafana)
                          ↘ Alerts (Slack/PagerDuty)
```

---

## 8. Appendix（附录）

### 8.1 Useful Headers
- `Retry-After` for 429/503 backpressure  
- `Cache-Control: stale-while-revalidate`  
- `Content-Security-Policy` + `Report-To`

### 8.2 Libraries
- **Retry/Backoff:** ky, axios-retry  
- **Circuit Breaker:** opossum (Node), custom client impl  
- **SW/Offline:** Workbox  
- **Flags:** LaunchDarkly, Unleash  
- **Monitoring:** Sentry, Datadog, OpenTelemetry

---

**EOF — Chapter 14: Resilient Front-End Systems**

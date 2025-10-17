# 08 — Front-End Observability & Monitoring (Part 1 of 4)
*(Sections 1–2 Deep Dive | Extended Bilingual Concepts Version)*

---

## 1. Observability Fundamentals — Deep Dive

Observability is the capability to **infer internal system state from external outputs**. In front-end architecture, those outputs are primarily: **metrics, logs, traces, errors, user actions, and network timings**.

> 💡 **中文解释：** 可观测性强调“通过外部信号推断内部状态”。前端的信号包含性能指标、日志、追踪、错误与网络时序等。

### 1.1 Why Observability Matters for Front-End

1. **User Experience (UX) Reality** — Lab scores differ from real devices and networks.  
2. **Release Safety** — Detect performance regressions and crash spikes immediately.  
3. **Ownership** — Tie issues to code changes (commit SHA) and owners.  
4. **Business Impact** — Correlate performance with conversion/retention.

> 💡 **中文解释：** 实验室分数（Lighthouse）与真实用户体验（RUM）可能差异很大；必须将性能与业务指标相关联。

### 1.2 Observability Topology (Front-End)

```
Browser SDK (RUM, Errors, Web Vitals, Traces)
   └─ Batch Queue (IndexedDB / Memory) → Beacon / Fetch
         └─ Ingest API (Auth, Schema Validation)
               └─ Stream Processor (Kafka/Kinesis)
                     ├─ Metrics DB (TSDB: Prometheus/ClickHouse)
                     ├─ Logs Store (Elastic/Loki)
                     ├─ Traces Store (Jaeger/Tempo)
                     └─ Data Lake (S3/GCS) → BI/ML
```

> 💡 **中文解释：** 前端采集 SDK 将数据批量发送到采集端点（Ingest API），后端再写入指标库、日志库与追踪库。

---

## 2. Metrics — Web Vitals & Browser Timing APIs (Deep)

### 2.1 Core Web Vitals (Current)

| Metric | What it Measures | Good | Needs Improvement | Poor |
|--------|-------------------|------|-------------------|------|
| **LCP (Largest Contentful Paint)** | Loading speed of main content | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| **CLS (Cumulative Layout Shift)** | Visual stability | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| **INP (Interaction to Next Paint)** | Overall responsiveness to user input | ≤ 200ms | 200–500ms | > 500ms |

> 💡 **中文解释：** 当前核心指标包括 LCP/CLS/INP。INP 替代 FID，更全面反映交互响应性。

### 2.2 How They’re Computed

- **LCP**: the render time of the largest text or image element visible within the viewport. Browser emits `largest-contentful-paint` entries; the **last** candidate before user interaction is used.  
- **CLS**: sum of **layout shift scores** for unexpected shifts. Each shift score = *impact fraction × distance fraction*.  
- **INP**: the **worst** (or near-worst percentile) latency across all interactions (click, tap, key) from input to next paint; excludes outliers with a percentile (e.g., 98th).

> 💡 **中文解释：** LCP 取视口内最大元素的绘制时间；CLS 是多次布局偏移分数的累加；INP 取多次交互中的较差百分位。

### 2.3 PerformanceObserver Examples

**Collect LCP:**
```js
const po = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  for (const e of entries) {
    // e.renderTime or e.loadTime
    console.log("LCP candidate:", e.startTime, e);
  }
});
po.observe({ type: "largest-contentful-paint", buffered: true });
```

**Collect CLS:**
```js
let cls = 0;
const po = new PerformanceObserver((list) => {
  for (const e of list.getEntries()) {
    if (!e.hadRecentInput) cls += e.value; // ignore user-caused shifts
  }
});
po.observe({ type: "layout-shift", buffered: true });
```

**Collect INP (Event Timing):**
```js
new PerformanceObserver((list) => {
  for (const e of list.getEntries()) {
    if (e.entryType === "event" && e.name !== "pointermove") {
      const latency = e.duration; // processing + next paint
      console.log("Interaction latency:", latency);
    }
  }
}).observe({ type: "event", buffered: true, durationThreshold: 16 });
```

> 💡 **中文解释：** 使用 PerformanceObserver 监听特定条目（LCP、LayoutShift、Event）即可采集核心指标。

### 2.4 Navigation & Resource Timing

- **Navigation Timing v2** — DNS, TCP/TLS, TTFB, DOMContentLoaded, loadEventEnd.  
- **Resource Timing** — Per-resource DNS, TCP, TLS, TTFB, transferSize, encodedBodySize, decodedBodySize.  
- **Long Tasks** — Tasks >50ms block the main thread; high TBT/INP.

**Example:**
```js
performance.getEntriesByType("resource").forEach(r => {
  if (r.initiatorType === "script") {
    console.log(r.name, r.transferSize, r.encodedBodySize);
  }
});
```

> 💡 **中文解释：** 资源级别时序能定位哪个脚本或字体导致延迟；Long Task 直接影响交互响应性。

### 2.5 Business & Custom Metrics

- **Conversion-Funnel Timing** (search → product view → add-to-cart → checkout)  
- **Interaction Success Rate** (e.g., % of clicks leading to expected route)  
- **SPA Route Change Time** (route start to “visually ready”)  

**SPA Route Timing:**  
```js
const markStart = (route) => performance.mark(`route:${route}:start`);
const markEnd   = (route) => performance.mark(`route:${route}:end`);
const measureRoute = (route) => performance.measure(
  `route:${route}:duration`, `route:${route}:start`, `route:${route}:end`
);
```

> 💡 **中文解释：** 自定义业务指标比通用指标更能反映业务健康度。

# 08 — Front-End Observability & Monitoring (Part 2 of 4)
*(Sections 3–4 Deep Implementation | Extended Bilingual Concepts Version)*

---

## 3. Real User Monitoring (RUM) — Implementation

### 3.1 Client SDK Responsibilities

1. **Metrics Collection** — Web Vitals, Nav/Resource Timing, Long Tasks.  
2. **Error Capture** — `onerror`, `unhandledrejection`, `console` patch.  
3. **User & Session** — Anonymous ID, session ID, device, locale, network.  
4. **Batching & Retry** — Queue in memory/IndexedDB; send via **Beacon**/**Fetch**.  
5. **Sampling** — Head-based (random %) + Tail-based (sample slow/error sessions).  
6. **Privacy** — PII masking, allow-list fields, domain restriction.

> 💡 **中文解释：** RUM SDK 需负责采集、批量发送、采样与隐私保护，并保证离线可用（IndexedDB 缓存）。

### 3.2 Data Model (Example JSON)

```json
{
  "ts": 1731701000,
  "app": "shop-web",
  "sessionId": "s_9f8a",
  "userId": null,
  "route": "/product/123",
  "device": { "ua": "Chrome 118", "dpr": 2, "mem": 8 },
  "network": { "downlink": 10, "rtt": 50, "type": "4g" },
  "metrics": {
    "lcp": 2200, "cls": 0.04, "inp": 120,
    "ttfb": 120, "jsBundle": 280000
  },
  "resources": [
    { "name": "/main.js", "ttfb": 40, "transfer": 150000 }
  ],
  "errors": [],
  "log": [{ "level": "info", "msg": "render:product", "t": 12 }],
  "trace": { "traceId": "4bf92f...", "spanId": "00f067..." }
}
```

### 3.3 Batching & Network Send

```js
const queue = [];
function enqueue(ev) {
  queue.push(ev);
  if (queue.length >= 20) flush();
}
function flush() {
  const blob = new Blob([JSON.stringify(queue)], { type: "application/json" });
  if (navigator.sendBeacon) {
    navigator.sendBeacon("/rum/ingest", blob);
  } else {
    fetch("/rum/ingest", { method: "POST", body: blob, keepalive: true });
  }
  queue.length = 0;
}
window.addEventListener("beforeunload", flush);
```

> 💡 **中文解释：** 使用 sendBeacon 在页面关闭时也能可靠发送数据；否则 fallback 到 fetch(keepalive)。

### 3.4 Privacy & Security

- **Do:** Encrypt over HTTPS; mask PII (email, token).  
- **Don’t:** Log sensitive fields or raw HTML content.  
- **Controls:** Content allow-listing; domain-level CORS restrictions.  
- **Compliance:** GDPR/CCPA opt-in; “Do Not Track” handling.

---

## 4. Error & Crash Tracking — Advanced

### 4.1 Capture Sources

- **JS runtime**: `window.onerror`, `unhandledrejection`  
- **Network**: `fetch`/`XMLHttpRequest` wrappers for HTTP status & latency  
- **UI**: React Error Boundaries + boundaries per route/shell  
- **Resource**: `error` events on `<img>`, `<script>`  

**React Error Boundary:**
```jsx
class Boundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(err, info) {
    sendError({ err: String(err), info });
  }
  render() { return this.state.hasError ? <h1>Error</h1> : this.props.children; }
}
```

### 4.2 Source Maps & Symbolication

- Upload source maps at build time tagged with **release version** and **commit SHA**.  
- Store mapping securely; never expose to public CDN.  
- De-minify stack traces on server.

**Build CI (pseudo):**
```bash
export RELEASE=v1.2.3
npm run build
sentry-cli releases new $RELEASE
sentry-cli releases files $RELEASE upload-sourcemaps ./dist --rewrite
sentry-cli releases finalize $RELEASE
```

> 💡 **中文解释：** Source Map 上传与版本管理应在 CI 中自动化并与代码版本绑定，便于快速定位。

### 4.3 Session Replay (Optional)

- Record DOM mutations, inputs (masked), network events.  
- Sampling: capture only error sessions or Lighthouse-poor sessions.  
- Store compressed (gzip/brotli) with strict retention policy.

> 💡 **中文解释：** 会话回放能大幅提升定位效率，但需严格脱敏与采样控制。

# 08 — Front-End Observability & Monitoring (Part 3 of 4)
*(Sections 5 Deep: Distributed Tracing | Extended Bilingual Concepts Version)*

---

## 5. Distributed Tracing — W3C Context & OpenTelemetry

### 5.1 W3C Trace Context

**Headers:**
- `traceparent`: `version-traceid-spanid-flags`  
- `tracestate`: vendor-specific key/value list

**Example:**
```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
tracestate: vendorA=foo,bar=baz
```

> 💡 **中文解释：** W3C 标准化了跨系统传递 trace 的字段，使前端到后端的全链路追踪成为可能。

### 5.2 Front-End Propagation

- Create a root span for **page load** and spans for **API calls**.  
- Inject `traceparent` header into `fetch` requests.  
- Backend propagates context to downstream services (gRPC/HTTP).

**Fetch Wrapper:**
```js
function withTrace(fetchImpl = fetch) {
  return (url, init = {}) => {
    const traceId = genTraceId(); const spanId = genSpanId();
    const headers = new Headers(init.headers || {});
    headers.set("traceparent", `00-${traceId}-${spanId}-01`);
    return fetchImpl(url, { ...init, headers });
  };
}
const tracedFetch = withTrace();
```

### 5.3 OpenTelemetry in Browser

```js
import { WebTracerProvider } from "@opentelemetry/sdk-trace-web";
import { ConsoleSpanExporter, SimpleSpanProcessor } from "@opentelemetry/sdk-trace-base";
const provider = new WebTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.register();
```

**Resource Detection:** attach metadata (service.name, version, deployment).

> 💡 **中文解释：** 在浏览器中使用 OpenTelemetry 可以自动生成加载与资源请求的 Span，并与后端追踪对齐。

### 5.4 Sampling Strategies

- **Head-based sampling**: decide at trace start (fast, but may miss rare problems).  
- **Tail-based sampling**: decide after observing spans (captures anomalies, but costlier).  
- **Rules**: keep all error traces; sample 1% of healthy traces; keep p95 slow ones.

> 💡 **中文解释：** 采样策略平衡成本与洞察：错误与慢请求全量保留，健康请求抽样。

### 5.5 Correlation with Logs & Metrics

- Enrich logs with `traceId`/`spanId`.  
- Attach business dimensions (userId, route, checkoutId).  
- Build **explainable dashboards**: e.g., LCP by route, API latency by region.

**Log Example:**
```json
{ "level":"error", "msg":"payment failed", "traceId":"4bf92f...", "spanId":"00f067..." }
```

> 💡 **中文解释：** 在日志与指标中保留追踪 ID，可实现快速“点选跳转”定位到具体请求路径。

# 08 — Front-End Observability & Monitoring (Part 4 of 4)
*(Sections 6–7 Deep: Logging, Security, Governance, Interview)*

---

## 6. Logging Architecture, Security & Governance

### 6.1 Structured Logging — Front-End Guidelines

- Use JSON format; include `timestamp`, `level`, `component`, `route`, `user/session` (anonymized).  
- Distinguish **operational logs** (errors, warnings) from **business logs** (add-to-cart).  
- Rate-limit identical errors; group by stack fingerprint.

**Logger Utility (TS):**
```ts
type Log = { t: number; level: "info"|"warn"|"error"; comp: string; msg: string; ctx?: any };
const queue: Log[] = [];
export function log(level: Log["level"], comp: string, msg: string, ctx?: any) {
  queue.push({ t: Date.now(), level, comp, msg, ctx });
  if (queue.length > 50) flush();
}
```

### 6.2 Secure Transport & Privacy

- Send via HTTPS with HSTS; allowlisted origins.  
- Sign requests or use mTLS (enterprise).  
- Mask PII & secrets; tokenize user IDs.  
- Implement **data retention** (e.g., 30/90 days).

> 💡 **中文解释：** 日志与追踪数据常包含敏感信息，必须全程加密、脱敏，并设置保留周期。

### 6.3 SLO / SLI / SLA & Alerting

- **SLI** (indicator): e.g., % sessions with LCP < 2.5s.  
- **SLO** (objective): e.g., 95% of sessions LCP < 2.5s.  
- **SLA** (agreement): contractual target (often backend).

**Alert Policy Examples:**
- “LCP poor rate > 10% for 10 mins (by region=SEA)”  
- “Error rate p95 > baseline × 2 after release”

> 💡 **中文解释：** 将性能与稳定性目标量化为 SLO，并用阈值和回归检测触发告警。

### 6.4 Performance Budgets & CI Gates

- Bundle ≤ 300KB; critical path ≤ 5 requests.  
- LCP ≤ 2.5s on mid-tier devices (3G/4G).  
- Lighthouse score ≥ 90 enforced via CI; block merges if violated.

**budgets.json (example):**
```json
{
  "resourceSizes": [{ "resourceType": "script", "budget": 300000 }],
  "timings": [{ "metric": "interactive", "budget": 3800 }]
}
```

---

## 7. Interview-Oriented Summary

### 7.1 System Design: “Build an Observability Platform for a SPA”

**Answer Framework:**
1. **Data** — Web Vitals + Nav/Resource Timing + Errors + Traces.  
2. **SDK** — Batching, Beacon, sampling (head+tail), privacy.  
3. **Pipeline** — Ingest (auth) → stream → TSDB/Logs/Traces.  
4. **Dashboards** — Route-level LCP, API latency, error heatmap.  
5. **Governance** — SLOs, alerting, budgets, release gates.

> 💡 **中文解释：** 按数据、SDK、管线、可视化与治理五个维度组织答案，展示系统化思维。

### 7.2 Troubleshooting Scenario

“LCP suddenly degraded in APAC after deployment.”  
- Check **deploy diff** and **CDN region**.  
- Compare **resource timing** (TTFB ↑ ?) and **image sizes**.  
- Inspect **cache hit ratio** and **edge routing**.  
- Rollback or canary to isolate change.

### 7.3 Key Takeaways

- Combine **RUM + Synthetic + Tracing** for full coverage.  
- Prioritize **INP/LCP/CLS**; track route-level budgets.  
- Enforce **privacy** and **data governance** from day one.  
- In interviews, focus on **trade-offs** and **reasoning**.

---

**End of Document**
# 08 — Front-End Observability & Monitoring (Part 1 of 3)
*(Sections 1–2 | Extended Bilingual Concepts Version)*

---

## 1. Observability Fundamentals

Observability measures how well you can **understand system behavior from its outputs** — metrics, logs, and traces.

> 💡 **中文解释：** 可观测性指通过系统输出（指标、日志、追踪）推断其内部状态的能力，是现代前端架构的重要组成。

### 1.1 The Three Pillars of Observability

| Pillar | Description | Tools |
|---------|--------------|--------|
| **Metrics** | Quantitative measurements (e.g., latency, memory) | Prometheus, Grafana |
| **Logs** | Event-based textual data | Elastic, Loki |
| **Traces** | End-to-end request tracking | OpenTelemetry, Jaeger |

> 💡 **中文解释：** Metrics 关注数值变化；Logs 记录事件；Traces 用于追踪请求链路。三者协同才能全面掌控系统健康。

### 1.2 Observability vs Monitoring

| Concept | Purpose | Analogy |
|----------|----------|----------|
| **Monitoring** | Detect known issues via thresholds | “报警系统” |
| **Observability** | Explore unknown issues via data correlation | “诊断系统” |

> 💡 **中文解释：** 监控用于检测已知问题；可观测性则用于分析未知问题。后者更适合现代前后端分布式架构。

### 1.3 Observability Stack (High-Level)

```
Browser → Collector → Ingest API → Storage → Visualization (Dashboard)
         ↑
  PerformanceObserver, Errors, Logs, Web Vitals
```

> 💡 **中文解释：** 前端可观测体系通常由数据采集（RUM）→ 数据传输（Ingest API）→ 可视化（Dashboard）组成。

---

## 2. Metrics, Logs, and Traces in Front-End

### 2.1 Metrics (性能指标)

Metrics quantify performance over time.

| Metric Type | Example | Collection Method |
|--------------|----------|-------------------|
| **Performance Metrics** | FCP, LCP, CLS | PerformanceObserver |
| **Resource Metrics** | JS bundle size, API latency | Resource Timing API |
| **Business Metrics** | Conversion rate, CTR | Custom instrumentation |

> 💡 **中文解释：** 前端可量化指标包括性能指标、资源指标与业务指标，可通过 Performance API 采集。

**Example: Collect LCP**
```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log("LCP:", entry.startTime);
  }
}).observe({ type: "largest-contentful-paint", buffered: true });
```

---

### 2.2 Logs (事件日志)

Logs provide contextual information about application state.

| Type | Description | Example |
|-------|--------------|----------|
| **Console Logs** | Developer debugging | `console.error("API failed")` |
| **Error Logs** | Captured exceptions | `window.onerror` |
| **Custom Logs** | Business events | `{ event: "checkout", userId: 123 }` |

> 💡 **中文解释：** 日志是系统运行的文本记录，可帮助定位问题和重现事件。生产环境应使用结构化日志。

**Structured Log Example**
```json
{
  "level": "error",
  "timestamp": 1731700000,
  "component": "CheckoutPage",
  "message": "Payment failed",
  "userId": 123
}
```

---

### 2.3 Traces (分布式追踪)

Traces track requests as they move through multiple systems.

**Trace Model:**
```
Trace → Span → Event
```
- **Trace:** entire request lifecycle  
- **Span:** single operation (e.g., fetch call)  
- **Event:** log within span

> 💡 **中文解释：** Trace 表示一次完整请求；Span 表示请求的单步操作；Event 用于记录 Span 内的细节。

**Trace Context Example**
```js
fetch("/api/orders", {
  headers: { "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-00" }
});
```

> 💡 **中文解释：** `traceparent` Header 可在前后端间传递追踪上下文，实现全链路追踪。

# 08 — Front-End Observability & Monitoring (Part 2 of 3)
*(Sections 3–4 | Extended Bilingual Concepts Version)*

---

## 3. Real User Monitoring (RUM) and Synthetic Monitoring

### 3.1 Real User Monitoring (RUM)

RUM collects metrics from **actual users in production**.

| Metric | Description | API Source |
|---------|--------------|-------------|
| **LCP** | Largest content render | PerformanceObserver |
| **FID** | First input delay | Event Timing API |
| **CLS** | Layout stability | LayoutShift entries |

**Example (RUM Collection):**
```js
import { onCLS, onLCP, onFID } from "web-vitals";
onLCP(console.log);
onCLS(console.log);
onFID(console.log);
```

> 💡 **中文解释：** RUM 从真实用户浏览器收集性能数据，比实验环境更贴近真实体验。

**Advantages:**
- Real-world accuracy  
- Measures user context (device, network)

**Disadvantages:**
- Harder to reproduce issues  
- Privacy constraints

---

### 3.2 Synthetic Monitoring

Synthetic monitoring simulates user sessions using bots or CI pipelines (Lighthouse CI, WebPageTest).

| Tool | Description |
|-------|--------------|
| **Lighthouse CI** | Automated performance regression testing |
| **WebPageTest** | Network-level and rendering detail |
| **Sitespeed.io** | Continuous synthetic monitoring |

> 💡 **中文解释：** 合成监控通过脚本模拟访问，可自动化检测性能回退。RUM 与 Synthetic 应结合使用。

**Example Lighthouse CI Config:**
```yaml
ci:
  collect:
    url:
      - https://example.com
  assert:
    performance: ["error", { minScore: 0.9 }]
```

---

## 4. Error & Crash Tracking

### 4.1 JavaScript Error Handling

Global error capture APIs:

```js
window.onerror = (msg, url, line, col, err) => {
  console.error("Global Error:", msg, url, line, col);
};
window.addEventListener("unhandledrejection", e => {
  console.error("Unhandled Promise:", e.reason);
});
```

> 💡 **中文解释：** 使用 `window.onerror` 和 `unhandledrejection` 可捕获未处理异常与 Promise 错误。

### 4.2 Stack Trace & Source Maps

Minified JS → unreadable stack traces; use **source maps** to map errors to original source.

```js
//# sourceMappingURL=main.js.map
```

**Example (Sentry Upload):**
```bash
sentry-cli releases files v1.0 upload-sourcemaps ./dist
```

> 💡 **中文解释：** Source Map 用于将压缩代码错误映射回源码，Sentry 等工具自动处理上传。

### 4.3 Error Aggregation and Alerting

| Tool | Features |
|-------|-----------|
| **Sentry** | Stack trace, session replay, alerts |
| **Datadog** | Correlate logs and metrics |
| **Rollbar** | Realtime error grouping |

**Error Payload Example:**
```json
{
  "type": "TypeError",
  "message": "Cannot read property 'x' of undefined",
  "stack": "main.js:120:15",
  "browser": "Chrome 118",
  "userId": 42
}
```

> 💡 **中文解释：** 错误上报 payload 应包括类型、堆栈、浏览器环境与用户标识，用于快速定位问题。

# 08 — Front-End Observability & Monitoring (Part 3 of 3)
*(Sections 5–6 | Extended Bilingual Concepts Version)*

---

## 5. Distributed Tracing in Front-End Systems

### 5.1 Correlating Frontend and Backend

Distributed tracing links a user’s browser event to backend spans.

**Trace Propagation:**
```
Frontend → API Gateway → Backend Services → Database
     |            |             |
 trace-id       span-id      parent-id
```

> 💡 **中文解释：** 分布式追踪通过 trace-id 将前端请求与后端链路关联，形成完整调用路径。

**OpenTelemetry Example:**
```js
import { WebTracerProvider } from "@opentelemetry/sdk-trace-web";
const provider = new WebTracerProvider();
provider.register();
```

### 5.2 Trace Context Headers

| Header | Purpose |
|---------|----------|
| `traceparent` | Unique request trace ID |
| `tracestate` | Vendor-specific metadata |

**Example:**
```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-00
```

> 💡 **中文解释：** traceparent Header 允许前后端系统共享追踪上下文，实现跨系统可观测性。

---

## 6. Logging Architecture & Security Considerations

### 6.1 Logging Pipeline

```
Frontend (Logs)
   ↓
Collector API
   ↓
Storage (Elastic / Loki)
   ↓
Dashboard (Kibana / Grafana)
```

> 💡 **中文解释：** 日志管线包括采集、传输、存储与展示。常用 Elastic Stack 或 Loki + Grafana 实现。

**Best Practices:**
- Use structured logs (JSON).  
- Include timestamp, userId, sessionId.  
- Mask PII (personally identifiable info).  
- Support sampling to reduce volume.

---

### 6.2 Security & Privacy

1. Mask sensitive fields (`email`, `token`).  
2. Use HTTPS for all log transmissions.  
3. Rotate access keys regularly.  
4. Comply with GDPR / CCPA.

> 💡 **中文解释：** 前端日志应严格保护隐私，传输加密并遵循隐私合规要求。

---

## 7. Interview-Oriented Summary

| Question | What It Tests | Hint |
|-----------|----------------|------|
| “How do you design a front-end observability system?” | Architecture design | Explain metrics/logs/traces pipeline |
| “How do you link frontend actions to backend latency?” | Tracing knowledge | Use trace-id propagation |
| “How do you monitor performance regressions?” | Monitoring strategy | Combine RUM + Lighthouse CI |
| “What metrics would you collect in a SPA?” | Analytical thinking | LCP, FID, CLS, API latency |

**Sample Answer:**
> “I would integrate RUM for real-user metrics, Lighthouse CI for synthetic monitoring, and OpenTelemetry for distributed tracing. Errors are aggregated in Sentry, visualized in Grafana.”

> 💡 **中文解释：** 面试回答应清晰描述监控体系：从数据采集、传输到展示与告警，展示全局设计能力。

---

**End of Document**
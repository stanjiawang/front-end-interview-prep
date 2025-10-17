# 18 — Front-End Observability & Monitoring (学习级详细版 / Full Learning Guide)

---

## 🧠 Overview 概述

Frontend **observability** extends beyond error logs — it’s about understanding **how users experience** your app in production.  
The modern frontend observability stack covers **logs**, **metrics**, **traces**, and **user sessions**.

> 💡 中文：前端可观测性不仅是错误日志，更是了解真实用户体验的体系。它包含日志、指标、追踪与用户行为。

---

## 1. Observability Fundamentals（可观测性基础）

### 1.1 The Three Pillars
| Pillar | Definition | Frontend Example |
|---------|-------------|------------------|
| **Logs** | Event or error messages | ErrorBoundary, window.onerror |
| **Metrics** | Numerical time-series | LCP, INP, CLS, API latency |
| **Traces** | Request path & timing | User click → API → DB trace |

### 1.2 Why It Matters
- Detect regressions before users complain.  
- Correlate UI errors with backend failures.  
- Improve performance budgets using data-driven insights.

**Diagram:**
```
Browser (RUM SDK)
   │
   ▼
Collector (API Gateway)
   │
   ▼
Storage (Time-series DB / Logs)
   │
   ▼
Visualization (Grafana / Datadog / Kibana)
```

> 💡 中文：浏览器端通过 SDK 收集指标 → 传输至日志与指标存储 → 通过可视化面板进行监控。

---

## 2. Logging & Error Capture（日志与错误捕获）

### 2.1 Global Error Handlers
```js
window.onerror = (msg, src, line, col, err) => {
  sendToServer({ type: "error", msg, src, line, col, stack: err?.stack });
};

window.onunhandledrejection = (e) => {
  sendToServer({ type: "promise", reason: e.reason });
};
```

### 2.2 React Error Boundaries
```tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) {
    sendToServer({ error, info });
  }
  render() {
    return this.state.hasError ? <Fallback /> : this.props.children;
  }
}
```

### 2.3 RUM SDK (Real User Monitoring)
```js
function initRUM() {
  addEventListener("error", (e) => report("jsError", e));
  addEventListener("unhandledrejection", (e) => report("promise", e));
  addEventListener("click", (e) => report("click", { target: e.target.tagName }));
}
```

> 💡 中文：RUM SDK 负责捕获运行时错误、Promise 拒绝与用户交互事件。

---

## 3. Metrics & Performance Monitoring（指标与性能监控）

### 3.1 Web Vitals Integration
```js
import { onLCP, onINP, onCLS } from "web-vitals";
const send = (metric) => navigator.sendBeacon("/metrics", JSON.stringify(metric));
onLCP(send); onINP(send); onCLS(send);
```

| Metric | Description | Target |
|---------|-------------|--------|
| **LCP** | Largest contentful paint | < 2.5s |
| **INP** | Input latency | < 200ms |
| **CLS** | Layout stability | < 0.1 |

### 3.2 Custom Metrics
```js
performance.mark("renderStart");
// ... render
performance.mark("renderEnd");
performance.measure("renderTime", "renderStart", "renderEnd");
```

### 3.3 Aggregation & Sampling
- Send 1% of events to reduce network cost.  
- Use `navigator.sendBeacon()` for non-blocking uploads.  
- Aggregate metrics every 10 seconds before sending.

> 💡 中文：前端指标采样上报应轻量且非阻塞。使用 sendBeacon 避免干扰用户体验。

---

## 4. Tracing & Distributed Correlation（分布式追踪）

### 4.1 What is a Trace?
A **trace** links frontend actions to backend operations.

```
Click "Checkout" → API /orders → DB write → Response
```

### 4.2 Trace Context Propagation
Frontend adds **trace ID** headers to all outgoing requests:

```js
const traceId = crypto.randomUUID();
fetch("/api/data", { headers: { "x-trace-id": traceId } });
```

Backend logs include the same trace ID — enabling correlation across systems.

### 4.3 OpenTelemetry Integration
```js
import { WebTracerProvider } from "@opentelemetry/sdk-trace-web";
import { SimpleSpanProcessor } from "@opentelemetry/sdk-trace-base";
import { ConsoleSpanExporter } from "@opentelemetry/sdk-trace-base";

const provider = new WebTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.register();
```

> 💡 中文：通过在前后端统一 trace ID，可在 Grafana Tempo / Jaeger 中查看完整链路。

---

## 5. Visualization & Alerting（可视化与告警）

### 5.1 Dashboard Design
| Tool | Usage |
|------|--------|
| **Grafana** | Metrics dashboard (Web Vitals, latency) |
| **Sentry** | Error aggregation and alerting |
| **Datadog** | Logs, RUM, traces unified view |
| **Prometheus** | Time-series data collection |

**Example Panels:**
- Error Rate (% of sessions)
- LCP / INP trend over time
- JS Bundle size trend
- User geography (heatmap)

### 5.2 Alerting Thresholds
| Metric | Warning | Critical |
|---------|----------|----------|
| LCP | > 3s | > 4s |
| Error Rate | > 1% | > 5% |
| API Latency | > 1s | > 2s |

### 5.3 Correlation Example
```
Error spike → Logs show network 500 → Trace links backend DB timeout.
```

> 💡 中文：前端可观测性与后端指标联动，可快速定位根因并自动触发报警。

---

## 6. Session Replay & Privacy（会话回放与隐私）

### 6.1 Session Replay
Tools like **FullStory**, **Datadog RUM**, or **Sentry Replay** capture DOM mutations and user events for debugging.

### 6.2 Privacy Safeguards
- Mask sensitive fields (`input[type=password]`).  
- Obfuscate PII before upload.  
- Configurable sampling (1% sessions).

```js
if (isSensitiveField(node)) node.textContent = "***";
```

> 💡 中文：会话回放需严格保护隐私，通过掩码与采样控制防止敏感信息泄露。

---

## 7. Interview-Oriented Section（面试导向）

### 7.1 Key Question
**“How would you design an observability system for a large frontend app?”**

**Answer Framework:**
1. Capture runtime errors and Promise rejections.  
2. Collect performance metrics (Web Vitals).  
3. Add trace ID to outbound requests.  
4. Send to centralized collector.  
5. Visualize via Grafana / Sentry.  
6. Set alerting thresholds.

### 7.2 Trade-off Table
| Approach | Pros | Cons |
|-----------|------|------|
| Client-side SDK | Full context | Increases bundle size |
| Sampling | Reduces cost | May lose edge cases |
| Trace correlation | Cross-service insight | Requires backend support |

---

## 🧩 Summary 总结

| Category | Focus | Tools |
|-----------|--------|-------|
| Logs | Error & Event tracking | Sentry, Datadog, custom SDK |
| Metrics | Performance KPIs | Web Vitals, Prometheus |
| Traces | End-to-end correlation | OpenTelemetry, Tempo |
| Visualization | Dashboards | Grafana, Kibana |
| Alerting | Proactive detection | PagerDuty, Slack Webhook |

> 💡 中文总结：前端可观测性让性能、错误与用户体验透明化。它是构建高可靠性系统的关键一环。

---

📘 **Next Chapter → 19. DevOps for Front-End**

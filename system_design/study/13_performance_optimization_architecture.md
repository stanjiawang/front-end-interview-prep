# 13 — Designing for Performance Optimization (Part 1 of 4)
*(Sections 1–2 | Core Metrics & Measurement Tools — Full Detailed Version)*

---

## 1. Core Web Performance Metrics

Web performance is measured using **user-centric metrics** called **Web Vitals**.  
These metrics reflect real-world user experience — not synthetic benchmarks.

| Metric | Full Name | Description | Good | Needs Improvement | Poor |
|--------|------------|--------------|------|------------------|------|
| **FCP** | First Contentful Paint | Time until first DOM element rendered | ≤ 1.8s | 1.8–3.0s | > 3.0s |
| **LCP** | Largest Contentful Paint | Time when main content is visible | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| **CLS** | Cumulative Layout Shift | Layout stability (visual jumps) | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| **INP** | Interaction to Next Paint | Interaction latency | ≤ 200ms | 200–500ms | > 500ms |
| **TBT** | Total Blocking Time | Thread-blocking during load | ≤ 200ms | 200–600ms | > 600ms |
| **TTI** | Time to Interactive | Time to become responsive | ≤ 5s | 5–10s | > 10s |

> 💡 **中文解释：**  
> FCP 表示首个内容渲染时间；LCP 衡量主要内容渲染速度；CLS 表示页面布局抖动；INP 测量交互响应延迟。  

### 1.1 Understanding the Relationships
```
FCP → LCP → TTI → INP
```
- **FCP/LCP** focus on visual feedback (加载阶段)  
- **TBT/TTI** reflect interactivity (脚本执行阶段)  
- **INP** evaluates responsiveness after interaction (运行阶段)

---

## 2. Measurement Tools

### 2.1 Lighthouse
Command-line & GUI tool for lab analysis.

```bash
npx lighthouse https://example.com --view
```
Produces:
- Performance score (0–100)
- Breakdown by opportunities (render-blocking, image compression)
- Supports budgets in CI/CD pipelines

> 💡 **中文解释：** Lighthouse 提供基于实验室的性能检测报告，可与 CI 集成。

---

### 2.2 Real User Monitoring (RUM)

Capture real metrics via the **PerformanceObserver API**:

```js
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === "largest-contentful-paint") {
      console.log("LCP:", entry.startTime);
    }
  }
}).observe({ type: "largest-contentful-paint", buffered: true });
```

> 💡 **中文解释：** PerformanceObserver 可在生产环境监控真实用户性能。

---

### 2.3 Web Vitals Library

```js
import { onCLS, onLCP, onINP } from "web-vitals";
onCLS(console.log);
onLCP(console.log);
onINP(console.log);
```

> 💡 **中文解释：** web-vitals SDK 可将真实指标上报至分析平台（如 Datadog / Grafana）。

# 13 — Designing for Performance Optimization (Part 2 of 4)
*(Sections 3–4 | Loading Optimization — Full Detailed Version)*

---

## 3. Loading Performance

### 3.1 Critical Rendering Path

Goal: Reduce steps to **first paint**.  
Critical Rendering Path (CRP) includes parsing HTML → CSSOM → JS → Render Tree.

**Optimization Checklist:**
- Inline critical CSS.
- Defer non-critical JS.
- Compress & cache assets.
- Reduce DOM depth.

**Example:**
```html
<link rel="preload" as="style" href="critical.css">
<script src="analytics.js" async></script>
```

> 💡 **中文解释：** 关键渲染路径优化可显著降低 FCP/LCP。

---

### 3.2 Code Splitting & Lazy Loading

**React Example:**
```jsx
import { lazy, Suspense } from "react";
const Chart = lazy(() => import("./Chart"));

export default function Dashboard() {
  return (
    <Suspense fallback={<p>Loading Chart...</p>}>
      <Chart />
    </Suspense>
  );
}
```

> 💡 **中文解释：** 按需加载（Lazy Loading）通过拆分包文件减少首屏负载。

---

### 3.3 Resource Hints (Preload, Prefetch)

| Type | Description | Example |
|------|--------------|----------|
| **Preload** | Load critical resource early | `<link rel="preload" as="script" href="/main.js">` |
| **Prefetch** | Load future resource in idle | `<link rel="prefetch" href="/next.js">` |
| **DNS Prefetch** | Resolve domain DNS early | `<link rel="dns-prefetch" href="//cdn.com">` |
| **Preconnect** | Establish TCP+TLS early | `<link rel="preconnect" href="//api.com">` |

> 💡 **中文解释：** 预加载策略通过提前建立连接和加载关键资源提升整体加载速度。

# 13 — Designing for Performance Optimization (Part 3 of 4)
*(Sections 5–6 | Runtime & Rendering Optimization — Full Detailed Version)*

---

## 4. Runtime & Rendering

### 4.1 React Rendering Optimization

**Memoization Example:**
```jsx
const Item = React.memo(({ value }) => <li>{value}</li>);
function List({ items }) {
  return items.map((v) => <Item key={v} value={v} />);
}
```

> 💡 **中文解释：** React.memo 避免相同 props 导致不必要重渲染。

**useCallback Example:**
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => setCount(c => c + 1), []);
  return <button onClick={increment}>{count}</button>;
}
```

---

### 4.2 Virtualized Rendering

Render only visible content in long lists.

```jsx
import { FixedSizeList as List } from "react-window";
<List height={400} itemCount={10000} itemSize={35} width={300}>
  {({ index, style }) => <div style={style}>Row {index}</div>}
</List>
```

> 💡 **中文解释：** 虚拟滚动技术避免渲染整个列表，大幅提升渲染效率。

---

### 4.3 Main Thread Optimization

| Technique | Description | Example |
|------------|-------------|----------|
| **Web Worker** | Run heavy tasks off-thread | `new Worker("calc.js")` |
| **OffscreenCanvas** | Render graphics off main thread | `canvas.transferControlToOffscreen()` |
| **requestIdleCallback** | Schedule low-priority tasks | `requestIdleCallback(fn)` |

**Worker Example:**
```js
// worker.js
onmessage = e => postMessage(e.data * 2);
// main.js
const w = new Worker("worker.js");
w.onmessage = e => console.log(e.data);
w.postMessage(21);
```

> 💡 **中文解释：** Web Worker 可避免阻塞 UI 渲染线程，提升交互流畅度。

# 13 — Designing for Performance Optimization (Part 4 of 4)
*(Sections 7–8 | CI Integration, Monitoring & Interview Practice — Full Detailed Version)*

---

## 5. CI Integration & Monitoring

### 5.1 Lighthouse CI
```yaml
name: Lighthouse Audit
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install
        run: npm ci
      - name: Audit Performance
        run: npx lhci autorun
```

> 💡 **中文解释：** 可将 Lighthouse 集成进 GitHub Actions，用于持续性能审计。

---

### 5.2 Synthetic vs Real Monitoring

| Type | Source | Strength | Limitation |
|------|---------|-----------|-------------|
| **Synthetic** | Simulated env (Lighthouse) | Controlled, reproducible | Not real user data |
| **RUM** | Real User Metrics | Real-world insight | Noisy, needs SDK |

---

## 6. Interview Q&A

### 6.1 “How would you optimize a React app’s performance?”

**Answer Framework:**
1. Optimize load: code-splitting, compression, CDN.  
2. Optimize render: memoization, virtualization.  
3. Optimize runtime: move work to workers.  
4. Optimize delivery: prefetch critical routes.  
5. Measure: Lighthouse + RUM dashboards.

> 💡 **中文解释：** 面试回答要从加载、渲染、运行时三个阶段说明优化策略。

---

### 6.2 “How do you debug performance issues?”

- Use **Performance Tab** → analyze long tasks.  
- Use **React Profiler** → find re-renders.  
- Add **Web Vitals SDK** → track LCP/INP in prod.

---

### 6.3 Takeaways

1. Optimize for **user perception**, not just metrics.  
2. Always **measure first**, optimize later.  
3. Automate performance testing in CI/CD.  
4. Treat performance as a **feature**, not an afterthought.

> 💡 **中文解释：** 性能优化是一项长期工程，应与开发流程集成，实现可观测与持续改进。
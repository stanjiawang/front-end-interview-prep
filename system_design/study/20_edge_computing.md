# 20 — Edge Computing & CDN Optimization (学习级详细版 / Full Learning Guide)

---

## 🧠 Overview 概述

**Edge computing** brings computation closer to the end user — reducing latency, improving reliability, and lowering server load.  
In modern frontend systems, edge-based rendering and CDN optimization are key to fast, scalable web performance.

> 💡 中文：边缘计算让计算更靠近用户，减少延迟并提升可用性。前端中的 Edge 与 CDN 优化是现代性能架构的核心。

---

## 1. Edge Computing Fundamentals（边缘计算基础）

### 1.1 Cloud vs Edge
| Feature | Cloud | Edge |
|----------|--------|------|
| Location | Centralized (data centers) | Distributed (PoPs worldwide) |
| Latency | Higher | Low (near user) |
| Scalability | Vertical | Horizontal, Geo-distributed |
| Use Cases | Heavy computation, storage | Caching, pre-rendering, auth |

**Diagram:**
```
User → Edge Node → Regional Node → Origin Server
```

### 1.2 Edge Network Architecture
- **PoP (Point of Presence):** Edge data centers located globally.  
- **Anycast Routing:** Directs users to nearest PoP.  
- **Edge Function:** Executes JS logic near the user.  

> 💡 中文：CDN 的节点即 Edge 节点，用户请求由最近的节点处理。

---

## 2. CDN Optimization Strategies（CDN 优化策略）

### 2.1 Multi-Layer Cache Hierarchy
```
Browser Cache → CDN Edge Cache → CDN Regional Cache → Origin Server
```

### 2.2 Cache-Control Headers
| Header | Description |
|---------|-------------|
| `Cache-Control` | Defines caching policy |
| `ETag` | Validates cache version |
| `s-maxage` | Shared (CDN) cache lifetime |
| `stale-while-revalidate` | Serve old data while refreshing |

**Example:**
```http
Cache-Control: public, max-age=600, s-maxage=3600, stale-while-revalidate=30
ETag: "v1.4.0"
```

> 💡 中文：通过 Cache-Control 与 ETag 控制缓存有效期与版本校验。

### 2.3 Geo Routing & Content Localization
- **Geo DNS / Anycast IP** routes user to closest edge.  
- **Edge KV Stores** deliver localized config (language, region).  

**Example (Edge KV):**
```js
const lang = request.headers.get("Accept-Language")?.split(",")[0];
const regionConfig = await KV.get(`config:${lang}`);
```

---

## 3. Serverless & Edge Functions（无服务器与边缘函数）

### 3.1 Cloudflare Workers Example
```js
export default {
  async fetch(request) {
    return new Response("Hello from Edge!", { status: 200 });
  },
};
```

### 3.2 Vercel Edge Functions (Next.js Middleware)
```js
// middleware.ts
import { NextResponse } from "next/server";
export function middleware(req) {
  const geo = req.geo;
  if (geo?.country === "CN") return NextResponse.rewrite("/cn");
  return NextResponse.next();
}
```

### 3.3 Edge KV / D1 / Durable Objects
| Storage Type | Description | Use Case |
|---------------|-------------|-----------|
| **KV Store** | Key-value, low-latency | Config, static data |
| **D1 DB** | SQLite on edge | Small datasets |
| **Durable Object** | Stateful logic | Session handling |

> 💡 中文：边缘函数结合持久化存储（KV、Durable Objects）可构建近实时应用。

---

## 4. Performance Optimization at Edge（性能优化）

### 4.1 Prefetching & Early Hints
```http
103 Early Hints
Link: </static/app.js>; rel=preload; as=script
```
> 💡 浏览器在收到 200 前可提前加载关键资源。

### 4.2 Edge SSR & Streaming
- **Edge Rendering:** SSR 执行于最近节点，减少 TTFB。  
- **Streaming SSR:** Send HTML progressively via `ReadableStream`.  

**Next.js Example:**
```tsx
export const runtime = "edge";
export default async function Page() {
  return <div>Hello from Edge SSR!</div>;
}
```

### 4.3 Compute Offload
Offload expensive computations (e.g., markdown parsing, image resize) to edge workers.

```js
import sharp from "sharp";
export default async function handle(req) {
  const img = await sharp(await req.arrayBuffer()).resize(200).toBuffer();
  return new Response(img, { headers: { "Content-Type": "image/png" } });
}
```

> 💡 中文：通过在边缘进行 SSR 与资源处理，可极大降低后端压力。

---

## 5. Monitoring, Security & Observability（监控与安全）

### 5.1 Metrics & Logs
- Edge logs collected via Workers Analytics Engine / Datadog Edge.  
- Log fields: latency, cacheHit, geo, CPUTime.

```js
console.log(JSON.stringify({ latency: 23, cacheHit: true, geo: "SG" }));
```

### 5.2 Security Controls
| Risk | Mitigation |
|------|-------------|
| Edge code injection | Deploy from signed bundles |
| Unauthorized access | JWT / IP allowlists |
| DDoS | Rate limiting per IP / region |

> 💡 中文：边缘安全可通过签名包、访问控制与限流防护实现。

---

## 6. Interview-Oriented Section（面试导向）

### 6.1 Key Question
**“How would you design an edge-aware front-end system?”**

**Answer Outline:**
1. Use CDN caching for static content.  
2. Render at edge using serverless functions.  
3. Store config/data in KV near user.  
4. Implement Geo Routing for localization.  
5. Monitor latency & cache hit ratios.  

### 6.2 Trade-off Table
| Approach | Pros | Cons |
|-----------|------|------|
| Edge SSR | Low latency | Cold start time |
| CDN Cache | Fast delivery | Stale content risk |
| Geo Routing | Personalized | Complexity |
| KV Store | Low latency | Eventual consistency |

---

## 🧩 Summary 总结

| Concept | Focus | Example |
|----------|--------|----------|
| Edge Function | Compute near user | Cloudflare Workers |
| CDN Cache | Reduce latency | CloudFront, Akamai |
| Geo Routing | User proximity | Anycast, DNS Routing |
| Performance | Prefetch & streaming | HTTP 103, Edge SSR |

> 💡 中文总结：边缘计算与 CDN 优化的核心是“近用户执行 + 智能分发”。通过边缘函数、缓存层与地理路由可实现极速加载与高可靠性。

---

📘 **Next Chapter → 21. AI-Assisted Front-End Development**

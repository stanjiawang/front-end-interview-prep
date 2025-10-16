# System Design Deep Dive — Data Visualization Platforms (3 Questions, Full Answers)

This single document provides detailed, production‑grade answers for three front‑end–centric **system design** questions that frequently appear for senior/staff data‑platform roles:

1. **Design a real‑time data visualization dashboard.**  
2. **Design a caching strategy between the Frontend (FE) and the Backend‑For‑Frontend (BFF).**  
3. **Handle latency and rendering consistency when consuming streaming APIs.**

> Brand‑neutral. Code samples are TypeScript/JavaScript and Node/Express for clarity. Replace with your stack as needed.

---

## Table of Contents

- [1) Real‑Time Data Visualization Dashboard](#1-real-time-data-visualization-dashboard)
  - [1.1 Requirements & Constraints](#11-requirements--constraints)
  - [1.2 Reference Architecture](#12-reference-architecture)
  - [1.3 Protocol Choice: Polling vs SSE vs WebSocket](#13-protocol-choice-polling-vs-sse-vs-websocket)
  - [1.4 Backpressure, Ordering, and Fault Tolerance](#14-backpressure-ordering-and-fault-tolerance)
  - [1.5 Data Modeling & Storage](#15-data-modeling--storage)
  - [1.6 Security & Multitenancy](#16-security--multitenancy)
  - [1.7 Observability & Testing](#17-observability--testing)
  - [1.8 Example: Node SSE Endpoint + React FE](#18-example-node-sse-endpoint--react-fe)
- [2) Caching Strategy Between FE and BFF](#2-caching-strategy-between-fe-and-bff)
  - [2.1 Goals & Cache Layers](#21-goals--cache-layers)
  - [2.2 Cache Key Design](#22-cache-key-design)
  - [2.3 TTL, SWR, Validation, and Busting](#23-ttl-swr-validation-and-busting)
  - [2.4 Idempotency & Write Paths](#24-idempotency--write-paths)
  - [2.5 Multi‑Tenant & Privacy Considerations](#25-multi-tenant--privacy-considerations)
  - [2.6 Example: Express Route with ETag + Cache‑Control](#26-example-express-route-with-etag--cache-control)
  - [2.7 Example: FE Cache with SWR‑style Fetcher](#27-example-fe-cache-with-swr-style-fetcher)
- [3) Latency & Rendering Consistency with Streaming APIs](#3-latency--rendering-consistency-with-streaming-apis)
  - [3.1 Snapshot + Delta Model](#31-snapshot--delta-model)
  - [3.2 Event Time, Ordering, Watermarks](#32-event-time-ordering-watermarks)
  - [3.3 Client‑Side Buffering & Reconciliation](#33-client-side-buffering--reconciliation)
  - [3.4 UI Strategies for Progressive Delivery](#34-ui-strategies-for-progressive-delivery)
  - [3.5 Failure Modes & Retries](#35-failure-modes--retries)
  - [3.6 Example: WebSocket Feed with Monotonic Sequence IDs](#36-example-websocket-feed-with-monotonic-sequence-ids)
  - [3.7 Example: UI Consistency with React + D3](#37-example-ui-consistency-with-react--d3)

---

## 1) Real‑Time Data Visualization Dashboard

### 1.1 Requirements & Constraints

**Functional**
- Live charts (time series, counters, top‑N leaderboards).
- Filters (time window, region, product), drill‑down navigation.
- Multi‑tenant isolation; each user sees only permitted data.
- Access via browsers; responsive; a11y friendly.

**Non‑Functional**
- **Low latency** updates (< 1–2s end‑to‑end for live stats).
- **High fan‑out** (many clients) with modest server CPU.
- Resilient to reconnects and network turbulence.
- **Cost‑aware**: minimize redundant computation/transfers.
- Auditable and observable (logs, metrics, traces).

### 1.2 Reference Architecture

- **FE (React + D3)**: renders charts; maintains in‑memory store per panel; debounces expensive layouts; switches to Canvas/WebGL for dense layers.
- **BFF (Node/Express)**: normalizes filter params; enforces authz; queries **pre‑aggregated** stores; publishes updates via **SSE** or **WebSocket**.
- **Aggregation Layer**: stream processor (e.g., Flink/Spark/Beam) or DB continuous aggregates; maintains ready‑to‑serve series.
- **Data Store**: time‑series DB or columnar warehouse; materialized views for windows (1m/5m/1h/day).
- **Cache**: Redis/Memory for hot windows; CDN for static assets.
- **Observability**: tracing FE→BFF→store; chart render timing; error budgets.

### 1.3 Protocol Choice: Polling vs SSE vs WebSocket

| Protocol | Pros | Cons | Best for |
|---|---|---|---|
| **Polling** (interval `fetch`) | Simple; caches work | Latency by interval; wasteful | Low‑change widgets |
| **SSE** (Server‑Sent Events) | One‑way push; text/event‑stream; auto‑reconnect | No binary; no client→server push | Broadcast feeds, metrics |
| **WebSocket** | Full‑duplex; binary; low latency | Stateful; load balancers/trust issues | Bidirectional/controls |

Rule of thumb: start with **SSE** for dashboards (1‑way). Use **WebSocket** when the client must push live controls or needs binary.

### 1.4 Backpressure, Ordering, and Fault Tolerance

- **Backpressure**: throttle server publishes (min interval), coalesce bursts, and **send deltas** not full snapshots.
- **Ordering**: every message carries a **monotonic `seq`** and server timestamp. Drop or reorder on client as needed.
- **Fault Tolerance**: include **last seen seq** in reconnect query; BFF replays missed deltas (bounded window). For long gaps, send a compact snapshot.
- **Idempotency**: messages should be idempotent (re‑applying produces same state).

### 1.5 Data Modeling & Storage

- Store **pre‑aggregated** series: `{ bucketStart, dims..., metrics... }` with partitions by time/tenant.
- Keep multiple granularities (1m/5m/1h) for LOD switching.
- Maintain **top‑N tables** for categories to avoid client sorting on massive sets.
- Ensure **timezone** policy and closed windows (no ragged edges).

### 1.6 Security & Multitenancy

- AuthN: short‑lived JWTs; rotate; scope by tenant/project.
- AuthZ: server‑side filter allowlists; never trust client params.
- Isolation: cache keys include **tenant/user scopes**; avoid cross‑tenant cache bleed.
- Rate limits per token + IP.

### 1.7 Observability & Testing

- Metrics: feed latency, client connect count, bytes/sec, cache hit rate, FE render FPS, chart update time.
- Traces: connect→query→publish→render; tag with tenant & chart id (no PII).
- Tests: load (N clients), chaos (disconnect, reorder), functional (filter sequences), visual regression for charts.

### 1.8 Example: Node SSE Endpoint + React FE

**Server (Node/Express) — SSE**

```ts
import express from "express";
const app = express();

app.get("/events/metrics", (req, res) => {
  // Required SSE headers
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");
  res.flushHeaders?.();

  let seq = 0;
  const timer = setInterval(() => {
    const payload = {
      seq: ++seq,
      ts: Date.now(),
      // delta payload; client merges
      series: [{ bucket: Date.now(), value: Math.random() * 100 }]
    };
    res.write(`event: update
`);
    res.write(`data: ${JSON.stringify(payload)}

`);
  }, 1000);

  req.on("close", () => clearInterval(timer));
});

app.listen(3000);
```

**Client (React) — EventSource**

```tsx
import React, { useEffect, useRef, useState } from "react";

type Update = { seq: number; ts: number; series: { bucket: number; value: number }[] };

export function LivePanel() {
  const [points, setPoints] = useState<{ x: number; y: number }[]>([]);
  const lastSeq = useRef(0);

  useEffect(() => {
    const es = new EventSource("/events/metrics");
    es.addEventListener("update", (ev: MessageEvent) => {
      const msg: Update = JSON.parse(ev.data);
      if (msg.seq <= lastSeq.current) return;  // drop stale/out-of-order
      lastSeq.current = msg.seq;
      setPoints((prev) => [...prev, ...msg.series.map(s => ({ x: s.bucket, y: s.value }))].slice(-3000));
    });
    es.onerror = () => { es.close(); /* let browser reconnect */ };
    return () => es.close();
  }, []);

  return <div>{points.length} points (render with D3/Canvas)</div>;
}
```

---

## 2) Caching Strategy Between FE and BFF

### 2.1 Goals & Cache Layers

**Goals:** reduce latency/cost, smooth load, preserve correctness under freshness constraints.

**Layers (from client to origin):**
1. **Browser HTTP cache** (via `Cache-Control`, `ETag`).  
2. **Service Worker / in‑memory FE cache** (SWR/React Query).  
3. **BFF cache** (Redis/Memory) with normalized keys.  
4. **DB/materialized views** with periodic refresh.

### 2.2 Cache Key Design

Key must uniquely represent the requested **result identity**:

```
cache:v1:tenant={TENANT}:user={USER?}:resource=series:sales:
  window=6m:bucket=1m:region=NA:ver=agg_2025-09-01
```

- Include **tenant**, filter params, and **aggregation version** (schema change bumps version).
- Normalize (sort) unordered params; lowercase where appropriate.
- Keep keys short but explicit; avoid exposing secrets.

### 2.3 TTL, SWR, Validation, and Busting

- **TTL**: short for hot windows (e.g., 10–60s), longer for cold (e.g., 10–30m).
- **SWR (stale‑while‑revalidate)**: serve cached response immediately; revalidate in background and refresh.
- **Validation**: `ETag` (hash of body) + `If-None-Match` → cheap **304**.
- **Busting**: on data ingestion/ETL complete, publish **invalidation events**; BFF deletes matching keys or bumps `ver` prefix.

### 2.4 Idempotency & Write Paths

- Use **Idempotency‑Key** header for POST jobs (report generation). First call computes and caches; retries with same key return the same result.
- For multi‑step jobs: return `202 Accepted` + status URL; FE polls with **ETag/If‑None‑Match** to reduce payload.

### 2.5 Multi‑Tenant & Privacy Considerations

- Never share cache across tenants; bake **tenant id** into every key.
- For user‑scoped data, include **user id** (or role) in the key.
- Sanitize payloads; redact PII before caching.

### 2.6 Example: Express Route with ETag + Cache‑Control

```ts
import crypto from "crypto";
import express from "express";
import Redis from "ioredis";

const app = express();
const redis = new Redis();

function etag(body: string) {
  return `"W/${crypto.createHash("sha1").update(body).digest("base64")}"`;
}

app.get("/api/series/sales", async (req, res) => {
  const tenant = req.header("x-tenant-id") || "default";
  const windowParam = req.query.window || "6m";
  const key = `cache:v1:tenant=${tenant}:series:sales:window=${windowParam}:bucket=1m`;

  let payload = await redis.get(key);
  if (!payload) {
    // ... query DB/materialized view; simplified here
    payload = JSON.stringify({ series: [{ t: "2025-04", value: 123 }] });
    await redis.setex(key, 60, payload); // TTL 60s
  }

  const tag = etag(payload);
  if (req.headers["if-none-match"] === tag) {
    res.status(304).end();
    return;
  }
  res.setHeader("ETag", tag);
  res.setHeader("Cache-Control", "public, max-age=10, stale-while-revalidate=60");
  res.type("application/json").send(payload);
});

app.listen(3000);
```

### 2.7 Example: FE Cache with SWR‑style Fetcher

```tsx
import useSWR from "swr";
const fetcher = (url: string) => fetch(url, { headers: { "x-tenant-id": "acme" } }).then(r => r.json());

export function SalesChart() {
  const { data, error, isLoading, mutate } = useSWR("/api/series/sales?window=6m", fetcher, {
    revalidateOnFocus: true,
    dedupingInterval: 5000, // de-dupe simultaneous requests
    fallbackData: { series: [] }
  });

  if (error) return <div>Error</div>;
  if (isLoading) return <div>Loading…</div>;
  return <div>{data.series.length} points</div>;
}
```

---

## 3) Latency & Rendering Consistency with Streaming APIs

### 3.1 Snapshot + Delta Model

- **Snapshot**: initial compact state (e.g., last 5 minutes of buckets).  
- **Delta**: append‑only updates with `seq` and `eventTime`.  
- **Reconcile**: apply deltas in order; if **gap** detected, request a mini‑snapshot to resync.

### 3.2 Event Time, Ordering, Watermarks

- **Event Time** (when event happened) vs **Processing Time** (server handled).  
- Use **monotonic `seq`** per stream; for sharded sources encode shard id + seq.  
- **Watermarks**: server emits “we’ve seen all events up to T”; client can close buckets ≤ watermark.  
- Late events (< small tolerance) may be merged; beyond tolerance, store in a **corrections** side channel.

### 3.3 Client‑Side Buffering & Reconciliation

- Keep a **ring buffer** (e.g., last N minutes).  
- Buffer deltas for a tiny window (e.g., 200–500ms) to batch and preserve order.  
- On reconnect, include **last `seq`** so server can replay. Fallback: request snapshot.

### 3.4 UI Strategies for Progressive Delivery

- **Skeleton/Placeholder** for initial load; avoid layout shift.  
- **Progressive axes**: render axes/grid first, then the series.  
- **LOD switching**: downsample when zoomed out; fetch finer buckets on zoom in.  
- **Optimistic UI** for user‑triggered changes (toggle lines), later confirm server ack.

### 3.5 Failure Modes & Retries

- **Network drops**: exponential backoff; cap to avoid stampedes.  
- **Out‑of‑order**: hold small buffer; drop too old deltas; request correction snapshot.  
- **Duplicate messages**: idempotent apply (dedupe by `seq`).  
- **Clock skew**: always compute durations relative to server timestamps.

### 3.6 Example: WebSocket Feed with Monotonic Sequence IDs

**Server (pseudo)**

```ts
// On each publish:
const msg = { seq: ++seq, ts: Date.now(), bucket: Date.now(), value: compute() };
socket.send(JSON.stringify(msg));
```

**Client**

```ts
let lastSeq = 0;
const socket = new WebSocket("wss://example.com/ws/metrics");

socket.onmessage = (ev) => {
  const msg = JSON.parse(ev.data);
  if (msg.seq <= lastSeq) return;      // drop stale/dup
  lastSeq = msg.seq;
  applyDelta(msg);
};

socket.onclose = () => {
  // backoff & reconnect; include lastSeq so server can replay
  setTimeout(connect, Math.min(30000, backoffMs *= 2));
};
```

### 3.7 Example: UI Consistency with React + D3

```tsx
import React, { useMemo } from "react";
import * as d3 from "d3";

type Pt = { x: number; y: number };

export function SeriesSvg({ points }: { points: Pt[] }) {
  // Stable scales so minor updates don't thrash layout
  const [xMin, xMax] = useMemo(() => d3.extent(points, d => d.x) as [number, number], [points]);
  const yMax = useMemo(() => d3.max(points, d => d.y) || 0, [points]);

  const x = d3.scaleUtc().domain([new Date(xMin), new Date(xMax)]).range([40, 760]);
  const y = d3.scaleLinear().domain([0, yMax]).nice().range([360, 20]);
  const path = d3.line<Pt>().x(p => x(new Date(p.x))).y(p => y(p.y))(points) || "";

  return (
    <svg viewBox="0 0 800 400" role="img" aria-label="Live series">
      <path d={path} fill="none" stroke="#0d6efd" strokeWidth={2} />
      <g transform="translate(0,360)">{/* x-axis; omit for brevity */}</g>
      <g transform="translate(40,0)">{/* y-axis */}</g>
    </svg>
  );
}
```

**Tips**
- Throttle chart updates (e.g., at most 10–20fps).  
- For >10–20k visible points, switch to Canvas/WebGL.  
- Batch DOM updates; avoid per‑point React elements.  
- Keep GC pressure low; reuse arrays where possible.

---

## Final Takeaways

- **Start simple**: pre‑aggregate and push deltas via SSE; add WebSocket only when needed.  
- **Cache smartly**: design keys, TTL/SWR, and invalidation around the **result identity** and tenant scope.  
- **Stream sanely**: snapshot + ordered deltas + tiny buffer; reconcile with watermarks; keep UI consistent and responsive.


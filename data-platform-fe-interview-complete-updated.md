# Data Platform Front-End Engineer Interview Preparation Pack (Updated)

This guide consolidates **React, D3, Webpack, AI-driven data visualization**, and **system design patterns**
into a single file suitable for senior/staff front-end engineer interview preparation.
All sections are optimized for GitHub rendering and readability.

> Images in Section 2 use **relative PNG paths** (GitHub-safe). Place the PNGs in the **same folder** as this `.md`.

---

## 1) Overview
- Build interactive, high-performance data applications with **React + D3**.
- Integrate **AI intent → SQL/DSL** pipelines via a **Node/Express BFF**.
- Use **Webpack 5** for bundling, splitting, and production optimizations.
- Handle **big data** with pre-aggregation, downsampling, Canvas/WebGL, workers, and virtualization.
- Include an interview Q&A bank and a study roadmap.

---

## 2) Visual Samples

Below are static previews (PNG). Place these PNGs in the **same folder** as this .md file.

- **Combo (bars + moving average)**  
  ![Combo (bars + moving average)](./chart_combo_bars_ma.png)

- **Area with range selection (concept)**  
  ![Area with range selection (concept)](./chart_area.png)

- **Hexbin (density for very large scatter)**  
  ![Hexbin (density for very large scatter)](./chart_hexbin.png)

- **AI data flow (schematic)**  
  ![AI data flow (schematic)](./chart_ai_data_flow.png)

---

## 3) Architecture Overview (Simplified)

```mermaid
flowchart LR
  U[User (Web UI)]
  FE[React App]
  BFF[Node/Express BFF]
  AI[AI Intent Service]
  DW[(Data Warehouse)]
  D3[D3 Visualization Layer]
  CACHE[(Cache)]
  OBS[(Observability)]
  U --> FE
  FE --> BFF
  BFF --> AI
  AI --> BFF
  BFF --> DW
  DW --> BFF
  BFF --> FE
  FE --> D3
  BFF --> CACHE
  BFF --> OBS
```

**Notes**
- FE sends requests with filter params (or NL text).
- BFF handles validation, caching (e.g., Redis), prompt/intent routing, and parameterized SQL to DW.
- FE renders charts; logs UX metrics; traces requests end-to-end.

---

## 4) React + D3 Integration (Minimal Example)

```tsx
import * as d3 from "d3";
import React, { useEffect, useRef } from "react";

export const LineChart: React.FC<{ data: { x: number; y: number }[] }> = ({ data }) => {
  const ref = useRef<SVGSVGElement | null>(null);

  useEffect(() => {
    const svg = d3.select(ref.current);
    const width = 600, height = 300, margin = 40;

    const x = d3.scaleLinear().domain(d3.extent(data, d => d.x) as [number, number]).range([margin, width - margin]);
    const y = d3.scaleLinear().domain([0, d3.max(data, d => d.y)!]).range([height - margin, margin]);

    svg.selectAll("*").remove();
    svg.append("path")
      .datum(data)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 2)
      .attr("d", d3.line<{ x: number; y: number }>().x(d => x(d.x)).y(d => y(d.y)));

    svg.append("g").attr("transform", `translate(0,${height - margin})`).call(d3.axisBottom(x));
    svg.append("g").attr("transform", `translate(${margin},0)`).call(d3.axisLeft(y));
  }, [data]);

  return <svg ref={ref} width={600} height={300} />;
};
```

---

## 5) Webpack 5 Configuration (Simplified Example)

```js
// webpack.config.js
const path = require("path");

module.exports = {
  entry: "./src/index.tsx",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
    clean: true,
    publicPath: "/"
  },
  mode: "production",
  module: {
    rules: [
      { test: /\.tsx?$/, use: "ts-loader", exclude: /node_modules/ },
      { test: /\.s?css$/, use: ["style-loader", "css-loader", "sass-loader"] },
      { test: /\.svg$/, type: "asset/inline" }
    ],
  },
  resolve: { extensions: [".tsx", ".ts", ".js"] },
  optimization: {
    splitChunks: { chunks: "all" },
    runtimeChunk: "single"
  },
  devtool: "source-map",
};
```

**Best Practices**
- Prefer ESM packages and set `"sideEffects": false` cautiously for tree-shaking.
- Lazy import heavy chart modules; create a vendor chunk for React/D3.
- Add budgets: keep initial JS < 200 KB gz when possible.

---

## 6) Handling Big Data Visualization (Deep Dive)

This section expands practical techniques to keep dashboards fast and readable.

### 6.1 Pre-aggregate on backend

**Why**
- Raw telemetry can be huge (millions of rows/day).
- Plotting raw points causes overplotting, high memory/DOM cost, and slow first paint.

**Strategies**
- Time bucketing (second → minute → hour → day → month).
- Category rollups (SKU → family → region).
- Materialized views with scheduled refresh.
- Pre-computed tiles: `{time_bucket, dim, metric}` with partitioning.

**SQL Example (monthly buckets)**

```sql
SELECT
  date_trunc('month', ts AT TIME ZONE 'UTC') AS month_utc,
  region,
  SUM(amount) AS sales
FROM fact_sales
WHERE ts >= date_trunc('month', NOW() - INTERVAL '6 months')
GROUP BY 1, 2
ORDER BY 1, 2;
```

**API shape (BFF → FE)**

```ts
// GET /api/series/sales?window=6m&groupBy=month&region=NA
type SeriesPoint = { t: string /* ISO month */, value: number };
type SeriesPayload = { meta: { window: "6m", groupBy: "month", region?: string }, series: SeriesPoint[] };
```

**Pitfalls**
- Timezones; partial last month; ragged series. Document your window-closure policy.

---

### 6.2 Downsample with LTTB; LOD via brush

**Goal**: Preserve shape with far fewer points for long lines (e.g., 500k samples).

**LTTB (Largest-Triangle-Three-Buckets) — TypeScript**

```ts
type Pt = { x: number; y: number };

export function lttb(data: Pt[], threshold: number): Pt[] {
  if (threshold >= data.length || threshold <= 0) return data.slice();
  const sampled: Pt[] = [];
  const bucketSize = (data.length - 2) / (threshold - 2);

  let a = 0;
  sampled.push(data[a]);

  for (let i = 0; i < threshold - 2; i++) {
    const rangeStart = Math.floor((i + 1) * bucketSize) + 1;
    const rangeEnd = Math.floor((i + 2) * bucketSize) + 1;
    const range = data.slice(rangeStart, Math.min(rangeEnd, data.length));

    const nextStart = Math.floor((i + 2) * bucketSize) + 1;
    const nextEnd = Math.floor((i + 3) * bucketSize) + 1;
    const nextRange = data.slice(nextStart, Math.min(nextEnd, data.length));
    const avgX = nextRange.reduce((s, p) => s + p.x, 0) / (nextRange.length || 1);
    const avgY = nextRange.reduce((s, p) => s + p.y, 0) / (nextRange.length || 1);

    let maxArea = -1;
    let maxIdx = 0;
    for (let j = 0; j < range.length; j++) {
      const p = range[j];
      const area = Math.abs((data[a].x - avgX) * (p.y - data[a].y) - (data[a].x - p.x) * (avgY - data[a].y));
      if (area > maxArea) { maxArea = area; maxIdx = j; }
    }
    const chosen = range[maxIdx];
    sampled.push(chosen);
    a = data.indexOf(chosen);
  }
  sampled.push(data[data.length - 1]);
  return sampled;
}
```

**LOD (overview + detail) via brush**
- Render an overview chart with downsampled data.
- A brush interaction picks a time window.
- On brush change, filter locally or refetch server-side aggregates to render the detailed chart.

---

### 6.3 Density plots for >100k points (hexbin / KDE)

**Hexbin (fast & simple)**
```tsx
import * as d3 from "d3";
import { hexbin as d3hexbin } from "d3-hexbin";

type P = { x: number; y: number };

function HexLayer(svg: SVGSVGElement, points: P[], width: number, height: number) {
  const g = d3.select(svg).append("g");
  const x = d3.scaleLinear().domain(d3.extent(points, d => d.x) as [number, number]).range([0, width]);
  const y = d3.scaleLinear().domain(d3.extent(points, d => d.y) as [number, number]).range([height, 0]);

  const hex = d3hexbin<P>().x(d => x(d.x)).y(d => y(d.y)).radius(10).extent([[0,0],[width,height]]);
  const bins = hex(points);
  const color = d3.scaleSequential(d3.interpolateBlues).domain([0, d3.max(bins, b => b.length) || 1]);

  g.selectAll("path").data(bins).join("path")
    .attr("d", hex.hexagon())
    .attr("transform", d => `translate(${d.x},${d.y})`)
    .attr("fill", d => color(d.length) as string);
}
```

**KDE**: Compute on a worker/server grid, render as a Canvas heatmap for continuous density.

---

### 6.4 Canvas/WebGL fallback

**When**: SVG slows down at ~10–20k DOM nodes; for 100k+ points switch to Canvas or WebGL.

**Canvas scatter (minimal)**

```tsx
import React, { useRef, useEffect } from "react";
type P = { x: number; y: number };

export function CanvasScatter({ points, width=800, height=400 }: { points: P[]; width?: number; height?: number }) {
  const ref = useRef<HTMLCanvasElement | null>(null);

  useEffect(() => {
    const ctx = ref.current?.getContext("2d");
    if (!ctx) return;
    ctx.clearRect(0,0,width,height);

    const xs = points.map(p => p.x), ys = points.map(p => p.y);
    const xmin = Math.min(...xs), xmax = Math.max(...xs);
    const ymin = Math.min(...ys), ymax = Math.max(...ys);
    const sx = (v: number) => ((v - xmin) / (xmax - xmin || 1)) * (width - 40) + 20;
    const sy = (v: number) => height - (((v - ymin) / (ymax - ymin || 1)) * (height - 40) + 20);

    ctx.globalAlpha = 0.8;
    for (const p of points) { ctx.beginPath(); ctx.arc(sx(p.x), sy(p.y), 1.5, 0, Math.PI*2); ctx.fill(); }
  }, [points, width, height]);

  return <canvas ref={ref} width={width} height={height} />;
}
```

**WebGL**: use `regl` / `deck.gl` for millions of marks, shader-based heatmaps/contours.

---

### 6.5 Web Workers (off-thread transforms)

**Why**: Heavy transforms (LTTB, aggregation, CSV parsing) block the main thread and cause jank.

**Bootstrapping (Webpack 5)**

```ts
// main thread
const worker = new Worker(new URL("./lttb.worker.ts", import.meta.url));
worker.postMessage({ type: "LTTB", data, threshold: 2000 });
worker.onmessage = (ev) => setDownsampled(ev.data.result);
```

```ts
// lttb.worker.ts
import { lttb } from "./lttb";
self.onmessage = (ev: MessageEvent) => {
  const { data, threshold } = ev.data;
  const result = lttb(data, threshold);
  (self as any).postMessage({ result });
};
```

**Tips**
- Prefer `Float32Array`/TypedArrays; pass as Transferables to avoid copies.
- Keep workers stateless; terminate when idle.
- Consider `Comlink` to simplify RPC.

---

### 6.6 Virtualize long lists/tables (drilldowns)

**react-window (simple & fast)**

```tsx
import { FixedSizeList as List } from "react-window";

function Row({ index, style, data }: any) {
  const row = data.rows[index];
  return (
    <div style={style} className="row">
      <span className="cell">{row.time}</span>
      <span className="cell">{row.region}</span>
      <span className="cell">{row.value.toLocaleString()}</span>
    </div>
  );
}

export function BigTable({ rows }: { rows: any[] }) {
  return (
    <List height={480} itemCount={rows.length} itemSize={36} width={960} itemData={{ rows }}>
      {Row}
    </List>
  );
}
```

**Best practices**
- Keep a sticky header outside the virtualized list.
- Fixed-height rows are fastest; avoid measuring.
- For sort/filter, offload to server or worker.
- Add ARIA roles and keyboard focus management.

---

### 6.7 Decision Guide

| Situation | Recommended technique |
|---|---|
| Line series with >50k points | LTTB downsample to 1–3k + brush LOD |
| Scatter >100k | Hexbin density or Canvas/WebGL scatter |
| Continuous density desired | KDE (worker/server) → Canvas heatmap |
| Over‑time aggregates | Pre‑aggregate on backend; materialized views |
| Long tables (>5k rows) | Virtualize (react-window/virtual) |
| Heavy math (binning, parsing) | Web Worker; TypedArrays & transfer |

**Heuristics**
- Keep **SVG nodes < 10–20k**.
- Use **Canvas/WebGL** for massive point layers.
- Always **pre‑aggregate** when fine granularity isn’t needed.
- Provide an escape hatch to fetch raw slices on demand.

---

## 7) Interview Question Bank (Highlights)

**React Core**
- Explain reconciliation and the virtual DOM.
- What’s the difference between `useMemo` and `useCallback`? When not to use them?
- When does a child component re-render even if props seem unchanged?

**Performance**
- How do you optimize React rendering for large tables?
- Why prefer `memo` + `useCallback` over manual `shouldComponentUpdate`?

**D3 / Visualization**
- Difference between `scaleLinear` and `scaleTime`? When to use Canvas over SVG?
- How would you handle 1,000,000 points?

**Build Systems**
- What does tree-shaking mean in Webpack 5?
- How does Module Federation help in micro-frontends?

**System Design**
- Architect a data visualization dashboard with real-time updates.
- Caching strategies between BFF and frontend (keys, TTLs, busting).
- Handling latency and rendering consistency with streaming APIs.

---

## 8) Recommended Study Roadmap

| Topic | Resources |
|-------|-----------|
| React Performance | React Docs (Concurrent Rendering), EpicReact.dev |
| D3 Advanced | ObservableHQ, “Interactive Data Visualization for the Web” |
| Webpack 5 | Official Webpack Docs, SurviveJS |
| System Design for FE | FrontendMasters “Scaling React Apps” |
| Visualization Patterns | d3js.org, Mike Bostock’s blocks |

---

*Last updated: 2025-10-05*

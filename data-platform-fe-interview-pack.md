---
title: "Data Platform Front-End Interview Pack (Senior/Staff)"
description: "System design, D3-first visualization architecture, and end-to-end code (React + D3 + Node/Express + AI integration). Includes interview questions across React, Web Perf, APIs, D3, and DevOps."
date: 2025-10-05
---

# Data Platform Front-End Interview Pack (Senior/Staff)

This single file contains: **system design (AI-driven analytics flow)**, **D3-centric implementation details**, **ready-to-run code samples**, and a **question bank**.  
All content is **vendor-neutral** (no company-specific names).

> **Note on diagrams**: GitHub supports Mermaid. If you see “Unable to render rich display”, ensure the code fence starts with ```mermaid and that each node/edge is on its own line (we follow that here).

---

## Table of Contents

1. [Scenario & Requirements](#scenario--requirements)  
2. [System Design (End-to-End)](#system-design-end-to-end)  
   - [Architecture Diagram (Mermaid)](#architecture-diagram-mermaid)  
   - [Key Design Decisions](#key-design-decisions)  
   - [Data Contracts & Organization](#data-contracts--organization)  
   - [Caching, Security, and Observability](#caching-security-and-observability)  
3. [D3-First Frontend Architecture](#d3-first-frontend-architecture)  
   - [State & Data Flow](#state--data-flow)  
   - [Tidy/Columnar Data and Shaping](#tidycolumnar-data-and-shaping)  
   - [Handling Huge Datasets (Strategies)](#handling-huge-datasets-strategies)  
4. [D3 Examples (React + TypeScript)](#d3-examples-react--typescript)  
   - [Combo Chart: Monthly Bars + 3-Month MA Line](#combo-chart-monthly-bars--3-month-ma-line)  
   - [Zoom/Brush Area Chart (Range Selection)](#zoombrush-area-chart-range-selection)  
   - [Scatter + Density (LOD / Hexbin)](#scatter--density-lod--hexbin)  
   - [Sankey Diagram for AI Data Flow](#sankey-diagram-for-ai-data-flow)  
   - [Export (CSV & PNG)](#export-csv--png)  
5. [Backend (Node.js + Express BFF)](#backend-nodejs--express-bff)  
   - [REST Endpoints](#rest-endpoints)  
   - [AI Adapter (Mock) and Data Service](#ai-adapter-mock-and-data-service)  
   - [Validation & Error Handling](#validation--error-handling)  
6. [Testing Strategy](#testing-strategy)  
7. [Build & DevOps](#build--devops)  
8. [Interview Question Bank](#interview-question-bank)

---

## Scenario & Requirements

**User story:** “Show me the **last 6 months of sales**.”  
**Flow:** User selects parameters → UI triggers **AI intent parsing** (optional) → BFF orchestrates **AI → query generation → data warehouse** → returns **chart-ready series** for **D3**.

**Functional**: presets (last 6m, YTD), custom ranges, filters (product/region), multiple chart types, export CSV/PNG.  
**Non-functional**: P95 TTFB ≤ 300ms (cache hit), A11y (WCAG 2.1 AA), idempotent APIs, RBAC, audit logs.

---

## System Design (End-to-End)

### Architecture Diagram (Mermaid)

```mermaid
flowchart LR
  U[User (Web UI)]
  FE[React App]
  BFF[Node/Express BFF]
  AI[AI Intent Service]
  DW[(Data Warehouse)]
  D3[D3 Layer]
  OBS[(Observability)]
  CACHE[(Cache)]

  U -->|Query params| FE
  FE -->|/api/charts?sales| BFF
  BFF -->|NL prompt| AI
  AI -->|SQL/DSL| BFF
  BFF -->|Parameterized SQL| DW
  DW -->|Aggregates| BFF
  BFF -->|Shape series + cache| FE
  FE -->|Render SVG| D3
  BFF -->|Traces/Logs| OBS
  BFF <-->|redis| CACHE
```

> If GitHub still fails to render Mermaid, switch to **markdown preview in your IDE** or paste this block into https://mermaid.live to validate.

### Key Design Decisions

- **BFF** shapes UI-specific payloads, normalizes params, calls AI, guards auth, and handles caching.  
- **AI intent parsing** converts natural language → **domain DSL/SQL** with allowlists; always parameterize.  
- **Data shape** returned is **chart-ready**: `[{{"date":"YYYY-MM-DD","value":n}}]` + meta + stats.  
- **Security**: JWT between tiers, least-privileged DW access, row-level policies for multi-tenant data.  
- **Observability**: OpenTelemetry traces: FE → BFF → AI → DW; propagate correlation id.

### Data Contracts & Organization

- Prefer **tidy data**: each row is an observation; columns are variables; one table per data type.
- For time series:
  ```json
  [
    {{ "date": "2025-04-01", "metric": "sales", "value": 102340000, "region": "NA", "product": "Phone" }}
  ]
  ```
- **Client shape** (minimized):
  ```json
  {{
    "meta": {{ "range": "2025-04..2025-09", "groupBy": "month", "unit": "USD" }},
    "series": [
      {{ "date": "2025-04-01", "sales": 102340000 }},
      {{ "date": "2025-05-01", "sales": 99870000 }}
    ]
  }}
  ```

### Caching, Security, and Observability

- **Cache key:** `tenant:{{tid}}:uid:{{uid}}:v1:hash(sorted(params))`  
- **Cache layers:** Redis (server); optional CDN with short TTL.  
- **Rate limits:** token bucket per user/tenant; `429` with `Retry-After`.  
- **Traces:** attributes: tenant, query hash, rows, latency buckets.

---

## D3-First Frontend Architecture

### State & Data Flow

- **Global async state**: React Query or SWR for caching, retries, and de-dupe.
- **Local viz state**: zoom/brush window, hover index, selected series, tooltip model.
- **Pure render**: compute scales with `useMemo`; mutate only within a ref’d `<svg>` via D3.

### Tidy/Columnar Data and Shaping

- Convert incoming series to **Date objects** and numeric types once.  
- Derive **moving averages**, **bins**, **quantiles** via d3-array (memoized).  
- Separate **raw data** from **derived data**; snapshot derived state for tooltips/export.

### Handling Huge Datasets (Strategies)

- **Pre-aggregate** in DW (groupBy month/week).  
- **Downsample** (largest-triangle-three-buckets) or **LOD** selection via brush.  
- **Binning** (hexbin/density) for scatter with millions of points.  
- **Worker** offload for compute-heavy transforms.  
- **Canvas/WebGL** fallbacks when SVG node counts exceed ~10–20k.  
- **Virtualization** for tabular drill-down.

---

## D3 Examples (React + TypeScript)

> All examples are self-contained components; swap data props to use with real APIs.

### Combo Chart: Monthly Bars + 3-Month MA Line

```tsx
// components/ComboChart.tsx
import React, { useEffect, useMemo, useRef } from "react";
import * as d3 from "d3";

type Row = {{ date: string; sales: number }};
export function ComboChart({ data }: {{ data: Row[] }}) {{
  const ref = useRef<SVGSVGElement | null>(null);

  const series = useMemo(() => {{
    const parse = d3.timeParse("%Y-%m-%d");
    const pts = data.map(d => ({{ date: parse(d.date)!, sales: +d.sales }}));
    const arr = pts.map(d => d.sales);
    const ma = arr.map((_, i) => (i >= 2 ? d3.mean(arr.slice(i - 2, i + 1))! : null));
    return pts.map((d, i) => ({{ ...d, ma: ma[i] as number | null }}));
  }}, [data]);

  useEffect(() => {{
    if (!ref.current || series.length === 0) return;
    const svg = d3.select(ref.current);
    const W = 820, H = 360, M = {{ t: 20, r: 24, b: 32, l: 56 }};
    svg.attr("viewBox", `0 0 ${{W}} ${{H}}`).attr("role", "img");

    svg.selectAll("*").remove();
    const g = svg.append("g").attr("transform", `translate(${{M.l}},${{M.t}})`);
    const w = W - M.l - M.r, h = H - M.t - M.b;

    const x = d3.scaleUtc()
      .domain(d3.extent(series, d => d.date) as [Date, Date])
      .range([0, w]);

    const y = d3.scaleLinear()
      .domain([0, d3.max(series, d => Math.max(d.sales, d.ma ?? 0))!]).nice()
      .range([h, 0]);

    const bw = Math.max(8, (w / series.length) * 0.6);
    g.append("g").selectAll("rect").data(series).join("rect")
      .attr("x", d => x(d.date) - bw / 2)
      .attr("y", d => y(d.sales))
      .attr("width", bw)
      .attr("height", d => h - y(d.sales))
      .attr("fill", "#6ea8fe")
      .append("title").text(d => `${{d3.timeFormat("%b %Y")(d.date)}}: ${{d3.format(",")(d.sales)}}`);

    const line = d3.line<any>().defined(d => d.ma != null).x(d => x(d.date)).y(d => y(d.ma));
    g.append("path").datum(series).attr("fill","none").attr("stroke","#0d6efd").attr("stroke-width",2).attr("d", line as any);

    g.append("g").attr("transform", `translate(0,${{h}})`).call(d3.axisBottom(x).ticks(series.length).tickFormat(d3.timeFormat("%b") as any));
    g.append("g").call(d3.axisLeft(y).ticks(5).tickFormat(d3.format("~s") as any));
  }}, [series]);

  return <svg ref={ref} className="w-100" />;
}}
```

### Zoom/Brush Area Chart (Range Selection)

```tsx
// components/BrushArea.tsx
import React, { useEffect, useMemo, useRef, useState } from "react";
import * as d3 from "d3";

type R = {{ date: string; value: number }};
export function BrushArea({ data, onRange }: {{ data: R[]; onRange?: (d0: Date, d1: Date)=>void }}) {{
  const ref = useRef<SVGSVGElement | null>(null);
  const [domain, setDomain] = useState<[Date, Date] | null>(null);

  const pts = useMemo(() => {{
    const parse = d3.timeParse("%Y-%m-%d");
    return data.map(d => ({{ date: parse(d.date)!, value: +d.value }}));
  }}, [data]);

  useEffect(() => {{
    if (!ref.current || pts.length===0) return;
    const svg = d3.select(ref.current);
    const W = 860, H = 420, M = {{ t:20, r:24, b:80, l:56 }};
    const h1 = 280, h2 = 80;
    svg.attr("viewBox", `0 0 ${{W}} ${{H}}`);
    svg.selectAll("*").remove();

    const x = d3.scaleUtc().domain(d3.extent(pts, d=>d.date) as [Date, Date]).range([M.l, W - M.r]);
    const y = d3.scaleLinear().domain([0, d3.max(pts, d=>d.value)!]).nice().range([M.t + h1, M.t]);

    const area = d3.area<any>().x(d=>x(d.date)).y0(y(0)).y1(d=>y(d.value));

    svg.append("path").datum(pts).attr("fill","#cfe2ff").attr("d", area as any);
    svg.append("g").attr("transform", `translate(0,${{M.t + h1}})`).call(d3.axisBottom(x));
    svg.append("g").attr("transform", `translate(${{M.l}},0)`).call(d3.axisLeft(y));

    const yc = d3.scaleLinear().domain(y.domain()).range([H - M.b, H - M.b - h2]);
    const areac = d3.area<any>().x(d=>x(d.date)).y0(yc(0)).y1(d=>yc(d.value));
    svg.append("path").datum(pts).attr("fill","#e7f1ff").attr("d", areac as any);

    const brush = d3.brushX().extent([[M.l, H - M.b - h2],[W - M.r, H - M.b]])
      .on("brush end", (ev:any) => {{
        if (!ev.selection) return;
        const [x0,x1] = ev.selection.map(x.invert);
        setDomain([x0,x1]);
        if (onRange) onRange(x0,x1);
      }});

    svg.append("g").attr("class","brush").call(brush).call(brush.move, [x.range()[0]+40, x.range()[1]-40]);
  }}, [pts, onRange]);

  return <svg ref={ref} className="w-100" />;
}}
```

### Scatter + Density (LOD / Hexbin)

```tsx
// components/HexScatter.tsx
import React, { useEffect, useMemo, useRef } from "react";
import * as d3 from "d3";
import {{ hexbin as d3hexbin }} from "d3-hexbin";

type P = {{ x: number; y: number }};
export function HexScatter({ points }: {{ points: P[] }}) {{
  const ref = useRef<SVGSVGElement | null>(null);
  const data = useMemo(() => points, [points]);

  useEffect(() => {{
    if (!ref.current || data.length===0) return;
    const W=820,H=360,M={{t:20,r:24,b:32,l:56}};
    const svg = d3.select(ref.current).attr("viewBox",`0 0 ${{W}} ${{H}}`);
    svg.selectAll("*").remove();
    const w=W-M.l-M.r,h=H-M.t-M.b;
    const g=svg.append("g").attr("transform",`translate(${{M.l}},${{M.t}})`);

    const x = d3.scaleLinear().domain(d3.extent(data,d=>d.x) as [number,number]).nice().range([0,w]);
    const y = d3.scaleLinear().domain(d3.extent(data,d=>d.y) as [number,number]).nice().range([h,0]);

    const hex = d3hexbin().x(d=>x((d as any).x)).y(d=>y((d as any).y)).radius(10).extent([[0,0],[w,h]]);
    const bins = hex(data as any);
    const c = d3.scaleSequential(d3.interpolateBlues).domain([0, d3.max(bins,b=>b.length) || 1]);

    g.append("g").selectAll("path").data(bins).join("path")
      .attr("d", hex.hexagon())
      .attr("transform", (d:any) => `translate(${{d.x}},${{d.y}})`)
      .attr("fill", (d:any) => c(d.length) as string);

    g.append("g").attr("transform",`translate(0,${{h}})`).call(d3.axisBottom(x));
    g.append("g").call(d3.axisLeft(y));
  }}, [data]);

  return <svg ref={ref} className="w-100" />;
}}
```

### Sankey Diagram for AI Data Flow

```tsx
// components/AISankey.tsx
import React, { useEffect, useRef } from "react";
import {{ sankey, sankeyLinkHorizontal, sankeyLeft }} from "d3-sankey";
import * as d3 from "d3";

type SankeyNode = {{ name: string }};
type SankeyLink = {{ source: number; target: number; value: number }};
export function AISankey() {{
  const ref = useRef<SVGSVGElement | null>(null);

  useEffect(() => {{
    if (!ref.current) return;
    const nodes: SankeyNode[] = [
      {{ name: "User Query" }},
      {{ name: "AI Intent" }},
      {{ name: "Query Builder" }},
      {{ name: "Warehouse" }},
      {{ name: "Shape/Cache" }},
      {{ name: "D3 Chart" }}
    ];
    const links: SankeyLink[] = [
      {{ source: 0, target: 1, value: 1 }},
      {{ source: 1, target: 2, value: 1 }},
      {{ source: 2, target: 3, value: 1 }},
      {{ source: 3, target: 4, value: 1 }},
      {{ source: 4, target: 5, value: 1 }}
    ];

    const W=860,H=300,M={{t:20,r:20,b:20,l:20}};
    const svg = d3.select(ref.current).attr("viewBox",`0 0 ${{W}} ${{H}}`);
    svg.selectAll("*").remove();

    const {{ nodes: n, links: l }} = sankey<SankeyNode, SankeyLink>()
      .nodeAlign(sankeyLeft)
      .nodeWidth(14).nodePadding(20)
      .extent([[M.l,M.t],[W-M.r,H-M.b]])({{
        nodes: nodes.map(d=>Object.assign({{}},d)),
        links: links.map(d=>Object.assign({{}},d))
      }});

    const color = d3.scaleOrdinal(d3.schemeSet2);

    svg.append("g").selectAll("rect").data(n).join("rect")
      .attr("x", d => (d as any).x0).attr("y", d => (d as any).y0)
      .attr("height", d => (d as any).y1 - (d as any).y0)
      .attr("width", d => (d as any).x1 - (d as any).x0)
      .attr("fill", (_,i)=> color(String(i)) as string);

    svg.append("g").selectAll("path").data(l).join("path")
      .attr("d", sankeyLinkHorizontal() as any)
      .attr("fill","none")
      .attr("stroke","#888")
      .attr("stroke-width", d => Math.max(1,(d as any).width))
      .attr("opacity",0.6);

    svg.append("g").selectAll("text").data(n).join("text")
      .attr("x", d => (d as any).x0 - 6).attr("y", d => ((d as any).y0 + (d as any).y1)/2)
      .attr("dy","0.35em").attr("text-anchor","end")
      .text(d => (d as any).name);
  }}, []);

  return <svg ref={ref} className="w-100" />;
}}
```

### Export (CSV & PNG)

```ts
// utils/exports.ts
export function exportCSV(series: {{date:string; value:number}}[], file="data.csv") {{
  const header = "date,value\n";
  const rows = series.map(s => `${{s.date}},${{s.value}}`).join("\n");
  const blob = new Blob([header + rows], {{ type: "text/csv;charset=utf-8" }});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = file; a.click();
}
```

---

## Backend (Node.js + Express BFF)

> The BFF authenticates, calls **AI adapter** (optional) to convert NL → SQL/DSL, queries the warehouse, shapes the response, and caches results.

### REST Endpoints

```ts
// server/index.ts
import express from "express";
import cors from "cors";
import * as crypto from "crypto";
import {{ aiToSql }} from "./services/ai";
import {{ queryDw }} from "./services/dw";
import {{ z }} from "zod";
import Redis from "ioredis";

const app = express();
app.use(cors({{ origin: true, credentials: true }}));
app.use(express.json());

const redis = new Redis(process.env.REDIS_URL ?? "");

const Params = z.object({{
  range: z.string().default("last_6m"),
  groupBy: z.enum(["day","week","month"]).default("month"),
  region: z.string().optional(),
  product: z.string().optional()
}});

app.get("/api/charts/sales", async (req, res) => {{
  try {{
    const user = {{ id: req.header("x-user-id") ?? "demo", scopes: ["read:sales"] }};
    const params = Params.parse(req.query);
    const key = `sales:${{user.id}}:${{crypto.createHash("sha1").update(JSON.stringify(params)).digest("hex")}}`;

    const cached = await redis.get(key);
    if (cached) return res.set("x-cache","HIT").json(JSON.parse(cached));

    const prompt = `Give me ${{params.range}} sales grouped by ${{params.groupBy}}` +
      (params.region ? ` in region ${{params.region}}` : "") +
      (params.product ? ` for product ${{params.product}}` : "");
    const sql = await aiToSql(prompt);

    const rows = await queryDw(sql);
    const resp = {{ meta: {{ ...params }}, series: rows }};

    await redis.setex(key, 60, JSON.stringify(resp));
    res.set("x-cache","MISS").json(resp);
  }} catch (e:any) {{
    res.status(400).json({{ error: e.message ?? "Bad Request" }});
  }}
}});

app.listen(process.env.PORT ?? 3000, () => console.log("API up"));
```

### AI Adapter (Mock) and Data Service

```ts
// server/services/ai.ts
export default async function aiToSql(nl: string): Promise<string> {{
  return `SELECT DATE_TRUNC('month', date) AS date, SUM(sales) AS sales
          FROM fact_sales
          WHERE date >= CURRENT_DATE - INTERVAL '6 months'
          GROUP BY 1 ORDER BY 1`;
}}
```

```ts
// server/services/dw.ts
type Row = {{ date: string; sales: number }};
export async function queryDw(sql: string): Promise<Row[]> {{
  return [
    {{ date: "2025-04-01", sales: 102340000 }},
    {{ date: "2025-05-01", sales: 99870000 }},
    {{ date: "2025-06-01", sales: 120120000 }},
    {{ date: "2025-07-01", sales: 118770000 }},
    {{ date: "2025-08-01", sales: 130560000 }},
    {{ date: "2025-09-01", sales: 129990000 }}
  ];
}}
```

### Validation & Error Handling

- `zod` for input validation; never trust client-provided fields.  
- Central error response: `{{ "error": "message", "code": "E_VALIDATION" }}`.  
- Structured logs with correlation id; no sensitive data in logs.

---

## Testing Strategy

- **Unit**: helpers (moving averages, binning), AI adapter contract.  
- **Component**: D3 components render paths/rects correctly (test SVG DOM).  
- **E2E**: dashboard loads, brush selection narrows range, export triggers download.  
- **Contract tests** for BFF ↔ DW shape; snapshot of typical payloads.

---

## Build & DevOps

- Prefer **Vite**; enforce bundle budget (`< 200KB` gz for shell; charts lazy).  
- **Docker multi-stage**; read-only FS; non-root user.  
- **K8s**: readinessProbes, rolling updates, HPA; secrets via sealed-secrets.  
- **Observability**: OpenTelemetry auto-instrumentation; dashboards for latency, error rate, cache hit ratio.

---

## Interview Question Bank

- **React**: re-render vs re-mount; `useMemo`/`useCallback` trade-offs; Suspense boundaries.  
- **D3**: scale/axis layout; handling 1M+ points (binning/LOD/canvas/WebGL); accessibility patterns.  
- **APIs**: idempotency, pagination (cursor), versioning, auth propagation; cache invalidation.  
- **Perf**: LCP/INP/TTFB; code-splitting; streaming SSR; worker offload; memoization and referential stability.  
- **Testing**: MSW vs jest-fetch-mock; Playwright fixtures; visual regression strategy.  
- **DevOps**: zero-downtime deploy; config management; secret rotation; SLOs and alerting.

---

> **How to use**: Paste the React components into your project, wire the BFF route, and swap the mock AI/DW services for your real endpoints. D3 examples are minimal yet production-leaning (scales memoized, DOM mutated in effects, accessibility considered).

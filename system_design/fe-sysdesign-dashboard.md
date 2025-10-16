# Front-End System Design Deep Dive — Dashboard / Analytics Platform

> Design an **interactive analytics dashboard** displaying metrics, charts, filters, and live updates using **React + TypeScript**. Explore state management, data flow, performance, and visualization.

---

## 1. Requirements
**Functional**
- Multiple charts (line, bar, pie, KPI cards).
- Filtering, time range selector, drill-down.
- Real-time updates (WebSocket / polling).
- Responsive layout, dark mode.
- Export CSV/PDF.

**Non-Functional**
- Fast initial render (TTI < 3 s).
- Smooth chart interactions.
- Handle large datasets efficiently.
- Reliable updates and data consistency.

---

## 2. Architecture Overview
```
DashboardApp
 ├─ Header / Filters / DatePicker
 ├─ WidgetGrid
 │   ├─ ChartWidget (Line, Bar, Pie)
 │   ├─ KPIWidget
 │   └─ TableWidget
 ├─ useDashboardData() → cache, WS subscription, refetch
 └─ Recharts / Chart.js / D3 rendering layer
```

---

## 3. Tech Stack & Rationale

| Layer | Technology | Why | Alternatives |
|-------|-------------|-----|---------------|
| **UI Library** | React + TypeScript | Component reusability, type safety | Vue, Svelte |
| **State Mgmt** | React Query | Auto cache, background refetch, WS patching | Redux Toolkit, SWR |
| **Charts** | Recharts / Chart.js | Declarative, responsive, composable | D3 (for full control) |
| **Transport** | REST + WebSocket | Simpler than gRPC; WS for incremental updates | SSE (one‑way), Polling |
| **Layout** | CSS Grid + Responsive design | Auto‑flow for widget cards | flexbox, virtual grid |
| **Perf** | React.memo, virtualization | Prevent re‑renders | signals, selectors |

---

## 4. Implementation (Simplified)

```tsx
// useDashboardData.ts
import { useQuery } from "@tanstack/react-query";

export function useDashboardData(range: string) {
  return useQuery({
    queryKey: ["dashboard", range],
    queryFn: () => fetch(`/api/dashboard?range=${range}`).then(r => r.json()),
    staleTime: 60_000,
  });
}

// ChartWidget.tsx
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

export function ChartWidget({ data }: { data: any[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <XAxis dataKey="time" />
        <YAxis />
        <Tooltip />
        <Line type="monotone" dataKey="value" stroke="#2196f3" />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

---

## 5. Performance Optimizations
- Use **windowing / virtualization** for tables.
- Memoize heavy chart renders (`React.memo`, `useMemo`).
- Batch WS updates; debounce filter changes.
- Avoid unnecessary reflows by using `content-visibility: auto`.

---

## 6. Testing
- Unit: data transformation functions.
- Integration: filter interactions update charts.
- E2E: verify live updates via WS mock.

---

## 7. Interview Soundbite
> “I built dashboards with cached queries and batched WebSocket updates, using Recharts for declarative charts and React Query for smart caching.”

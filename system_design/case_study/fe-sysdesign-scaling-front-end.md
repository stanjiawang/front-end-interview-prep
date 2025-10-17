# Front-End System Design Deep Dive — Scaling the Front End (Performance & Ops)

> Practical strategies to scale **performance, reliability, and maintainability** of large React apps.

---

## 1. Performance Budgets & Goals
- LCP < 2.5s (mid-tier device, 4G), CLS < 0.1, TTI < 3.5s.
- JS ≤ ~200KB gz on initial route; images optimized (AVIF/WebP).

---

## 2. Asset Strategy
- **Code splitting** by route and heavy widgets; `dynamic import()` for editors/charts.
- **Tree-shaking**, remove dead deps; analyze bundles (Source Map Explorer).
- **HTTP/2/3** with compression (Brotli), long-lived cache headers.

---

## 3. Runtime Performance
- Virtualize large lists; memoize pure components; stabilize props.
- Avoid cascading re-renders via context; prefer selectors or signals.
- Use `content-visibility: auto` & skeletons to avoid CLS.

---

## 4. Images & Media
- Responsive `srcset` + `sizes`; lazy loading; preconnect to CDN.
- Encode to AVIF/WebP; strip metadata; serve width-appropriate variants.

---

## 5. Data & Network
- React Query/RTK Query caching; prefetch likely routes; backoff & retries.
- Service Worker: offline shell + stale-while-revalidate; Background Sync for outbox.
- Push updates via WS/SSE where appropriate to avoid polling.

---

## 6. Observability & Quality Gates
- Web Vitals RUM ingestion (LCP, INP, CLS). Budget regressions in CI via Lighthouse.
- Error tracking with source maps; categorize by device/network.
- Feature flags + gradual rollout; kill switches.

---

## 7. Security & Privacy
- Strict CSP; sanitize user content; avoid `dangerouslySetInnerHTML`.
- Token storage best practices (in-memory or httpOnly cookies).
- PII minimization in logs; redaction.

---

## 8. Testing Pyramid
- Unit: utils, reducers, hooks.
- Component: RTL tests for visual states, accessibility.
- E2E: Playwright with trace for perf; network flake simulation.

---

## 9. Team & Repo Hygiene
- Monorepo with build caching (Turborepo/Nx); lint + type checks.
- Consistent module boundaries; ADRs for architectural decisions.
- Automated dependency updates; bundle CI checks.

---

## 10. Interview Soundbite
> “Scaling is about budgets, split-by-default, strong caching, and continuous measurement. I enforce budgets in CI and design for resilience (offline, retries, fallbacks).”

# Front-End System Design — Part 1: Interview Mindset & Answer Structure (Ultra-Detail Guide)
---

## Table of Contents

1. What Front‑End **System Design** Really Evaluates
2. Core Vocabulary (Precise Definitions You Can Say Out Loud)
3. The Five‑Stage Answer Framework (**C‑D‑D‑O‑I**)
4. Requirement Discovery: Functional vs **NFRs** (Non‑Functional Requirements)
5. Architectural Decomposition for the Front End (U‑P‑D‑S‑O Model)
6. Decision‑Making & Trade‑Off Patterns (How to Compare Options)
7. Communicating Under Time Pressure (Whiteboard/Pad Strategy)
8. Risk, Reliability & **Resilience**: Failure Modes & Fallbacks
9. Perf Mindset from Day 0: Budgets, **Core Web Vitals**, and Cost of JS
10. State & Data Thinking: Local/UI vs Server State; Caching Hierarchy
11. Security & Privacy Prompts (CSP, XSS, CSRF, Cookies, Tokens)
12. Accessibility & Internationalization as First‑Class NFRs
13. Observability & Operability (RUM, logs, traces, alerting)
14. Testing Strategy to Support the Design (Unit→E2E→Perf→Contract)
15. Rollout, Experimentation & Change Management (Feature Flags, A/B)
16. Stakeholder Questions to Ask the Interviewer (To Uncover Constraints)
17. Anti‑Patterns & Red Flags (What Senior Candidates Must Avoid)
18. Reusable Answer Templates (Speak‑Aloud Scripts + Tables)
19. Practice Drills (Timed) + Self‑Assessment Rubric
20. Appendix: Glossary, Checklists, and One‑Page Answer Outline

---

## 1) What Front‑End **System Design** Really Evaluates

**Goal:** Show that you can design, deliver, and scale front‑end systems in ambiguous environments.

**Interview signals:**  
- **Architecture thinking**: Can you decompose the FE into presentation, data, state, networking, caching, and ops?  
- **Trade‑off judgment**: Can you compare SSR vs CSR vs streaming, REST vs GraphQL, global vs local state, etc., with concrete impacts?  
- **Non‑functional literacy**: Performance, accessibility, security, and reliability discussed proactively.  
- **Operational awareness**: Metrics, logging, testing, rollout, incident response.  
- **Communication**: Clear phases, diagrams, prioritization, and crisp decisions.

**Reality check:** In senior rounds, the “UI” is **only one layer**. You must reason about the *system* that the UI lives within: rendering/transport/state/caching/observability.

---

## 2) Core Vocabulary (Precise Definitions You Can Say Out Loud)

- **System Design (Front‑End)**: A process to model requirements, decompose architecture, define interfaces and constraints, and select trade‑offs to meet functional and non‑functional goals.
- **Functional Requirements (FRs)**: What the product must do (e.g., render a feed, filter products, chat in real time).
- **Non‑Functional Requirements (NFRs)**: Qualities of the system — performance (LCP), scalability, reliability (error budgets), security (CSP), accessibility (WCAG), privacy (GDPR/CCPA), observability, maintainability.
- **Critical Rendering Path (CRP)**: Browser flow: parse HTML → DOM; parse CSS → CSSOM; build render tree → layout → paint → composite.
- **Client/Server Rendering**: CSR/SSR/SSG/ISR/Streaming; **hydration** to attach event handlers to server‑rendered HTML.
- **State Types**: UI/local state (component), global app state (cross‑cutting concerns), **server state** (remote, cached, synchronized).
- **Caching Hierarchy**: Memory (in‑app) → Service Worker (Cache Storage/IndexedDB) → HTTP cache (ETag/Cache‑Control) → CDN → Origin.
- **Core Web Vitals (CWV)**: FCP, **LCP**, **CLS**, **TBT**, **TTI**. Know the target thresholds.
- **Resilience**: System continues to provide acceptable service despite failures (retries, backoff, timeouts, fallbacks, circuit breakers).
- **Observability**: Ability to understand internal state from external outputs (metrics/logs/traces/RUM).

> **Speak‑aloud tip:** Define terms once, briefly, then use them fluently — this projects seniority.

---

## 3) The Five‑Stage Answer Framework (**C‑D‑D‑O‑I**)

1) **Clarify** – Users, scale, constraints, SLAs/SLOs, success metrics.  
2) **Decompose** – Presentation, data, state, caching, navigation, error/empty/loading, security, a11y, i18n.  
3) **Design** – Propose architecture options; compare; choose and **justify**.  
4) **Operate** – Telemetry, testing, release strategy, feature flags, incident playbook.  
5) **Iterate** – MVP → Scale → Hardening; what you’d do next.

**Time boxing (45–60 min):**  
- 5–7 min clarification; 15–20 min decomposition & decisions; 10 min ops & risks; 5 min iteration & recap; leave time for questions.

**ASCII outline to draw quickly**

```
Users/Use Cases ──► Constraints/Goals
        │
        ▼
Decompose FE: UI ▸ Data ▸ State ▸ Cache ▸ Nav ▸ Errors ▸ Security ▸ a11y
        │
        ▼
Design Options & Trade-offs (tables)
        │
        ▼
Operate: Metrics/Logs/Traces ▸ Tests ▸ Flags ▸ Rollout ▸ On-call
        │
        ▼
Iterate: MVP → Scale → Hardening (perf/security/observability)
```

---

## 4) Requirement Discovery: Functional vs **NFRs**

**Ask for FRs (what to build):** users, devices, interactions, SEO, real‑time needs, offline behavior.  
**Ask for NFRs (how it should behave):**

- **Performance:** p75 LCP target? JS budget (gzip)? Image policy (WebP/AVIF)?
- **Scalability:** DAU/MAU, QPS, pages/routes/components growth.
- **Availability/Latency:** SLOs (e.g., p95 < 200ms API latency).
- **Security/Privacy:** CSP level, auth (OAuth 2.0 + PKCE/OIDC), PII handling.
- **Accessibility:** WCAG 2.2 AA, keyboard flows, screen readers.
- **Internationalization:** locales, RTL, currency/timezone.
- **Compliance:** GDPR/CCPA data rights.
- **Operability:** error budget policy, alert routes, SLI/SLO reporting.

> **Template — NFR sentence:**  
> “We’ll target **LCP ≤ 2.5s p75**, **CLS ≤ 0.1**, **TBT ≤ 200ms**, ship **≤ 170 KB gzipped JS** on first load, meet **WCAG 2.2 AA**, enforce **strict CSP**, and log **RUM** for regression detection.”

---

## 5) Architectural Decomposition (U‑P‑D‑S‑O Model)

- **U — Users & Use Cases:** who/where/how many; SEO; device/Net conditions; growth assumptions.
- **P — Presentation Layer:** SPA/MPA; SSR/SSG/ISR/Streaming; routing; hydration strategy; skeletons; virtualization; media strategy.
- **D — Data Layer:** REST vs GraphQL; pagination (offset/cursor); realtime (WS/SSE/WebRTC); error contracts; versioning.
- **S — State & Storage:** UI/global/server state split; cache keys; invalidation; SW caching; IndexedDB; optimistic updates.
- **O — Observability & Ops:** CWV collection; logs; distributed tracing (**traceparent**); feature flags; A/B; canary; rollback.

**One‑glance canvas:**

```
U: users, scale, SEO, devices
P: SSR? CSR? Streaming? hydration? split points
D: REST/GraphQL, pagination, retries, backoff
S: server-state cache, UI state, SW/IDB
O: RUM, logs, traces, flags, A/B, SLOs
```

---

## 6) Decision‑Making & Trade‑Off Patterns

**Rendering Strategy** (simplified table):

| Option | Pros | Cons | Use When |
|---|---|---|---|
| **CSR (SPA)** | Rich transitions, simple hosting | Poor FCP/SEO without tweaks | Auth‑heavy app, dashboard |
| **SSR** | Fast first paint, SEO | Server cost, hydration tax | Content or SEO oriented |
| **SSG** | Ultra fast, cheap | Stale content risk | Mostly static content |
| **ISR** | Balance freshness & cost | Staleness window | Blogs, catalogs |
| **Streaming SSR** | Progressive rendering | Complexity, infra req | Data‑heavy above‑the‑fold |

**Data API**: REST (simplicity, CDN‑friendly) vs GraphQL (shape control, fewer round trips).  
**Pagination**: Cursor (stable under insertions) vs Offset (simple but drift).  
**State**: Prefer **server state caches** (React Query/RTK Query) and minimize global state.  
**Caching**: Identify *cache keys*, define *invalidation*, choose *layer* (memory vs SW vs HTTP vs CDN).

> **Phrase to say:** “I’ll choose SSR + selective hydration to reduce LCP and JS cost; pair with cursor pagination and a server‑state cache to avoid re‑fetch churn.”

---

## 7) Communicating Under Time Pressure

- **Start broad, then narrow**: draw the 5 boxes (U‑P‑D‑S‑O), fill bullets quickly.
- **Use trade‑off tables** for two or three high‑impact choices.
- **Name metrics** with thresholds (LCP, CLS, JS budget). Numbers show seniority.
- **Narrate risks + mitigations**: “If WS drops, we fallback to polling with jittered backoff.”
- **Close with a recap** and “what I’d do next”.

**Two‑minute opener (script):**  
“First, I’ll clarify users, scale, SEO and latency goals. Then I’ll decompose front‑end concerns: rendering, data fetching, state split, caching and errors. I’ll compare SSR vs CSR and choose based on LCP budget. I’ll cover observability and rollout, then propose milestones.”

---

## 8) Risk, Reliability & Resilience

**Common failures:** network timeouts, partial data, auth expiry, third‑party outages, slow main‑thread.  
**Patterns:** timeouts, retries with backoff + jitter, circuit breaker (fast‑fail), deadline propagation, idempotent actions, *dead‑letter UI* (explain to user and recover).

**Failure table (fill in during interview):**

| Failure | Detect | Mitigation | UX Fallback |
|---|---|---|---|
| API timeout | client timer | retry w/ backoff | skeleton + “Try again” |
| WS disconnect | ping/pong | reconnect + exp backoff | switch to SSE/poll |
| Image CDN slow | RUM | adaptive quality | LQIP/blur‑hash |
| Hydration thrash | RUM/TBT | split/hydrate islands | basic non‑interactive shell |

---

## 9) Performance Mindset & JS Budget

- **Budgets**: First load JS ≤ **170 KB gz** (tune per org), LCP ≤ **2.5s p75**, CLS ≤ **0.1**, TBT ≤ **200ms**.
- **Levers**: code splitting, lazy hydration, defer non‑critical JS, image next‑gen formats, critical CSS, preload hero, edge caching.
- **RUM loop**: collect field data → compare to budget → ship fixes → guard with CI/Lighthouse budgets.

> **Sentence:** “We’ll gate PRs with a size budget and track p75 LCP via RUM; regressions page an on‑call.”

---

## 10) State & Data Thinking (Essentials)

- **Server state** is remote, shareable, and stale; treat it as a **cache** with keys, TTL, and invalidation.  
- **UI state** is ephemeral (modals, selections).  
- **Global state** only for cross‑cutting concerns (auth, theme).  
- **Consistency**: optimistic updates + rollback; dedupe concurrent requests; cancel stale queries (AbortController).

**Cache key mantra:** “Key = resource + params + auth scope”.

---

## 11) Security & Privacy Prompts (Front‑End Angle)

- **XSS**: escape/sanitize, avoid `dangerouslySetInnerHTML`, enforce **CSP** with nonces.  
- **CSRF**: SameSite cookies, CSRF tokens on state‑changing requests.  
- **Clickjacking**: `frame-ancestors` / `X-Frame-Options`.  
- **Auth**: OAuth 2.0 Authorization Code with **PKCE**; **OIDC** for identity; short‑lived tokens; refresh flows.  
- **PII**: minimize in logs; honor GDPR/CCPA; data retention policies.

> **Add to design**: cite auth flow, token storage (memory vs httpOnly cookie) and CSP policy level.

---

## 12) Accessibility & Internationalization

- **A11y**: semantic HTML, ARIA sparingly, focus management, keyboard traps avoidance, contrast, visible focus rings, error messages with `aria-live`.
- **i18n**: ICU messages, pluralization, RTL layouts, locale numbers/dates, timezone awareness, dynamic content direction.

> **Commit:** “We’ll target WCAG 2.2 AA; run axe in CI and ship keyboard E2E tests for modals/menus.”

---

## 13) Observability & Operability

- **Metrics**: CWV via RUM, API latency, error rates, retries, WS reconnects.  
- **Logs**: structured with correlation IDs; redact PII.  
- **Traces**: propagate W3C `traceparent` to backend; stitch client→server spans.  
- **Dashboards/Alerts**: p75 LCP, 5xx spikes, JS error rate, WS drop rate.

**Speak‑aloud:** “I’ll instrument front end with RUM and trace context so we can follow a user click across services.”

---

## 14) Testing Strategy to Support the Design

- **Unit** (logic/hooks), **Integration** (components + mocks), **E2E** (critical flows), **Contract** (FE‑BE schema), **Visual** (UI diffs), **Perf** (Lighthouse, WebPageTest).  
- **Shift‑left** perf and a11y checks in CI.

---

## 15) Rollout, Experimentation & Change Management

- **Feature flags** (server‑evaluated when possible) + gradual rollouts.  
- **A/B testing** with exposure logging and guardrails.  
- **Canary** + **rollback** playbook; error budgets to throttle launches.

---

## 16) Questions to Ask the Interviewer (Constraint Mining)

- Users/devices/regions? SEO requirements? Real‑time vs eventual consistency? Offline expectations?  
- Perf targets (LCP/JS budget)? Security posture (CSP, token model)?  
- Release cadence, on‑call expectations, incident SLAs?  
- Data retention/privacy needs (GDPR/CCPA)?

> **Why:** Good questions = better design choices and show seniority.

---

## 17) Anti‑Patterns & Red Flags

- Jumping into components before clarifying NFRs.  
- Declaring “use Redux for everything” (over‑globalization).  
- Ignoring SSR/streaming when SEO/perf matters.  
- No plan for failures/timeouts/retries.  
- Shipping unlimited JS on first load; no budgets/metrics.  
- Hand‑waving on security/a11y/i18n.

---

## 18) Reusable Answer Templates

### 18.1 One‑Minute Elevator Summary
> “I’ll clarify users, scale, SEO and perf targets; decompose FE into rendering/data/state/cache/error paths; compare SSR/CSR and choose based on LCP & SEO; define API shape and pagination; separate server vs UI state with a cache strategy; cover security, a11y, and observability; then outline rollout and risks.”

### 18.2 Trade‑Off Table Snippet (fill live)
| Decision | Option A | Option B | Criteria | Choice |
|---|---|---|---|---|
| Render | CSR | SSR/Streaming | LCP/SEO/complexity | SSR + selective hydration |
| API | REST | GraphQL | CDN, shape control | REST + typed DTOs |

### 18.3 Non‑Functional Targets (fill live)
- LCP ≤ 2.5s p75, CLS ≤ 0.1, TBT ≤ 200ms, JS ≤ 170KB gz  
- WCAG 2.2 AA; strict CSP; RUM + tracing

### 18.4 Failure Table (fill live) — see §8

---

## 19) Practice Drills + Self‑Assessment

**Drill (10 min each):**  
- Draw U‑P‑D‑S‑O for: News Feed / PLP / Chat / Infinite Photo Grid.  
- For one decision (SSR vs CSR), write a 3‑row trade‑off table and finalize a choice.  
- Define 3 NFR targets and 3 metrics to monitor post‑launch.

**Self‑rubric (0–2 each, total /10):**  
1) Clarified FRs/NFRs with numbers?  
2) Decomposed FE layers coherently?  
3) Made explicit trade‑offs with rationale?  
4) Included reliability & security?  
5) Closed with rollout & iteration plan?

---

## 20) Appendix: Glossary, Checklists, One‑Page Outline

### Glossary (abbrev → full)
- **CRP**: Critical Rendering Path  
- **CWV**: Core Web Vitals (FCP, LCP, CLS, TBT, TTI)  
- **NFRs**: Non‑Functional Requirements  
- **CSR/SSR/SSG/ISR**: Client/Server/Static/Incremental Static Regeneration  
- **CSP**: Content Security Policy  
- **OIDC**: OpenID Connect  
- **RUM**: Real User Monitoring

### One‑Page Answer Outline (print‑friendly)
1) Clarify FRs + NFRs (numbers!).  
2) U‑P‑D‑S‑O canvas bullets.  
3) 1–2 trade‑off tables; choose and justify.  
4) Perf/security/a11y plan + metrics.  
5) Testing + rollout + risks + next steps.

---

> **Quick recap for memory:**  
> - Use **C‑D‑D‑O‑I** as your spine.  
> - Sketch **U‑P‑D‑S‑O** to ensure completeness.  
> - Always name **numbers** (LCP/JS budget).  
> - Speak **trade‑offs** and **fallbacks**.  
> - End with **operate** (metrics, flags, rollback) and **iterate**.

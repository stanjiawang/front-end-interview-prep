# 🟢 Node.js + Express Case Study: Secure Payments Proxy for **purchase.webex.com**

A practical, interview-ready story describing how to design and implement a **payments proxy** using **Node.js + Express** between **purchase.webex.com** (frontend), **3rd‑party payment vendors**, and **Cisco backend microservices**.

---

## 📑 Table of Contents
- [1. Goals & Context](#1-goals--context)
- [2. Requirements](#2-requirements)
  - [2.1 Functional](#21-functional)
  - [2.2 Non-functional](#22-non-functional)
- [3. Architecture Overview](#3-architecture-overview)
  - [3.1 High-level Diagram](#31-high-level-diagram)
  - [3.2 Key Components](#32-key-components)
- [4. Request Lifecycle & Data Flow](#4-request-lifecycle--data-flow)
- [5. API Design](#5-api-design)
- [6. Express App: Middleware & Infrastructure](#6-express-app-middleware--infrastructure)
- [7. Payment Provider Adapters (Strategy pattern)](#7-payment-provider-adapters-strategy-pattern)
- [8. Webhooks: Signature Verification & Idempotency](#8-webhooks-signature-verification--idempotency)
- [9. Resilience & Reliability](#9-resilience--reliability)
- [10. Security & Compliance](#10-security--compliance)
- [11. Observability](#11-observability)
- [12. Configuration, Rollouts & Operations](#12-configuration-rollouts--operations)
- [13. Testing Strategy](#13-testing-strategy)
- [14. Code Snippets](#14-code-snippets)
- [15. Common Pitfalls \u0026 How We Avoided Them](#15-common-pitfalls--how-we-avoided-them)
- [16. STAR Interview Narrative (ready-to-tell)](#16-star-interview-narrative-ready-to-tell)
- [17. Follow-ups \u0026 Next Steps](#17-follow-ups--next-steps)

---

## 1. Goals & Context

- Build a **secure, resilient** Node.js + Express **proxy service** to sit between **purchase.webex.com** and:
  1) **External payment vendors** (e.g., “Provider A/B” for cards, wallets), and  
  2) **Cisco backend microservices** (orders, subscriptions, entitlements, invoicing).
- Centralize **security**, **routing**, **observability**, **idempotency**, and **compliance controls** without exposing vendors directly to the browser.
- Allow **config-driven vendor selection** by region/product and **gradual rollouts** (canary, feature flags).

---

## 2. Requirements

### 2.1 Functional
- Create/confirm **payment intents** and **captures**, handle **refunds**.
- Receive **webhooks** from vendors (payment succeeded/failed/disputed).
- Update Cisco internal systems (order service, subscription service) after payment events.
- **Idempotency** for all payment‑mutating endpoints.
- **Country/vendor routing** and **failover** to secondary vendor.

### 2.2 Non-functional
- **Security/Compliance**: No card data persistence; forward directly to PCI‑compliant vendors; encrypted transport; audit logs.
- **SLO**: P95 end‑to‑end checkout ≤ 800ms proxy overhead ≤ 50ms; **Availability** ≥ 99.9%.
- **Scalability**: 10k RPS peak with auto‑scale; **Backpressure** protection.
- **Observability**: Correlation IDs, metrics (latency, error rates), distributed tracing.
- **Cost**: Prefer managed caches/queues; efficient connection reuse (HTTP/2/keep‑alive).

---

## 3. Architecture Overview

### 3.1 High-level Diagram

```
+---------------------+           +----------------------+
|  purchase.webex.com |  HTTPS    |  Node.js Express     |
|  (Browser/SPA)      +---------->+  Payments Proxy      |
+----------+----------+           +----------+-----------+
           |                                 |
           |                                 |  (Strategy Adapters)
           |                                 v
           |                      +----------------------+
           |                      | Payment Vendor(s)    |
           |                      | (A / B / Wallets)    |
           |                      +----------+-----------+
           |                                 |
           |         Webhooks (HTTPS,mTLS)   |
           |<--------------------------------+
           |                                 |
           |                                 v
           |                      +----------------------+
           |                      | Cisco Microservices  |
           +--------------------->| Orders / Subs / ERP  |
                                  +----------------------+
```

### 3.2 Key Components
- **Express API**: Receives checkout actions, proxies to vendor adapters, orchestrates internal updates.
- **Adapters**: One per vendor; common interface; handles auth, signature, retries, mapping.
- **Cache/Store**: Redis for **idempotency keys**, rate limiting, and short‑lived tokens.
- **Queue (optional)**: For eventually consistent updates to Cisco microservices.
- **Secrets**: Managed via vault/KMS; never in source.
- **Observability**: Structured logs, metrics, tracing (W3C `traceparent`).

---

## 4. Request Lifecycle & Data Flow

1) Browser initiates **POST `/checkout/intent`** → proxy validates request and creates a vendor payment intent.  
2) Vendor returns **client secret** / **redirect URL**. Proxy responds to browser.  
3) User completes payment (3DS/redirect/etc). Vendor sends **webhook** → proxy verifies signature → marks payment **succeeded/failed**.  
4) On success, proxy **confirms order** by calling Cisco microservices (order, subscription, entitlement).  
5) Proxy emits metrics/logs and returns success to the frontend.

---

## 5. API Design

- `POST /v1/checkout/intent` – create payment intent (idempotent).  
- `POST /v1/checkout/capture` – capture/confirm payment.  
- `POST /v1/webhooks/:vendor` – vendor webhooks (verify HMAC/mTLS).  
- `GET /v1/payments/:id` – get payment status.  
- `POST /v1/refunds` – refund endpoint.  
- `GET /healthz` / `GET /readyz` – health & readiness.  
- `GET /metrics` – Prometheus metrics (auth restricted).

---

## 6. Express App: Middleware & Infrastructure

- **helmet**, **cors** (locked origins), **compression**, **morgan/pino** (structured logs).  
- **Correlation ID**: generate `x-correlation-id` if absent; propagate to vendors & Cisco services.  
- **Validation** (e.g., Zod) for `body/params/query`.  
- **Idempotency** middleware: use Redis keyed by `Idempotency-Key`.  
- **Rate limiting**: per IP/tenant; 429 on abuse.  
- **Timeouts**: 2s vendor calls; 5s internal; fail fast with clear error mapping.  
- **Error handler**: centralized, no stack traces in prod.

---

## 7. Payment Provider Adapters (Strategy pattern)

Define a common interface:
- `createIntent()`, `capture()`, `refund()`, `verifyWebhookSignature()`, `mapStatus()`  
Benefits: swap vendors without changing business logic; A/B by config.

---

## 8. Webhooks: Signature Verification & Idempotency

- Verify **HMAC** or vendor‑specific signature & timestamp; reject replays (nonce/exp).  
- Use **idempotency** store to avoid double processing (same event ID).  
- Respond **2xx** only after durable updates (e.g., enqueue message or write to DB).

---

## 9. Resilience & Reliability

- **Retries** with jittered exponential backoff for **idempotent** operations.  
- **Circuit breaker** per vendor host; failover to secondary vendor by region if configured.  
- **Bulkhead**: limit concurrent vendor calls; **queue** non‑critical post‑payment tasks.  
- **Dead Letter Queue** for webhook failures with replay tools.  

---

## 10. Security & Compliance

- **TLS everywhere**, optional **mTLS** with vendors/webhooks.  
- No storing of **card PAN/CVV**; use vendor tokenization.  
- **Secrets** via KMS/Vault; **rotate keys**.  
- **CSP**, **HSTS**, strict **CORS** for `purchase.webex.com`.  
- **PII minimization**; data retention policies; audit logs.  
- Adhere to **PCI DSS SAQ‑A** style boundary (front sends directly to vendor or via vendor’s SDK; proxy avoids touching raw card data).

---

## 11. Observability

- **Structured logs** (JSON) incl. `correlationId`, `tenant`, `country`, `vendor`, `latencyMs`.  
- **Metrics**: `http_request_duration_seconds`, `vendor_errors_total`, `payment_status_total`.  
- **Tracing**: propagate `traceparent` to vendors/internal; sample at 10%+ in prod.  
- **Dashboards/Alerts**: error rate spikes, p95 latency, webhook backlog, circuit open state.

---

## 12. Configuration, Rollouts & Operations

- **Config‑driven routing**: `country -> vendor`, **feature flags** (Kill‑switch per vendor).  
- **Canary**: route 5% traffic to new adapter; watch dashboards; roll forward/back.  
- **Blue/Green** deploys; **readiness** gates only when vendor health passes.  
- **Runbooks** for vendor outages; manual override to secondary vendor.  

---

## 13. Testing Strategy

- **Unit**: adapter mapping, signature checks, idempotency.  
- **Contract**: mock vendor webhooks & HTTP (Pact/MSW).  
- **Integration**: in‑memory Redis, local queues; end‑to‑end flows.  
- **Load tests**: checkout p95, concurrency, circuit behavior.  
- **Chaos**: inject timeouts/errors from vendors; validate failover.  

---

## 14. Code Snippets

### 14.1 Express App Bootstrap (`app.ts`)
```ts
import express from "express";
import helmet from "helmet";
import cors from "cors";
import compression from "compression";
import morgan from "morgan";
import routes from "./routes";
import { errorHandler } from "./lib/error-handler";
import { correlation } from "./lib/correlation";
import { rateLimiter } from "./lib/rate-limit";

const app = express();
app.set("trust proxy", true);
app.use(helmet());
app.use(cors({ origin: ["https://purchase.webex.com"], credentials: true }));
app.use(compression());
app.use(express.json({ limit: "1mb" }));
app.use(morgan("combined"));
app.use(correlation()); // inject x-correlation-id
app.use(rateLimiter());

app.use("/v1", routes);

app.get("/healthz", (_req, res) => res.send("ok"));
app.get("/readyz", (_req, res) => res.send("ready"));

app.use(errorHandler);

export default app;
```

### 14.2 Correlation ID Middleware (`lib/correlation.ts`)
```ts
import { v4 as uuid } from "uuid";
import { Request, Response, NextFunction } from "express";

export const correlation = () => (req: Request, res: Response, next: NextFunction) => {
  const id = req.header("x-correlation-id") || uuid();
  res.setHeader("x-correlation-id", id);
  (req as any).correlationId = id;
  next();
};
```

### 14.3 Idempotency Middleware (Redis) (`lib/idempotency.ts`)
```ts
import type { Request, Response, NextFunction } from "express";
import { createClient } from "redis";

const redis = createClient({ url: process.env.REDIS_URL });
redis.connect();

export function idempotent(ttlSeconds = 3600) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const key = req.header("Idempotency-Key");
    if (!key) return res.status(400).json({ message: "Missing Idempotency-Key" });

    const cached = await redis.get(key);
    if (cached) return res.status(200).json(JSON.parse(cached));

    // capture res.json to store the response once
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      redis.setEx(key, ttlSeconds, JSON.stringify(body)).catch(() => {});
      return originalJson(body);
    };
    next();
  };
}
```

### 14.4 Payment Provider Interface & Factory (`payments/provider.ts`)
```ts
export type CreateIntentInput = { amount: number; currency: string; customerId: string };
export type CreateIntentResult = { intentId: string; clientSecret?: string; redirectUrl?: string };

export interface PaymentProvider {
  createIntent(input: CreateIntentInput, ctx: Ctx): Promise<CreateIntentResult>;
  capture(intentId: string, ctx: Ctx): Promise<{ status: "succeeded" | "failed" | "pending" }>;
  refund(paymentId: string, amount: number, ctx: Ctx): Promise<{ refundId: string }>;
  verifyWebhookSignature(rawBody: string, headers: Record<string,string>): boolean;
  mapStatus(vendorStatus: string): "succeeded" | "failed" | "pending";
}

export type Ctx = { correlationId: string; country: string; tenant?: string };

export function providerFor(country: string): PaymentProvider {
  // config-driven routing
  if (country === "US") return new ProviderA();
  if (country === "DE") return new ProviderB();
  return new ProviderA(); // default
}

// dummy classes to illustrate
class ProviderA implements PaymentProvider { /* ... implement HTTP calls ... */ }
class ProviderB implements PaymentProvider { /* ... */ }
```

### 14.5 Vendor Adapter with Backoff & Timeouts (Example)
```ts
import fetch, { RequestInit } from "node-fetch";

async function callVendor(endpoint: string, init: RequestInit, attempts = 3) {
  const timeout = 2000;
  for (let i = 0; i < attempts; i++) {
    const ctrl = new AbortController();
    const t = setTimeout(() => ctrl.abort(), timeout + i * 250);
    try {
      const res = await fetch(endpoint, { ...init, signal: ctrl.signal });
      if (res.ok) return res.json();
      if (res.status >= 500) throw new Error(`Vendor 5xx: ${res.status}`);
      // 4xx: no retry
      return await res.json();
    } catch (e) {
      if (i === attempts - 1) throw e;
      await new Promise(r => setTimeout(r, 200 * Math.pow(2, i) + Math.random()*100));
    } finally {
      clearTimeout(t);
    }
  }
}
```

### 14.6 Routes (Intent + Capture) (`routes/payments.ts`)
```ts
import { Router } from "express";
import { idempotent } from "../lib/idempotency";
import { providerFor } from "../payments/provider";

const router = Router();

router.post("/checkout/intent", idempotent(), async (req, res, next) => {
  try {
    const { amount, currency, customerId, country } = req.body;
    const prov = providerFor(country);
    const result = await prov.createIntent({ amount, currency, customerId }, { correlationId: (req as any).correlationId, country });
    res.status(201).json(result);
  } catch (e) { next(e); }
});

router.post("/checkout/capture", idempotent(), async (req, res, next) => {
  try {
    const { intentId, country } = req.body;
    const prov = providerFor(country);
    const result = await prov.capture(intentId, { correlationId: (req as any).correlationId, country });
    res.json(result);
  } catch (e) { next(e); }
});

export default router;
```

### 14.7 Webhook Handler (`routes/webhooks.ts`)
```ts
import { Router } from "express";
import { providerFor } from "../payments/provider";

const router = Router();

// Use raw body for signature verification
router.post("/:vendor", express.raw({ type: "*/*" }), async (req, res) => {
  const vendor = req.params.vendor;
  const country = req.query.country as string || "US";
  const prov = providerFor(country);

  const raw = (req as any).body as Buffer;
  const ok = prov.verifyWebhookSignature(raw.toString("utf8"), req.headers as any);
  if (!ok) return res.status(400).send("bad signature");

  // Parse event, map status, update internal services (async/queue)
  // ... publish to Kafka/SQS or call Cisco microservices ...
  res.status(200).send("ok");
});

export default router;
```

### 14.8 Error Handler (`lib/error-handler.ts`)
```ts
import { Request, Response, NextFunction } from "express";

export function errorHandler(err: any, req: Request, res: Response, _next: NextFunction) {
  const status = err.status || 500;
  const correlationId = (req as any).correlationId;
  console.error(JSON.stringify({ level: "error", status, message: err.message, correlationId }));
  res.status(status).json({ message: "Internal Error", correlationId });
}
```

---

## 15. Common Pitfalls & How We Avoided Them
- **Double charges** → enforced **idempotency keys** on create/capture + vendor idempotent keys.  
- **Webhook replays** → signature timestamp + nonce store; dedupe by `eventId`.  
- **Vendor flakiness** → retries + circuit breaker + failover vendor.  
- **Leaking PII** → strict logging filters, data minimization, encryption in transit.  
- **Race conditions** between webhook and client poll → persisted payment state + last‑write‑wins with event versioning.  
- **CORS errors** → allowlist `purchase.webex.com` only, preflight tuned.  

---

## 16. STAR Interview Narrative (ready-to-tell)
- **Situation**: Webex checkout needed a unified payments proxy to isolate vendors and standardize security & observability.  
- **Task**: Design & build a Node.js/Express service enabling multi‑vendor routing, secure webhooks, and integration to Cisco microservices.  
- **Action**: Implemented strategy‑based vendor adapters, Redis idempotency, signature‑verified webhooks, circuit breakers, and config‑driven routing. Added correlation IDs, metrics, and structured logs. Ran canary rollout by country.  
- **Result**: Reduced checkout errors by **32%**, cut P95 proxy latency to **<45ms**, enabled fast onboarding of a second vendor, and improved incident MTTR via tracing/metrics.

---

## 17. Follow-ups & Next Steps
- Add **on‑demand replays** for failed webhooks (DLQ UI).  
- Multi‑region active/active with global load balancing.  
- Tokenized **vault** for limited PII, automated retention policy.  
- Expand **A/B routing** by product tier and experiment platform integration.

---

**Tip for interviews:** Emphasize **idempotency, signature verification, resilience, and observability**. Bring latency/error graphs and a simple sequence diagram.

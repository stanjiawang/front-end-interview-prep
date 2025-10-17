# 🚀 Express.js Interview Preparation Guide

A comprehensive guide to **Express.js** for building server-side APIs and apps.  
Covers fundamentals, routing, middleware, security, auth, testing, performance, and **frequently asked interview Q&A** — with copy‑paste code snippets.

---

## 📑 Table of Contents
- [1. What is Express.js?](#1-what-is-expressjs)
- [2. Install & Project Setup](#2-install--project-setup)
- [3. App Structure](#3-app-structure)
- [4. Routing](#4-routing)
- [5. Middleware](#5-middleware)
- [6. Error Handling](#6-error-handling)
- [7. Async Patterns](#7-async-patterns)
- [8. Validation](#8-validation)
- [9. Authentication & Authorization](#9-authentication--authorization)
- [10. Security Hardening](#10-security-hardening)
- [11. Files & Uploads](#11-files--uploads)
- [12. Logging & Monitoring](#12-logging--monitoring)
- [13. Testing (Jest + Supertest)](#13-testing-jest--supertest)
- [14. Performance & Scaling](#14-performance--scaling)
- [15. Realtime & Streaming](#15-realtime--streaming)
- [16. REST vs GraphQL](#16-rest-vs-graphql)
- [17. Pagination, Filtering, Versioning](#17-pagination-filtering-versioning)
- [18. Env & Configuration](#18-env---configuration)
- [19. Deployment Tips](#19-deployment-tips)
- [20. Common Interview Questions](#20-common-interview-questions)

---

## 1. What is Express.js?
Express.js is a **minimal, unopinionated** web framework for Node.js.  
- Provides routing, middleware, and HTTP utilities.  
- Lets you assemble your own stack (ORM, view engine, security, etc.).  

---

## 2. Install & Project Setup
```bash
mkdir express-app && cd $_
npm init -y
npm install express
npm i -D nodemon typescript @types/express ts-node @types/node
```

**Scripts (package.json):**
```json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/index.ts",
    "start": "node dist/index.js",
    "build": "tsc"
  }
}
```

---

## 3. App Structure
```
src/
  index.ts
  app.ts
  routes/
    user.routes.ts
  controllers/
    user.controller.ts
  middlewares/
    auth.ts
    error.ts
  validators/
    user.schema.ts
  services/
    user.service.ts
```

**src/app.ts**
```ts
import express from "express";
import userRouter from "./routes/user.routes";

const app = express();
app.use(express.json()); // body parser
app.use("/api/users", userRouter);

export default app;
```

**src/index.ts**
```ts
import app from "./app";
const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`API on http://localhost:${port}`));
```

---

## 4. Routing
**Basic:**
```ts
import { Router } from "express";
const router = Router();

router.get("/", (req, res) => res.json({ ok: true }));
router.get("/:id", (req, res) => res.json({ id: req.params.id }));
router.post("/", (req, res) => res.status(201).json(req.body));
router.put("/:id", (req, res) => res.json({ id: req.params.id, ...req.body }));
router.delete("/:id", (req, res) => res.status(204).send());

export default router;
```

**Router-level middleware & params:**
```ts
router.param("id", (req, _res, next, id) => { (req as any).userId = id; next(); });
router.get("/:id/profile", (req, res) => res.json({ userId: (req as any).userId }));
```

---

## 5. Middleware
- **Application-level**: `app.use(fn)`
- **Router-level**: `router.use(fn)`
- **Built-in**: `express.json`, `express.urlencoded`, `express.static`
- **Third-party**: `cors`, `helmet`, `morgan`, `cookie-parser`, `compression`

```ts
import cors from "cors";
import helmet from "helmet";
import morgan from "morgan";
import cookieParser from "cookie-parser";

app.use(helmet());
app.use(cors({ origin: ["http://localhost:3000"], credentials: true }));
app.use(morgan("dev"));
app.use(cookieParser());
```

---

## 6. Error Handling
Centralized error handler (must have 4 params):
```ts
// middlewares/error.ts
import { Request, Response, NextFunction } from "express";

export function errorHandler(err: any, _req: Request, res: Response, _next: NextFunction) {
  const status = err.status || 500;
  res.status(status).json({ message: err.message || "Internal Server Error" });
}
```

Register after routes:
```ts
import { errorHandler } from "./middlewares/error";
app.use(errorHandler);
```

Create errors:
```ts
export class HttpError extends Error { constructor(public status: number, message: string){ super(message); } }
throw new HttpError(404, "User not found");
```

---

## 7. Async Patterns
Use async/await with a tiny wrapper to forward errors:
```ts
const asyncHandler = (fn: any) => (req: any, res: any, next: any) =>
  Promise.resolve(fn(req, res, next)).catch(next);

router.get("/:id", asyncHandler(async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) throw new HttpError(404, "User not found");
  res.json(user);
}));
```

---

## 8. Validation
Use **Zod** (or Joi/Yup) for request validation.

```ts
// validators/user.schema.ts
import { z } from "zod";
export const createUserSchema = z.object({
  body: z.object({
    name: z.string().min(1),
    email: z.string().email(),
  }),
});

// middleware
export const validate =
  (schema: any) => (req: any, _res: any, next: any) => {
    schema.parse({ body: req.body, params: req.params, query: req.query });
    next();
  };

// route
router.post("/", validate(createUserSchema), (req, res) => res.status(201).json(req.body));
```

---

## 9. Authentication & Authorization
**JWT (stateless):**
```ts
import jwt from "jsonwebtoken";

const SECRET = process.env.JWT_SECRET || "dev-secret";

export function signJwt(payload: object) {
  return jwt.sign(payload, SECRET, { expiresIn: "1h" });
}

export function auth(req: any, res: any, next: any) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ message: "No token" });
  try {
    req.user = jwt.verify(token, SECRET);
    next();
  } catch {
    res.status(401).json({ message: "Invalid token" });
  }
}

// usage
router.get("/me", auth, (req, res) => res.json({ user: req.user }));
```

**Sessions (stateful):**
```ts
import session from "express-session";
app.use(session({ secret: "keyboard cat", resave: false, saveUninitialized: false, cookie: { secure: false } }));
```

Role-based auth:
```ts
const requireRole = (role: string) => (req: any, res: any, next: any) =>
  req.user?.role === role ? next() : res.status(403).json({ message: "Forbidden" });
```

---

## 10. Security Hardening
- `helmet()` for secure headers  
- `cors()` with explicit origins  
- Input validation & sanitize  
- **Rate limiting**  
- Avoid blocking operations on event loop  
- Hide stack traces in production  

```ts
import rateLimit from "express-rate-limit";
const limiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });
app.use(limiter);
```

---

## 11. Files & Uploads
Use **multer**:
```ts
import multer from "multer";
const upload = multer({ dest: "uploads/" });

router.post("/avatar", upload.single("avatar"), (req, res) => {
  res.json({ file: req.file });
});
```

---

## 12. Logging & Monitoring
- **morgan** for HTTP logs  
- **winston/pino** for app logs  
- Health checks: `/health`, `/ready`  
- Observability: APM (Datadog, NewRelic), OpenTelemetry

```ts
import winston from "winston";
const logger = winston.createLogger({ transports: [new winston.transports.Console()] });
logger.info("App started");
```

---

## 13. Testing (Jest + Supertest)
```bash
npm i -D jest ts-jest @types/jest supertest @types/supertest
npx ts-jest config:init
```

**app.test.ts**
```ts
import request from "supertest";
import app from "../src/app";

describe("GET /api/users", () => {
  it("responds with ok", async () => {
    const res = await request(app).get("/api/users");
    expect(res.status).toBe(200);
  });
});
```

---

## 14. Performance & Scaling
- **Compression** (`compression` middleware)  
- **Caching** (etag, Redis)  
- **Clustering** with PM2 or Node cluster  
- Connection pooling at DB layer  
- Use Node 18+ with native fetch & better V8

```ts
import compression from "compression";
app.use(compression());
```

Cluster (PM2):
```bash
pm2 start dist/index.js -i max
```

---

## 15. Realtime & Streaming
**Server‑Sent Events (SSE):**
```ts
router.get("/events", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.flushHeaders();
  const timer = setInterval(() => res.write(`data: ${Date.now()}

`), 1000);
  req.on("close", () => clearInterval(timer));
});
```

**WebSockets (socket.io):**
```ts
import { createServer } from "http";
import { Server } from "socket.io";
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: "*" } });
io.on("connection", (socket) => socket.emit("hello", "world"));
httpServer.listen(3000);
```

---

## 16. REST vs GraphQL
- **REST**: resources via routes; caching via HTTP; simple & widespread.  
- **GraphQL**: typed schema, single endpoint, flexible queries.  

**Apollo Server + Express (example):**
```ts
import { ApolloServer } from "@apollo/server";
import { expressMiddleware } from "@apollo/server/express4";
import bodyParser from "body-parser";

const typeDefs = `#graphql
  type Query { hello: String! }
`;
const resolvers = { Query: { hello: () => "world" } };

const server = new ApolloServer({ typeDefs, resolvers });
await server.start();
app.use("/graphql", bodyParser.json(), expressMiddleware(server));
```

---

## 17. Pagination, Filtering, Versioning
```ts
// Pagination
router.get("/", async (req, res) => {
  const page = Math.max(parseInt(req.query.page as string) || 1, 1);
  const limit = Math.min(parseInt(req.query.limit as string) || 20, 100);
  const skip = (page - 1) * limit;
  const items = await listUsers({ skip, limit });
  res.json({ page, limit, items });
});

// Versioning
app.use("/v1", v1Router);
app.use("/v2", v2Router);
```

---

## 18. Env &  Configuration
```bash
npm i dotenv
```
```ts
import "dotenv/config";
const { PORT = 3000, NODE_ENV = "development" } = process.env;
```
- Keep secrets out of source control; use `.env` and CI/CD secrets.

---

## 19. Deployment Tips
- Reverse proxy with **Nginx**  
- Enable **HTTPS** (certs)  
- Zero‑downtime deploys with PM2  
- Containerize with Docker; health checks  
- Observe memory/leaks; set `NODE_ENV=production`  

---

## 20. Common Interview Questions

### Q1. How does Express middleware work?
**A:** Middleware are functions with signature `(req, res, next)`. They can modify the request/response or short‑circuit the pipeline, or call `next()` to pass control.

### Q2. Difference between app.use and app.get?
**A:** `app.use` registers middleware for **all** HTTP methods (optionally scoped by path). `app.get` registers a **route handler** for GET only.

### Q3. How do you handle errors in Express?
**A:** Use a centralized error handler with **four params** `(err, req, res, next)` registered **after** routes. Throw/forward errors to it.

### Q4. How do you protect routes?
**A:** JWT or sessions. For JWT, verify token in middleware and attach `req.user`; for sessions, use `express-session` and a store (Redis).

### Q5. How to prevent common attacks (XSS, CSRF)?
**A:** `helmet`, input validation, output encoding; CSRF tokens for state‑changing requests (if using cookies); CORS with allow‑list.

### Q6. How to scale Express?
**A:** Horizontal scaling with PM2/cluster behind a load balancer; use caching (Redis), DB pooling, and ensure handlers are non‑blocking.

### Q7. How to validate requests?
**A:** Use libraries like **Zod**/**Joi** to validate `req.body/params/query` and return 400 on failures.

### Q8. What is the difference between next() and return res.json(...)?
**A:** `next()` passes control to the next middleware; returning a response ends the cycle. Avoid calling `next()` **after** sending a response.

### Q9. How do you organize large Express apps?
**A:** Layered architecture: routes → controllers → services → repositories; feature folders; shared middlewares & utils.

### Q10. What are common performance pitfalls?
**A:** Blocking CPU tasks on event loop, unbounded JSON parsing, N+1 DB queries, missing indexes, no caching, logging synchronous IO.

---

🎯 **Final tip:** In interviews, demonstrate you can combine **routing + validation + auth + error handling + security** into a clean, testable Express app.

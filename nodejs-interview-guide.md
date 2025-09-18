# 🟢 Node.js Interview Preparation Guide

A complete guide to **Node.js** for backend/server-side interviews.  
Includes fundamentals, server-building basics, ecosystem tools, and **common interview questions with answers**.

---

## 📑 Table of Contents
- [1. Introduction](#1-introduction)
- [2. Core Concepts](#2-core-concepts)
  - [2.1 Event Loop](#21-event-loop)
  - [2.2 Non-blocking I/O](#22-non-blocking-io)
  - [2.3 Modules](#23-modules)
- [3. Building a Server with Node.js](#3-building-a-server-with-nodejs)
  - [3.1 Using http module](#31-using-http-module)
  - [3.2 Using Express.js](#32-using-expressjs)
- [4. Middleware](#4-middleware)
- [5. Working with Databases](#5-working-with-databases)
- [6. Authentication](#6-authentication)
- [7. Error Handling](#7-error-handling)
- [8. Performance & Scaling](#8-performance--scaling)
- [9. Common Interview Questions](#9-common-interview-questions)

---

## 1. Introduction

Node.js is a **JavaScript runtime built on Chrome’s V8 engine**.  
- Uses an **event-driven, non-blocking I/O model**.  
- Great for building **scalable network applications**.  
- Single-threaded with a **libuv** thread pool for async operations.  

---

## 2. Core Concepts

### 2.1 Event Loop
The event loop handles async operations (I/O, timers, promises).  
Phases: timers → I/O callbacks → idle → poll → check → close.  

### 2.2 Non-blocking I/O
Node delegates tasks to the system (via libuv), avoids blocking the main thread.  

### 2.3 Modules
- CommonJS (`require`)  
- ES Modules (`import`)  

```js
// CommonJS
const fs = require("fs");

// ES Modules
import fs from "fs";
```

---

## 3. Building a Server with Node.js

### 3.1 Using http module
```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello, Node.js Server!");
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

### 3.2 Using Express.js
Express simplifies server creation with routing & middleware.

```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello from Express!");
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

---

## 4. Middleware

Middleware are functions that execute during request/response lifecycle.

```js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

---

## 5. Working with Databases

Example with MongoDB (Mongoose):

```js
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/testdb");

const UserSchema = new mongoose.Schema({ name: String, age: Number });
const User = mongoose.model("User", UserSchema);

app.get("/users", async (req, res) => {
  const users = await User.find();
  res.json(users);
});
```

---

## 6. Authentication

JWT-based authentication:

```js
const jwt = require("jsonwebtoken");

app.post("/login", (req, res) => {
  const token = jwt.sign({ userId: 123 }, "secret", { expiresIn: "1h" });
  res.json({ token });
});

function verifyToken(req, res, next) {
  const token = req.headers["authorization"];
  if (!token) return res.status(403).send("No token");

  jwt.verify(token, "secret", (err, decoded) => {
    if (err) return res.status(401).send("Invalid token");
    req.userId = decoded.userId;
    next();
  });
}
```

---

## 7. Error Handling

Centralized error handler:

```js
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send("Something broke!");
});
```

---

## 8. Performance & Scaling

- Use **cluster** module or PM2 for multi-core utilization.  
- Cache frequently used data (Redis).  
- Use load balancers for horizontal scaling.  

```js
const cluster = require("cluster");
const os = require("os");

if (cluster.isMaster) {
  os.cpus().forEach(() => cluster.fork());
} else {
  const http = require("http");
  http.createServer((req, res) => res.end("Hello")).listen(3000);
}
```

---

## 9. Common Interview Questions

### Q1. What is the event loop in Node.js?
**A:** It’s the mechanism that handles async operations. Node.js is single-threaded, but the event loop enables concurrency via callbacks, promises, and async/await.

---

### Q2. Difference between process.nextTick(), setImmediate(), and setTimeout()?
**A:**  
- `process.nextTick()` → executes after current operation, before next event loop phase.  
- `setImmediate()` → executes in the check phase.  
- `setTimeout()` → executes after specified delay.  

---

### Q3. How do you handle asynchronous code in Node.js?
**A:** Using callbacks, promises, async/await.

---

### Q4. How do you secure a Node.js application?
**A:** Use HTTPS, input validation, helmet middleware, rate limiting, JWTs, parameterized queries to prevent SQL injection.

---

### Q5. Difference between CommonJS and ES Modules?
**A:** CommonJS uses `require`, synchronous; ES Modules use `import/export`, asynchronous, supported natively since Node.js 13+.

---

### Q6. How do you scale Node.js applications?
**A:** Use clustering, PM2, load balancing, containerization (Docker, Kubernetes).

---

### Q7. What are streams in Node.js?
**A:** Streams are objects for reading/writing data sequentially. Types: Readable, Writable, Duplex, Transform.

```js
const fs = require("fs");
fs.createReadStream("input.txt").pipe(fs.createWriteStream("output.txt"));
```

---

### Q8. What are some common security issues in Node.js?
**A:**  
- Injection attacks  
- Cross-site scripting (XSS)  
- Cross-site request forgery (CSRF)  
- Insecure deserialization  
- Solution: input sanitization, escaping, security libraries.

---

### Q9. Explain middleware in Express.
**A:** Middleware functions have access to `req`, `res`, and `next()`. They can modify requests, end responses, or call the next middleware.

---

### Q10. Difference between synchronous and asynchronous programming in Node.js?
**A:** Sync blocks the thread until completion, async allows other operations to run while waiting for results.

---

✍️ **Pro tip for interviews:** Always tie Node.js answers back to **event loop**, **non-blocking I/O**, and **scalability**.

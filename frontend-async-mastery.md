# Frontend Async Mastery — Callbacks, Promises, async/await, and Beyond

> A compact, practical guide to **how async works in the browser**, how to use each pattern, how to **handle errors** robustly, and the **most common interview questions** (with answers + code).

---

## Table of Contents
1. [Mental Model: Event Loop, Tasks & Timing](#mental-model-event-loop-tasks--timing)
2. [Callbacks](#callbacks)
3. [Promises](#promises)
4. [async/await](#asyncawait)
5. [Generators & `co`-style (FYI)](#generators--co-style-fyi)
6. [XHR, `fetch`, and common patterns](#xhr-fetch-and-common-patterns)
7. [Control Flow & Concurrency](#control-flow--concurrency)
8. [Cancellation (`AbortController`)](#cancellation-abortcontroller)
9. [Error Handling Patterns](#error-handling-patterns)
10. [Performance & UX Patterns](#performance--ux-patterns)
11. [Web Workers & Off‑Main‑Thread](#web-workers--offmainthread)
12. [Common Interview Questions & Answers](#common-interview-questions--answers)
13. [Cheat Sheet](#cheat-sheet)

---

## Mental Model: Event Loop, Tasks & Timing

- **Call stack** runs synchronous JS.
- **Task queues** hold work to run later:
  - **Macrotasks**: `setTimeout`, `setInterval`, DOM events, message channel, XHR callbacks.
  - **Microtasks**: **Promise callbacks** (`.then/.catch/.finally`), `queueMicrotask`, `MutationObserver`.
- **Event loop**: after the stack is empty → drain **all microtasks** → then take **one** macrotask → repeat.

```js
console.log('A');
setTimeout(() => console.log('B (macro)'), 0);
Promise.resolve().then(() => console.log('C (micro)'));
console.log('D');
// Order: A, D, C (microtasks), B (macrotask)
```

**Key rule:** Microtasks run **before** next macrotask. This is the root of many timing puzzles.

---

## Callbacks

### When to use
- Legacy APIs (`fs` in Node-style, older libs), DOM/event handlers.
- Simple one-off async operations (e.g., `img.onload`/`onerror`).

### Why (pros/cons)
- ✅ Very lightweight; no Promise overhead.
- ❌ **Callback hell** (nested pyramids), error propagation is manual, hard to compose.

### Basic pattern
```js
function loadImage(url, cb) {
  const img = new Image();
  img.onload = () => cb(null, img);
  img.onerror = (err) => cb(err || new Error('Image load failed'));
  img.src = url;
}

loadImage('/logo.webp', (err, img) => {
  if (err) return console.error(err);
  document.body.append(img);
});
```

### Avoiding “callback hell” (Node-style)
```js
function doA(a, cb)   { setTimeout(() => cb(null, a+1), 50); }
function doB(b, cb)   { setTimeout(() => cb(null, b*2), 50); }
function doC(c, cb)   { setTimeout(() => cb(null, c-3), 50); }

doA(1, (e, r1) => {
  if (e) return cb(e);
  doB(r1, (e, r2) => {
    if (e) return cb(e);
    doC(r2, (e, r3) => {
      if (e) return cb(e);
      console.log('result', r3);
    });
  });
});
```

---

## Promises

### Why Promises
- **Composability**: `.then/.catch/.finally`, `Promise.all|race|allSettled|any`.
- **Error funneling**: Throw → becomes rejected promise → catch once.

### Creating & consuming
```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve(42), 100);
});

p.then(v => v * 2)
 .then(v => console.log(v))       // 84
 .catch(err => console.error(err))
 .finally(() => console.log('done'));
```

### Chaining & returning promises
```js
fetch('/api/user')
  .then(r => r.json())      // return a promise → next then waits
  .then(user => fetch(`/api/orders?u=${user.id}`))
  .then(r => r.json())
  .then(orders => console.log(orders))
  .catch(console.error);
```

### Composition helpers
```js
// Run in parallel, fail fast
await Promise.all([fetchA(), fetchB()]); 

// Run in parallel, collect results + errors
const results = await Promise.allSettled([fetchA(), fetchB()]);

// First to settle (success OR failure)
const first = await Promise.race([slow(), fast()]);

// First to fulfill (ignores rejections unless ALL reject)
const one = await Promise.any([maybeFail(), succeed()]);
```

**Microtasks:** All `.then/.catch/.finally` schedule microtasks.

---

## async/await

### Why
- Reads like **synchronous code**; easier error handling with `try/catch`.
- Under the hood: **syntactic sugar** over Promises.

### Usage
```js
async function getUserOrders() {
  try {
    const user = await fetch('/api/user').then(r => r.json());
    const orders = await fetch(`/api/orders?u=${user.id}`).then(r => r.json());
    return orders;
  } catch (err) {
    // one place to handle errors
    console.error(err);
    throw err; // rethrow if caller should know
  }
}
```

### Parallel vs sequential with `await`
```js
// ❌ Sequential (slow)
const a = await fetchA();
const b = await fetchB();

// ✅ Parallel (fast)
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

### Top-level `await`
- Supported in **ES modules**: `type="module"` in browser, or `.mjs`.
```js
const data = await fetch('/api/data').then(r => r.json());
```

---

## Generators & `co`-style (FYI)
Before async/await, teams used **generators** + runners to yield Promises.

```js
function* task() {
  const a = yield fetchA(); // yield a Promise
  const b = yield fetchB(a);
  return b;
}
// A runner (`co`) auto-resumes when each yield resolves.
```

You’ll rarely write this today; know it for historical context.

---

## XHR, `fetch`, and common patterns

### `fetch` basics
```js
const res = await fetch('/api/items');
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const json = await res.json();
```

### JSON POST with timeout
```js
async function postJSON(url, data, { timeout = 8000 } = {}) {
  const ctrl = new AbortController();
  const t = setTimeout(() => ctrl.abort(), timeout);
  try {
    const res = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
      signal: ctrl.signal,
      credentials: 'include', // if you need cookies
    });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(t);
  }
}
```

### `XMLHttpRequest` (rare, but interviewers ask)
```js
function get(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.onload = () => (xhr.status >= 200 && xhr.status < 300)
      ? resolve(xhr.responseText)
      : reject(new Error(`HTTP ${xhr.status}`));
    xhr.onerror = () => reject(new Error('Network error'));
    xhr.send();
  });
}
```

---

## Control Flow & Concurrency

### Sequential vs Parallel
```js
// Sequential
for (const id of ids) {
  const item = await fetchItem(id);
  console.log(item);
}

// Parallel (limit unbounded fan‑out carefully)
const items = await Promise.all(ids.map(fetchItem));
```

### Concurrency limits (pooling)
```js
async function mapPool(items, worker, limit = 4) {
  const out = new Array(items.length);
  let i = 0;
  const runners = Array.from({ length: limit }, async () => {
    while (i < items.length) {
      const idx = i++;
      out[idx] = await worker(items[idx], idx);
    }
  });
  await Promise.all(runners);
  return out;
}

// Usage
await mapPool(ids, (id) => fetchItem(id), 5);
```

### Debounce / Throttle (classic UI async)
```js
// Debounce: wait for quiet period
function debounce(fn, wait = 200) {
  let t;
  return (...args) => {
    clearTimeout(t);
    t = setTimeout(() => fn(...args), wait);
  };
}

// Throttle: run at most every N ms
function throttle(fn, wait = 200) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= wait) {
      last = now;
      fn(...args);
    }
  };
}
```

---

## Cancellation (`AbortController`)

```js
const ctrl = new AbortController();
const p = fetch('/api/slow', { signal: ctrl.signal });
// later
ctrl.abort();

try {
  await p;
} catch (e) {
  if (e.name === 'AbortError') {
    console.log('Request cancelled');
  } else {
    throw e;
  }
}
```

- Works with `fetch`, `ReadableStream`, and many modern APIs.
- For your own async functions, **accept a `signal`** and stop work early:
```js
async function compute(signal) {
  for (let i = 0; i < 1e6; i++) {
    if (signal.aborted) throw new DOMException('Aborted', 'AbortError');
    // do work chunk
    if (i % 5000 === 0) await new Promise(r => setTimeout(r)); // yield
  }
}
```

---

## Error Handling Patterns

### try/catch with async/await
```js
try {
  const data = await getData();
} catch (err) {
  // classify & report
  if (err.name === 'AbortError') return;
  reportError(err);
  showToast('Something went wrong.');
}
```

### Guard HTTP status
```js
async function getJSON(res) {
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

### Retrying with backoff + jitter
```js
async function retry(fn, { retries = 3, base = 300 } = {}) {
  let attempt = 0;
  while (true) {
    try { return await fn(); }
    catch (e) {
      attempt++;
      if (attempt > retries) throw e;
      const jitter = Math.random() * base;
      const delay = base * 2 ** (attempt - 1) + jitter;
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

### Timeouts
```js
function timeout(ms, msg = 'Timeout') {
  return new Promise((_, reject) => setTimeout(() => reject(new Error(msg)), ms));
}

await Promise.race([fetch(url), timeout(5000)]);
```

### Aggregate errors
```js
try {
  const one = await Promise.any([fail1(), fail2(), succeed3()]);
} catch (e) {
  if (e instanceof AggregateError) {
    for (const err of e.errors) console.error(err);
  }
}
```

---

## Performance & UX Patterns

- **Batching**: combine multiple small requests into one.
- **Prefetch**: background-fetch likely-next data (on hover/visible).
- **Cache**: use HTTP caching + client cache (Memoization / SWR / React Query).
- **Streaming**: `ReadableStream` and `Response.body` for progressive UI.
- **`requestIdleCallback`** for low‑priority work; **`requestAnimationFrame`** for visual updates.
- **Microtask traps**: Avoid unbounded `.then()` chains that starve rendering; yield control with a macrotask if needed.

```js
// Yield back to rendering
await new Promise(r => setTimeout(r, 0));
```

---

## Web Workers & Off‑Main‑Thread

- Run CPU-heavy code off the UI thread.
```js
// main.js
const worker = new Worker(new URL('./worker.js', import.meta.url), { type: 'module' });
worker.postMessage({ input });
worker.onmessage = (e) => render(e.data);

// worker.js
self.onmessage = (e) => {
  const result = heavyCompute(e.data.input);
  self.postMessage(result);
};
```

- For **module workers**, use `{ type: 'module' }` and ESM imports.
- For shared state across tabs/workers: `BroadcastChannel`, `SharedArrayBuffer` (advanced).

---

## Common Interview Questions & Answers

### 1) Explain the event loop and the difference between microtasks and macrotasks.
- **Answer:** JS is single-threaded; async work is queued. **Microtasks** (Promises) run **after the current stack** and **before** the next macrotask. **Macrotasks** include timers, IO, UI events.

### 2) What’s the output?
```js
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
// 1, 4, 3, 2
```

### 3) Why is `await` inside a loop sometimes slow? How to fix?
- **Answer:** Sequential awaits serialize requests. Use `Promise.all` or a **concurrency pool** to parallelize with limits.

### 4) Difference: `Promise.all` vs `Promise.any` vs `Promise.allSettled` vs `Promise.race`?
- **Answer:** `all` waits for **all** (fails fast). `any` waits for **first fulfilled** (throws `AggregateError` if all reject). `allSettled` waits for all, never rejects. `race` settles on the **first settled** (fulfill or reject).

### 5) How do you cancel a `fetch`?
- **Answer:** `AbortController` + `signal`; handle `AbortError`.

### 6) How to implement a timeout for any promise?
- **Answer:** `Promise.race([task, timeout(ms)])` pattern.

### 7) Show a retry with exponential backoff.
- **Answer:** See [Retrying with backoff](#retrying-with-backoff--jitter). Discuss **idempotency**.

### 8) Why does `await` not block the event loop?
- **Answer:** `await` yields control; the function returns a **pending Promise**. The event loop continues processing other tasks.

### 9) What is a “thenable”?
- **Answer:** Any object with a `.then` method. Promises assimilate thenables per spec (can be a pitfall).

### 10) How to handle partial failures when fetching multiple endpoints?
- **Answer:** Use `Promise.allSettled`, merge successful results, surface errors appropriately.

### 11) Is `fetch` failing on HTTP 404?
- **Answer:** No, it resolves; you must check `res.ok`/`res.status`. Network errors reject.

### 12) Why might a `.then()` handler not run?
- **Answer:** The promise never settles; or synchronous error thrown earlier wasn’t caught; or a preflight/CORS error caused the browser to block (in devtools shows as network error).

### 13) How to avoid unhandled promise rejections?
- **Answer:** Always end chains with `.catch`, or wrap `await` in `try/catch`. In Node/browsers, listen to unhandled rejection events for logging.

### 14) How do you stream a large response to avoid blocking?
- **Answer:** Use `ReadableStream` with `res.body` and process chunks; update UI progressively.

```js
const res = await fetch('/stream');
const reader = res.body.getReader();
let done, value;
while ({ done, value } = await reader.read(), !done) {
  // process Uint8Array chunk
}
```

---

## Cheat Sheet

- Prefer **async/await** + `try/catch` for readability.
- Use `Promise.all` for parallel; **pool** for controlled concurrency.
- Always check `res.ok` with `fetch`; add **timeouts**, **retries**, **cancellation** as needed.
- Understand **microtask vs macrotask**. Know classic ordering puzzles.
- Use **AbortController** for cancel; surface `AbortError` gracefully.
- For UX: debounce/throttle, idle/animation frames, cache, prefetch.
- Offload heavy work to **Web Workers**.

---

**Happy shipping!** ✨

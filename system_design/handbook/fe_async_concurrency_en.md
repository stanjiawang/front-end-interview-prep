## Asynchronous and Concurrency Handling: Request Scheduling and Event Loop

### Problem Description

**Example Question:**
> “How does JavaScript handle asynchronous operations in the browser? Explain the event loop mechanism, and give an example of how to schedule a large number of concurrent requests without blocking the UI.”

This question evaluates a candidate’s understanding of **JavaScript’s asynchronous concurrency model** — including the event loop, task queues, and practical concurrency control such as managing multiple asynchronous requests efficiently.

---

### Interviewer’s Intent

The interviewer uses this question to assess whether the candidate:

1. Understands **the event loop**, including call stack, macro/microtasks, and rendering cycles.
2. Can apply that understanding to real-world **asynchronous programming** — e.g., request throttling, race condition handling, and async control flow.
3. Knows **different types of asynchronous tasks** in browsers (DOM events, timers, fetch/XHR, requestAnimationFrame, etc.) and their priorities.
4. Has practical experience managing concurrency safely and efficiently in front-end systems.

---

### Structured Answer Framework

#### 1. Event Loop Fundamentals

**1.1 Why the Event Loop Exists**
- JavaScript is single-threaded — one call stack executes all code.
- Long-running synchronous operations (I/O, fetch, timers) would block UI rendering.
- The **event loop** allows asynchronous tasks to run later without blocking the main thread.

**1.2 Call Stack**
- A LIFO (Last-In, First-Out) structure where synchronous functions execute.
- When the stack is full for too long, the UI freezes — so async tasks defer work to queues.

**1.3 Task Queue (Message Queue)**
- Stores callback functions from async operations (e.g., timers, network responses).
- When the call stack is empty, the event loop picks one queued task and executes it.

**1.4 Macro vs Micro Tasks**
- **Macro tasks:** script, setTimeout, setInterval, I/O, UI events.
- **Micro tasks:** Promise.then, MutationObserver, queueMicrotask.
- Execution order:
  - Run one macro task → Run all microtasks → Render → Next macro task.

Example:
```js
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('end');
// Output: start → end → promise → timeout
```

**1.5 Rendering and Special Queues**
- `requestAnimationFrame`: Runs before each frame render — ideal for animations.
- `requestIdleCallback`: Runs during browser idle time — for background/low-priority tasks.

---

#### 2. Practical Application: Asynchronous and Concurrency Management

**2.1 Request Scheduling (Concurrency Control)**
When sending many concurrent requests, it’s crucial to limit how many run simultaneously — for both network and server efficiency.

Example: limit concurrent requests to 5 at a time.
```js
const max = 5;
let running = 0;
const queue = [];

function scheduleRequest(task) {
  if (running < max) {
    running++;
    task().finally(() => {
      running--;
      if (queue.length) scheduleRequest(queue.shift());
    });
  } else {
    queue.push(task);
  }
}
```
- Avoids exceeding browser’s per-domain connection limits (~6 connections in Chrome).
- Common in bulk image loading or large dataset fetching.

**2.2 Preventing Race Conditions**
- Example: search autocomplete — multiple overlapping requests may return out of order.
- Solutions:
  - **Debounce:** Delay requests until user stops typing.
  - **Request Token / Version:** Only process the latest response.
  - **AbortController:** Cancel previous fetch calls.

```js
let controller;
async function search(keyword) {
  if (controller) controller.abort();
  controller = new AbortController();
  try {
    const res = await fetch(`/api/search?q=${keyword}`, { signal: controller.signal });
    return await res.json();
  } catch (e) {
    if (e.name !== 'AbortError') throw e;
  }
}
```

**2.3 Async Error and Timeout Handling**
- `Promise.allSettled()` handles all results (fulfilled/rejected).
- `Promise.race()` helps implement timeouts.

```js
const timeout = new Promise((_, rej) => setTimeout(() => rej('Timeout'), 5000));
await Promise.race([fetchData(), timeout]);
```

**2.4 async/await Concurrency Pitfall**
- `await` pauses execution — sequential by default.
- Use `Promise.all` to achieve parallel execution.
```js
// Sequential (slower)
await fetchA();
await fetchB();

// Concurrent (faster)
await Promise.all([fetchA(), fetchB()]);
```

---

#### 3. Event Loop Execution Example

**Classic ByteDance-style Interview Question:**
```js
console.log('script start');
setTimeout(() => console.log('setTimeout'), 0);
Promise.resolve().then(() => console.log('promise1')).then(() => console.log('promise2'));
console.log('script end');
```
**Output:**
```
script start
script end
promise1
promise2
setTimeout
```
**Explanation:**
- Synchronous code first (script start, script end).
- Promise callbacks (microtasks) run before the next macro task.
- setTimeout runs in the next event loop iteration.

---

#### 4. Web Workers and Parallelism

- The main thread is single-threaded, but **Web Workers** allow true parallel processing.
- Ideal for CPU-heavy tasks (encryption, image processing, data parsing).
- Communication happens via `postMessage` and `onmessage`.
- Workers can’t access the DOM — data is copied via structured cloning.

```js
// main.js
const worker = new Worker('worker.js');
worker.postMessage(data);
worker.onmessage = (e) => console.log('Result:', e.data);
```

---

#### 5. Throttling and Debouncing

**Debounce:** Execute only after the last event in a rapid series.
```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

**Throttle:** Execute once within a fixed time window.
```js
function throttle(fn, interval) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn(...args);
    }
  };
}
```

---

#### 6. Locks and Synchronization

In concurrent async logic, logical locks help prevent overlapping executions.
```js
let isRunning = false;
async function criticalTask() {
  if (isRunning) return;
  isRunning = true;
  try {
    await doWork();
  } finally {
    isRunning = false;
  }
}
```
- Ensures a function doesn’t execute twice simultaneously.
- Useful for preventing duplicate form submissions or overlapping API calls.

---

#### 7. Best Practices

1. **Understand the mechanism, not the memorized order.**  
   - Know *why* Promise.then executes before setTimeout.

2. **Never block the main thread.**  
   - Split long tasks using setTimeout(fn, 0) or requestIdleCallback.

3. **Control concurrency.**  
   - Implement request pools or throttling to limit simultaneous operations.

4. **Always handle errors and timeouts.**  
   - Provide fallbacks and retries for robustness.

5. **Use consistent async style.**  
   - Prefer async/await across the codebase.

6. **Monitor and profile async behavior.**  
   - Use Performance API and DevTools timeline to analyze async task timing.

7. **Stay current with new APIs.**  
   - Explore Promise.allSettled, AbortController, Atomics, SharedArrayBuffer, etc.

---

### Summary

By mastering the event loop and async concurrency model, you can:
- Write responsive, non-blocking JavaScript.
- Efficiently schedule and manage concurrent requests.
- Avoid race conditions, deadlocks, and UI freezes.

**Key takeaway:**
> Understanding the event loop is foundational to advanced front-end engineering — it’s the key to building high-performance, responsive applications and a frequent differentiator in top-tier interviews (e.g., ByteDance, TikTok, Apple).
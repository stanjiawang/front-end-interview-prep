# LinkedIn JavaScript Coding Study Guide

An interview-focused reference covering JavaScript fundamentals, common utility implementations, DOM problems, algorithms, and infinite scrolling.

> The code favors clear, explainable interview solutions. When a simplified solution has production limitations, those limitations are called out explicitly.

## Table of Contents

1. [Fibonacci](#1-fibonacci)
2. [Memoization](#2-memoization)
3. [Promises and the Event Loop](#3-promises-and-the-event-loop)
4. [AbortController](#4-abortcontroller)
5. [Retry with Exponential Backoff](#5-retry-with-exponential-backoff)
6. [Debounce](#6-debounce)
7. [Throttle](#7-throttle)
8. [Timeout Manager](#8-timeout-manager)
9. [Once](#9-once)
10. [Capturing, Bubbling, and Delegation](#10-capturing-bubbling-and-delegation)
11. [DOM Traversal](#11-dom-traversal)
12. [Maximum Subarray](#12-maximum-subarray)
13. [Reverse a Doubly Linked List](#13-reverse-a-doubly-linked-list)
14. [Reverse a String In Place](#14-reverse-a-string-in-place)
15. [Palindrome](#15-palindrome)
16. [String Repeat](#16-string-repeat)
17. [Tuple Parser](#17-tuple-parser)
18. [Flatten a Nested Array](#18-flatten-a-nested-array)
19. [Group By](#19-group-by)
20. [Event Emitter](#20-event-emitter)
21. [LRU Cache](#21-lru-cache)
22. [Infinite Scroll with IntersectionObserver](#22-infinite-scroll-with-intersectionobserver)
23. [Infinite Scroll with a Throttled Scroll Handler](#23-infinite-scroll-with-a-throttled-scroll-handler)
24. [Interview Communication Checklist](#24-interview-communication-checklist)

---

## 1. Fibonacci

### Recursive version

```js
function fibonacci(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return n;
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

The recursive version repeatedly solves the same subproblems.

- Time: `O(2^n)` as a common upper-bound explanation
- Call stack: `O(n)`

### Iterative version

```js
function fibonacci(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return n;
  }

  let previous = 0;
  let current = 1;

  for (let i = 2; i <= n; i += 1) {
    const next = previous + current;
    previous = current;
    current = next;
  }

  return current;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> The recursive solution is simple, but it repeatedly computes the same values. If the requirement is only to calculate Fibonacci efficiently, I would use the iterative version, which runs in linear time and constant space.

---

## 2. Memoization

### Single-argument memoize

```js
function memoize(fn) {
  const cache = new Map();

  return function memoized(arg) {
    if (cache.has(arg)) {
      return cache.get(arg);
    }

    const result = fn.call(this, arg);
    cache.set(arg, result);
    return result;
  };
}
```

Use `cache.has()` rather than testing the cached value’s truthiness. Valid cached results can be `0`, `false`, `""`, `null`, or `undefined`.

### Generalized interview baseline

```js
function memoize(fn) {
  const cache = new Map();

  return function memoized(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

This is a reasonable interview baseline, but `JSON.stringify()` is not a universal key resolver. Limitations include:

- Circular references
- Functions and symbols
- Serialization cost
- Loss of object identity
- Unsupported or ambiguous values

### Identity-based memoize

```js
function createNode() {
  return {
    primitives: new Map(),
    objects: new WeakMap(),
    hasValue: false,
    value: undefined,
  };
}

function isObject(value) {
  return (
    value !== null &&
    (typeof value === "object" || typeof value === "function")
  );
}

function memoize(fn) {
  const root = createNode();

  return function memoized(...args) {
    let node = root;
    const keys = [this, ...args];

    for (const key of keys) {
      const children = isObject(key) ? node.objects : node.primitives;

      if (!children.has(key)) {
        children.set(key, createNode());
      }

      node = children.get(key);
    }

    if (node.hasValue) {
      return node.value;
    }

    node.value = fn.apply(this, args);
    node.hasValue = true;
    return node.value;
  };
}
```

Including `this` in the key prevents different receivers from incorrectly sharing results when the function depends on receiver state.

### Memoized Fibonacci

Recursive calls must go through the memoized function:

```js
const fibonacci = memoize(function calculate(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return BigInt(n);
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(100));
```

- Time: `O(n)`
- Cache: `O(n)`
- Call stack: `O(n)`

### Async memoize

```js
function memoizeAsync(fn, getKey) {
  const cache = new Map();

  return function memoized(...args) {
    const key = getKey(...args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const promise = Promise.resolve(fn.apply(this, args)).catch((error) => {
      cache.delete(key);
      throw error;
    });

    cache.set(key, promise);
    return promise;
  };
}
```

Caching the Promise immediately deduplicates concurrent calls. Removing rejected Promises allows later calls to retry.

Possible cache controls:

- Maximum size
- LRU eviction
- TTL expiration
- Manual invalidation
- Weak object keys

---

## 3. Promises and the Event Loop

JavaScript executes synchronous code on the call stack. Promise reactions and `queueMicrotask()` callbacks enter the microtask queue. Timer callbacks run as tasks. After the current stack is empty, the runtime drains microtasks before starting the next task.

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Output:

```text
A
D
C
B
```

### Implement `Promise.all`

```js
function promiseAll(iterable) {
  return new Promise((resolve, reject) => {
    const values = Array.from(iterable);
    const results = new Array(values.length);

    if (values.length === 0) {
      resolve([]);
      return;
    }

    let completed = 0;

    values.forEach((value, index) => {
      Promise.resolve(value).then((result) => {
        results[index] = result;
        completed += 1;

        if (completed === values.length) {
          resolve(results);
        }
      }, reject);
    });
  });
}
```

Important behavior:

- Accepts an iterable
- Accepts Promises and plain values
- Preserves input order, not completion order
- Resolves empty input to `[]`
- Rejects when any input rejects

**Spoken answer**

> I store each result at its original index so the output preserves input order. I use `Promise.resolve` to normalize plain values and Promises.

---

## 4. AbortController

`AbortController` can cancel the previous request when a newer search begins:

```js
let controller = null;

async function search(query) {
  controller?.abort();
  controller = new AbortController();

  const response = await fetch(
    `/api/search?q=${encodeURIComponent(query)}`,
    { signal: controller.signal },
  );

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  return response.json();
}
```

When handling the error, distinguish cancellation:

```js
try {
  const results = await search(query);
  render(results);
} catch (error) {
  if (error.name !== "AbortError") {
    showError(error);
  }
}
```

---

## 5. Retry with Exponential Backoff

`await` pauses for Promises. `setTimeout()` returns a timer ID, so it must be wrapped in a Promise to become awaitable.

```js
function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function retry(
  operation,
  {
    retries = 3,
    initialDelay = 200,
    factor = 2,
    shouldRetry = () => true,
  } = {},
) {
  let lastError;
  let delay = initialDelay;

  for (let attempt = 0; attempt <= retries; attempt += 1) {
    try {
      return await operation(attempt);
    } catch (error) {
      lastError = error;
      const finalAttempt = attempt === retries;

      if (finalAttempt || !shouldRetry(error)) {
        throw error;
      }

      await wait(delay);
      delay *= factor;
    }
  }

  throw lastError;
}
```

Production follow-ups:

- Add random jitter to prevent synchronized retries
- Retry only transient failures
- Respect cancellation
- Consider a maximum total duration

---

## 6. Debounce

Debounce waits until calls have stopped for a specified period.

```js
function debounce(fn, delay) {
  let timer = null;
  let lastArgs;
  let lastThis;

  function debounced(...args) {
    lastArgs = args;
    lastThis = this;

    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      fn.apply(lastThis, lastArgs);
    }, delay);
  }

  debounced.cancel = function cancel() {
    clearTimeout(timer);
    timer = null;
  };

  debounced.flush = function flush() {
    if (timer === null) {
      return undefined;
    }

    clearTimeout(timer);
    timer = null;
    return fn.apply(lastThis, lastArgs);
  };

  return debounced;
}
```

This version stores the latest context and arguments so `flush()` invokes the actual pending call.

---

## 7. Throttle

Throttle limits how frequently a function executes while calls continue.

### Simple leading-only version

```js
function throttle(fn, delay) {
  let lastRun = 0;

  return function throttled(...args) {
    const now = Date.now();

    if (now - lastRun < delay) {
      return;
    }

    lastRun = now;
    return fn.apply(this, args);
  };
}
```

### Leading and trailing version

```js
function throttle(fn, interval) {
  let lastRun = 0;
  let timer = null;
  let latestArgs;
  let latestThis;

  function invoke() {
    lastRun = Date.now();
    timer = null;
    fn.apply(latestThis, latestArgs);
    latestArgs = undefined;
    latestThis = undefined;
  }

  return function throttled(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastRun);

    latestArgs = args;
    latestThis = this;

    if (remaining <= 0) {
      clearTimeout(timer);
      timer = null;
      invoke();
    } else if (timer === null) {
      timer = setTimeout(invoke, remaining);
    }
  };
}
```

**Debounce vs. throttle**

> Debounce waits for activity to stop. Throttle allows periodic execution while activity continues. I would debounce a search input and throttle a scroll handler.

---

## 8. Timeout Manager

Browsers do not provide a standard “clear every timeout” API. The application must track the timers it creates.

```js
function createTimeoutManager({
  setTimeoutFn = setTimeout,
  clearTimeoutFn = clearTimeout,
} = {}) {
  const active = new Set();

  function set(callback, delay, ...args) {
    let id;

    id = setTimeoutFn(() => {
      active.delete(id);
      callback(...args);
    }, delay);

    active.add(id);
    return id;
  }

  function clear(id) {
    active.delete(id);
    clearTimeoutFn(id);
  }

  function clearAll() {
    for (const id of active) {
      clearTimeoutFn(id);
    }

    active.clear();
  }

  return {
    setTimeout: set,
    clearTimeout: clear,
    clearAllTimeouts: clearAll,
    getActiveCount: () => active.size,
  };
}

const timers = createTimeoutManager();

timers.setTimeout(() => console.log("A"), 1000);
timers.setTimeout(() => console.log("B"), 2000);
timers.clearAllTimeouts();
```

---

## 9. Once

```js
function once(fn) {
  let called = false;
  let result;

  return function onceWrapper(...args) {
    if (!called) {
      result = fn.apply(this, args);
      called = true;
    }

    return result;
  };
}
```

This version counts only a successful return as the first call. If `fn` throws, a later call may try again.

---

## 10. Capturing, Bubbling, and Delegation

An event normally travels:

1. From the document toward the target during capture
2. At the target
3. Back through ancestors during bubbling

`event.target` is where the event originated. `event.currentTarget` is the element whose listener is currently running.

### Event delegation

```js
const list = document.querySelector("#list");

function handleClick(event) {
  const button = event.target.closest("button[data-action]");

  if (!button || !list.contains(button)) {
    return;
  }

  const item = button.closest("[data-item-id]");

  if (!item) {
    return;
  }

  const action = button.dataset.action;
  const itemId = item.dataset.itemId;

  handleAction(action, itemId);
}

list.addEventListener("click", handleClick);

// Cleanup:
list.removeEventListener("click", handleClick);
```

**Spoken answer**

> Event delegation attaches one listener to a stable ancestor. I use `closest` because the user may click a nested icon or span inside the button. The containment check ensures the match belongs to this component.

---

## 11. DOM Traversal

### Implement `getElementsByClassName`

This version searches descendants only and requires all supplied classes.

```js
function getElementsByClassName(root, classNames) {
  if (!(root instanceof Element) && !(root instanceof Document)) {
    throw new TypeError("root must be an Element or Document");
  }

  const required = classNames.trim().split(/\s+/).filter(Boolean);

  if (required.length === 0) {
    return [];
  }

  const results = [];
  const stack = [...root.children].reverse();

  while (stack.length > 0) {
    const element = stack.pop();
    const matches = required.every((name) =>
      element.classList.contains(name),
    );

    if (matches) {
      results.push(element);
    }

    for (let i = element.children.length - 1; i >= 0; i -= 1) {
      stack.push(element.children[i]);
    }
  }

  return results;
}
```

Let `n` be the number of descendants and `c` the number of required classes:

- Time: `O(n × c)`
- Auxiliary traversal space: up to `O(n)`

### Is one node a descendant of another?

Native version:

```js
function isDescendant(parent, child) {
  return parent !== child && parent.contains(child);
}
```

Manual version:

```js
function isDescendant(parent, child) {
  let current = child.parentNode;

  while (current !== null) {
    if (current === parent) {
      return true;
    }

    current = current.parentNode;
  }

  return false;
}
```

Walking upward follows one parent chain; searching downward may inspect the entire subtree.

---

## 12. Maximum Subarray

```js
function maximumSubarray(values) {
  if (!Array.isArray(values)) {
    throw new TypeError("values must be an array");
  }

  if (values.length === 0) {
    return null;
  }

  let currentSum = values[0];
  let currentStart = 0;
  let bestSum = values[0];
  let bestStart = 0;
  let bestEnd = 0;

  for (let i = 1; i < values.length; i += 1) {
    const value = values[i];

    if (value > currentSum + value) {
      currentSum = value;
      currentStart = i;
    } else {
      currentSum += value;
    }

    if (currentSum > bestSum) {
      bestSum = currentSum;
      bestStart = currentStart;
      bestEnd = i;
    }
  }

  return {
    sum: bestSum,
    start: bestStart,
    end: bestEnd,
  };
}
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> At each index, I decide whether to extend the previous subarray or start a new one. I track the temporary start index and copy it when I find a new global maximum.

Initializing from the first element correctly handles all-negative arrays.

---

## 13. Reverse a Doubly Linked List

```js
class ListNode {
  constructor(value) {
    this.value = value;
    this.previous = null;
    this.next = null;
  }
}

function reverseDoublyLinkedList(head) {
  let current = head;
  let newHead = null;

  while (current !== null) {
    const originalNext = current.next;

    current.next = current.previous;
    current.previous = originalNext;

    newHead = current;
    current = originalNext;
  }

  return newHead;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

> I save the original next pointer before swapping the links. After the swap, `current.next` points backward, so I need the saved pointer to continue along the original forward direction.

---

## 14. Reverse a String In Place

Assume the input is a mutable character array:

```js
function reverseString(chars) {
  let left = 0;
  let right = chars.length - 1;

  while (left < right) {
    [chars[left], chars[right]] = [chars[right], chars[left]];
    left += 1;
    right -= 1;
  }

  return chars;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

---

## 15. Palindrome

```js
function isPalindrome(value) {
  let left = 0;
  let right = value.length - 1;

  while (left < right) {
    if (value[left] !== value[right]) {
      return false;
    }

    left += 1;
    right -= 1;
  }

  return true;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

Clarify case sensitivity, punctuation, whitespace, and Unicode normalization.

---

## 16. String Repeat

### Straightforward version

```js
function repeatString(value, times) {
  const count = Number(times);

  if (!Number.isInteger(count) || count < 0) {
    throw new RangeError("times must be a non-negative integer");
  }

  let result = "";

  for (let i = 0; i < count; i += 1) {
    result += value;
  }

  return result;
}
```

### Doubling version

```js
function repeatString(value, times) {
  let count = Number(times);

  if (!Number.isInteger(count) || count < 0) {
    throw new RangeError("times must be a non-negative integer");
  }

  let result = "";
  let chunk = String(value);

  while (count > 0) {
    if (count % 2 === 1) {
      result += chunk;
    }

    count = Math.floor(count / 2);

    if (count > 0) {
      chunk += chunk;
    }
  }

  return result;
}
```

The doubling approach uses `O(log times)` high-level iterations, but total work cannot be less than the output size.

---

## 17. Tuple Parser

Given:

```js
const result = tuple("(1, 2, 3), (4, 5, 6), (7, 8, 9)");
result.multiply(2); // 2 * 5 * 8 = 80
```

Assume `multiply()` uses one-based positions.

```js
function tuple(input) {
  if (typeof input !== "string") {
    throw new TypeError("input must be a string");
  }

  const tuples = [];
  const pattern = /\(([^()]*)\)/g;
  let match;

  while ((match = pattern.exec(input)) !== null) {
    const values = match[1].split(",").map((token) => {
      const value = Number(token.trim());

      if (!Number.isFinite(value)) {
        throw new SyntaxError(`Invalid value: ${token}`);
      }

      return value;
    });

    tuples.push(values);
  }

  if (tuples.length === 0) {
    throw new SyntaxError("No tuples found");
  }

  return {
    values: tuples.map((values) => [...values]),

    multiply(position) {
      if (!Number.isInteger(position) || position < 1) {
        throw new RangeError("position must be a positive integer");
      }

      const index = position - 1;

      return tuples.reduce((product, values) => {
        if (index >= values.length) {
          throw new RangeError(`Tuple has no position ${position}`);
        }

        return product * values[index];
      }, 1);
    },
  };
}
```

- Parsing: linear in the input length for this simplified grammar
- `multiply`: `O(t)`, where `t` is the number of tuples

Nested tuples and quoted commas require a tokenizer or state-machine parser.

---

## 18. Flatten a Nested Array

```js
function flatten(values) {
  const result = [];
  const stack = [...values].reverse();

  while (stack.length > 0) {
    const value = stack.pop();

    if (Array.isArray(value)) {
      for (let i = value.length - 1; i >= 0; i -= 1) {
        stack.push(value[i]);
      }
    } else {
      result.push(value);
    }
  }

  return result;
}
```

The reverse initialization and reverse child insertion preserve left-to-right order.

---

## 19. Group By

```js
function groupBy(values, getKey) {
  const groups = new Map();

  for (const value of values) {
    const key = getKey(value);

    if (!groups.has(key)) {
      groups.set(key, []);
    }

    groups.get(key).push(value);
  }

  return groups;
}
```

- Time: `O(n)` expected
- Space: `O(n)`

---

## 20. Event Emitter

### Array-based interview version

```js
class EventEmitter {
  constructor() {
    this.events = new Map();
  }

  on(name, fn) {
    if (typeof fn !== "function") {
      throw new TypeError("fn must be a function");
    }

    const list = this.events.get(name) ?? [];
    list.push(fn);
    this.events.set(name, list);

    return () => this.off(name, fn);
  }

  off(name, fn) {
    const list = this.events.get(name);

    if (!list) {
      return false;
    }

    const index = list.indexOf(fn);

    if (index === -1) {
      return false;
    }

    list.splice(index, 1);

    if (list.length === 0) {
      this.events.delete(name);
    }

    return true;
  }

  once(name, fn) {
    const wrapper = (...args) => {
      this.off(name, wrapper);
      fn.apply(this, args);
    };

    return this.on(name, wrapper);
  }

  emit(name, ...args) {
    const list = this.events.get(name);

    if (!list) {
      return false;
    }

    for (const fn of [...list]) {
      fn.apply(this, args);
    }

    return true;
  }

  listenerCount(name) {
    return this.events.get(name)?.length ?? 0;
  }

  removeAllListeners(name) {
    if (name === undefined) {
      this.events.clear();
    } else {
      this.events.delete(name);
    }
  }
}
```

Why use a snapshot in `emit()`?

> A listener may subscribe or unsubscribe during emission. Iterating over a copy gives the current emission stable behavior.

Why remove a once-listener before invoking it?

> If the listener synchronously emits the same event, removing the wrapper first guarantees true once-only behavior.

Array semantics:

- Allows duplicate listener registrations
- `on`: `O(1)`
- `off`: `O(n)`
- `emit`: `O(n)`

With a `Set`, duplicate registrations are ignored and deletion is expected `O(1)`.

---

## 21. LRU Cache

Invariant:

```text
Beginning of Map → least recently used
End of Map       → most recently used
```

```js
class LRUCache {
  constructor(capacity) {
    if (!Number.isInteger(capacity) || capacity <= 0) {
      throw new RangeError("capacity must be a positive integer");
    }

    this.capacity = capacity;
    this.cache = new Map();
  }

  get size() {
    return this.cache.size;
  }

  has(key) {
    return this.cache.has(key);
  }

  get(key) {
    if (!this.cache.has(key)) {
      return undefined;
    }

    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    this.cache.set(key, value);

    if (this.cache.size > this.capacity) {
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }

    return this;
  }

  delete(key) {
    return this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }
}
```

`Map` iterates in insertion order. A successful `get()` deletes and reinserts the entry, moving it to the end. `map.keys()` returns an iterator, and its first `next()` result contains the oldest key.

- Expected `get`: `O(1)`
- Expected `set`: `O(1)`
- Space: `O(capacity)`

---

## 22. Infinite Scroll with IntersectionObserver

Use a CoderPad HTML/CSS/JavaScript environment.

### `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Infinite Scroll</title>
    <link rel="stylesheet" href="./style.css" />
    <script src="./script.js" defer></script>
  </head>
  <body>
    <main>
      <ul id="feed" aria-label="Feed"></ul>
      <p id="status" aria-live="polite"></p>
      <button id="retry" type="button" hidden>Retry</button>
      <div id="sentinel" aria-hidden="true"></div>
    </main>
  </body>
</html>
```

### `style.css`

```css
* {
  box-sizing: border-box;
}

#feed {
  margin: 0;
  padding: 0;
  list-style: none;
}

#feed li {
  min-height: 100px;
  margin-bottom: 10px;
  padding: 16px;
  border: 1px solid #ccc;
}

#status {
  min-height: 24px;
}

#sentinel {
  height: 1px;
}
```

### `script.js` with a runnable mock API

```js
const data = Array.from({ length: 35 }, (_, i) => ({
  id: i + 1,
  title: `Item ${i + 1}`,
}));

function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function fetchPage(page) {
  await wait(400);

  const size = 10;
  const start = page * size;
  const items = data.slice(start, start + size);

  return {
    items,
    hasMore: start + size < data.length,
  };
}

const feed = document.querySelector("#feed");
const status = document.querySelector("#status");
const retry = document.querySelector("#retry");
const sentinel = document.querySelector("#sentinel");

let page = 0;
let loading = false;
let done = false;

function render(items) {
  const fragment = document.createDocumentFragment();

  for (const item of items) {
    const li = document.createElement("li");
    li.textContent = item.title;
    fragment.append(li);
  }

  feed.append(fragment);
}

async function loadMore() {
  if (loading || done) {
    return;
  }

  loading = true;
  retry.hidden = true;
  status.textContent = "Loading...";

  try {
    const { items, hasMore } = await fetchPage(page);

    render(items);
    page += 1;
    done = !hasMore;
    status.textContent = done ? "No more items." : "";

    if (done) {
      observer.disconnect();
    }
  } catch (error) {
    console.error(error);
    status.textContent = "Failed to load.";
    retry.hidden = false;
  } finally {
    loading = false;
  }
}

const observer = new IntersectionObserver(
  ([entry]) => {
    // We observe one target, so use the first reported entry.
    if (entry.isIntersecting) {
      loadMore();
    }
  },
  {
    root: null, // Use the browser viewport.
    rootMargin: "0px 0px 300px 0px", // Preload 300px early.
    threshold: 0, // Trigger when any part intersects.
  },
);

retry.addEventListener("click", loadMore);
observer.observe(sentinel);
loadMore();
```

The sentinel is not fixed to the viewport. It is a normal element after the feed. Appending items increases the feed’s height and naturally pushes the sentinel downward.

### Real HTTP version of `fetchPage`

Use this only when the environment supplies an API:

```js
async function fetchPage(page) {
  const url = new URL("/api/feed", window.location.origin);
  url.searchParams.set("page", String(page));
  url.searchParams.set("limit", "10");

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  const result = await response.json();

  return {
    items: result.items,
    hasMore: result.hasMore,
  };
}
```

The caller remains:

```js
const { items, hasMore } = await fetchPage(page);
```

`fetchPage()` encapsulates the HTTP response check and JSON parsing, returning normalized application data.

---

## 23. Infinite Scroll with a Throttled Scroll Handler

Replace the observer setup with:

```js
function throttle(fn, delay) {
  let lastRun = 0;

  return function throttled(...args) {
    const now = Date.now();

    if (now - lastRun < delay) {
      return;
    }

    lastRun = now;
    return fn.apply(this, args);
  };
}

const handleScroll = throttle(() => {
  // Full height of the page, including off-screen content.
  const pageHeight = document.documentElement.scrollHeight;

  // Position of the viewport's bottom edge within the page.
  const viewportBottom = window.scrollY + window.innerHeight;

  // Remaining pixels between the viewport and the page bottom.
  const distance = pageHeight - viewportBottom;

  if (distance < 300) {
    loadMore();
  }
}, 200);

window.addEventListener("scroll", handleScroll, {
  // The handler will not cancel scrolling with preventDefault().
  passive: true,
});

loadMore();
```

Cleanup:

```js
window.removeEventListener("scroll", handleScroll);
```

Throttle and the `loading` guard solve different problems:

- Throttle limits layout calculations caused by frequent scroll events.
- `loading` prevents another network request while one is already pending.

**Spoken answer**

> The page height is the full document height. `scrollY + innerHeight` gives the current viewport’s bottom position. Their difference is the remaining distance to the page bottom. I throttle this calculation because scroll events fire frequently, and I still keep the loading guard because a network request can take longer than the throttle interval.

---

## 24. Interview Communication Checklist

### Before coding

> Let me restate the requirements to make sure I understand the problem correctly.

> Should invalid input throw, return an empty value, or can I assume valid input?

> Should object arguments be compared by identity or by value?

> I’ll start with a simple correct implementation, test it, and then discuss improvements.

### While coding

> I’m using a `Map` because I need efficient lookup and must correctly preserve falsy values.

> I’m setting the loading flag before the first `await` so another call cannot pass the guard.

> I’m iterating over a snapshot because listeners may unsubscribe during emission.

### After coding

> Let me walk through normal input, empty input, invalid input, duplicates, and boundary cases.

> Let `n` be the number of input elements. The algorithm visits each element once, so its time complexity is `O(n)`.

> This is the simplest working version. In production, I would additionally consider cancellation, cache limits, cleanup, accessibility, and observability.



# Table of Contents
- [1. debounce（防抖 / Debounce）](#1-debounce防抖--debounce)
- [2. throttle（节流 / Throttle）](#2-throttle节流--throttle)
- [3. once（只执行一次 / Once）](#3-once只执行一次--once)
- [4. memoize（记忆化 / Memoize）](#4-memoize记忆化--memoize)
- [5. curry（柯里化 / Curry）](#5-curry柯里化--curry)
- [6. compose / pipe（函数组合 / Function Composition）](#6-compose--pipe函数组合--function-composition)
- [7. deepClone（深拷贝 / Deep Clone）+ WeakMap/WeakSet](#7-deepclone深拷贝--deep-clone-weakmapweakset)
- [8. shallowClone（浅拷贝 / Shallow Clone）](#8-shallowclone浅拷贝--shallow-clone)
- [9. deepEqual（深度比较 / Deep Equal）](#9-deepequal深度比较--deep-equal)
- [10. instanceof 实现（Prototype Chain Check）](#10-instanceof-实现prototype-chain-check)
- [11. Object.create 实现](#11-objectcreate-实现)
- [12. Promise.all 实现](#12-promiseall-实现)
- [13. Promise.race 实现](#13-promiserace-实现)
- [14. sleep（延迟 / Delay）](#14-sleep延迟--delay)
- [15. EventEmitter（发布-订阅 / Pub-Sub）](#15-eventemitter发布-订阅--pub-sub)
- [16. flatten（数组扁平化 / Array Flatten）](#16-flatten数组扁平化--array-flatten)
- [17. unique（数组去重 / Array Unique）](#17-unique数组去重--array-unique)
- [18. LRUCache（最近最少使用缓存）](#18-lrucache最近最少使用缓存)
- [19. new 操作符实现（Simulate `new`）](#19-new-操作符实现simulate-new)
- [20. call / apply / bind 实现](#20-call--apply--bind-实现)
- [附录 A：重点术语中英双语速记 / Bilingual Cheat Sheet](#附录-a重点术语中英双语速记--bilingual-cheat-sheet)
- [附录 B：面试口述模板（英文） / Interview Spiels](#附录-b面试口述模板英文--interview-spiels)

---

## 1. debounce（防抖 / Debounce）
**定义（Definition）**：在连续触发的事件中，只有当触发停止达到一定时间间隔后才执行目标函数。  
**典型场景（Use cases）**：输入框搜索、表单校验、窗口 `resize`、按钮防连击。  
**关键点（Key points）**：`clearTimeout + setTimeout`、保持调用时 `this`、仅在结尾触发（trailing）。

### Standard — JavaScript (full comments)
```js
/**
 * Debounce (trailing) — execute after no calls occur for `delay` ms.
 * Preserves dynamic `this` from the call site (class methods / event handlers).
 */
function debounce(fn, delay = 300) {
  let timerId;

  // Return the debounced wrapper so callers can attach as an event handler
  return function debouncedFunction(...args) {
    const dynamicThis = this; // capture dynamic `this`

    // Cancel any previously scheduled execution
    clearTimeout(timerId);

    // Schedule a fresh execution for the trailing edge
    timerId = setTimeout(() => {
      fn.apply(dynamicThis, args);
    }, delay);
  };
}
```

### React-friendly — JavaScript (hook-safe creation)
```js
/**
 * React-friendly usage: create the debounced function once (e.g., with useCallback).
 * NOTE: In React function components there is no meaningful `this` — still keep .apply for generality.
 */
import { useCallback } from "react";

function useDebouncedHandler(handler, delay = 300) {
  // Create once; ensure handler is stable or intentionally captured
  // eslint-disable-next-line react-hooks/exhaustive-deps
  return useCallback(debounce(handler, delay), []);
}
```

### Standard — TypeScript
```ts
/**
 * Debounce (trailing) — TypeScript version.
 */
export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay = 300
) {
  let timerId: ReturnType<typeof setTimeout> | null = null;

  return function debounced(this: ThisParameterType<T>, ...args: Parameters<T>) {
    const dynamicThis = this;
    if (timerId) clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(dynamicThis, args);
    }, delay);
  };
}
```

### React-friendly — TypeScript
```ts
import { useCallback } from "react";
import { debounce } from "./debounce";

export function useDebouncedHandler<T extends (...args: any[]) => any>(
  handler: T,
  delay = 300
) {
  // Create once to avoid resetting the internal timer on every render
  // eslint-disable-next-line react-hooks/exhaustive-deps
  return useCallback(debounce(handler, delay), []);
}
```

#### Usage Examples
```jsx
// 1) Search box: trigger API only after user stops typing for 500ms
function SearchBox() {
  const onSearch = (value) => fetch(`/api/search?q=${encodeURIComponent(value)}`);

  const handleChange = useDebouncedHandler((e) => {
    onSearch(e.target.value);
  }, 500);

  return <input onChange={handleChange} placeholder="Type to search..." />;
}
```

#### Interview (English)
- “I clear the previous timer and schedule a new one, so the function fires only on the trailing edge after the user stops triggering.”  
- “I keep `this` via `apply`, which is crucial for class methods / event handlers.”  
- “In React, I memoize the debounced function (e.g., `useCallback`) to avoid re-creating it each render.”

---

## 2. throttle（节流 / Throttle）
**定义**：在固定时间窗口内最多执行一次。  
**场景**：滚动、拖拽、`resize` 等高频事件。

### Standard — JavaScript (leading-only)
```js
/**
 * Throttle (leading-only): run at most once every `delay` ms.
 */
function throttle(fn, delay = 300) {
  let lastTime = 0;

  return function throttled(...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

### React-friendly — JavaScript (leading+trailing)
```js
/**
 * Leading + trailing throttle useful for UI to not miss the last call.
 */
function throttleLeadingTrailing(fn, delay = 300) {
  let lastTime = 0;
  let timerId = null;

  return function throttled(...args) {
    const context = this;
    const now = Date.now();
    const remaining = delay - (now - lastTime);

    if (remaining <= 0) {
      if (timerId) {
        clearTimeout(timerId);
        timerId = null;
      }
      lastTime = now;
      fn.apply(context, args);
    } else if (!timerId) {
      timerId = setTimeout(() => {
        lastTime = Date.now();
        timerId = null;
        fn.apply(context, args);
      }, remaining);
    }
  };
}
```

### Standard — TypeScript
```ts
export function throttle<T extends (...args: any[]) => any>(fn: T, delay = 300) {
  let lastTime = 0;
  return function throttled(this: ThisParameterType<T>, ...args: Parameters<T>) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

### React-friendly — TypeScript
```ts
export function throttleLeadingTrailing<T extends (...args: any[]) => any>(fn: T, delay = 300) {
  let lastTime = 0;
  let timerId: ReturnType<typeof setTimeout> | null = null;
  return function throttled(this: ThisParameterType<T>, ...args: Parameters<T>) {
    const context = this;
    const now = Date.now();
    const remaining = delay - (now - lastTime);
    if (remaining <= 0) {
      if (timerId) {
        clearTimeout(timerId);
        timerId = null;
      }
      lastTime = now;
      fn.apply(context, args);
    } else if (!timerId) {
      timerId = setTimeout(() => {
        lastTime = Date.now();
        timerId = null;
        fn.apply(context, args);
      }, remaining);
    }
  };
}
```

#### Usage Examples
```jsx
// Window scroll handler: limit expensive work
function ScrollStats() {
  const onScroll = useCallback(throttleLeadingTrailing(() => {
    // e.g., compute viewport metrics
    console.log(window.scrollY);
  }, 200), []);

  useEffect(() => {
    window.addEventListener("scroll", onScroll);
    return () => window.removeEventListener("scroll", onScroll);
  }, [onScroll]);

  return null;
}
```

#### Interview (English)
- “I track the last execution time and ensure we run at most once per interval.”  
- “For UX, a trailing call ensures we don’t miss the last meaningful event.”  
- “I memoize the throttled handler in React to avoid timer re-initialization.”

---

## 3. once（只执行一次 / Once）
**定义**：只在第一次调用时执行，之后返回缓存结果。

### Standard — JS
```js
function once(fn) {
  let called = false;
  let result;
  return function onceWrapper(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

### React-friendly — JS
```js
/**
 * Use once to ensure a setup routine runs a single time across rapid events.
 */
const initializeOnce = once(() => {
  // heavy init here
});
// later in events
// initializeOnce();
```

### Standard — TS
```ts
export function once<T extends (...args: any[]) => any>(fn: T) {
  let called = false;
  let result: ReturnType<T>;
  return function onceWrapper(this: ThisParameterType<T>, ...args: Parameters<T>): ReturnType<T> {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

#### Usage Examples
```js
const initSDK = once(() => console.log("SDK initialized"));
initSDK(); // runs
initSDK(); // no-op, returns cached result
```

#### Interview (English)
- “A closure flag plus cached result ensures a function only runs once.”  
- “This is useful for idempotent initialization such as bootstrapping SDKs.”

---

## 4. memoize（记忆化 / Memoize）
**定义**：缓存输入到输出的映射以避免重复计算。  
**注意**：`JSON.stringify` 适用于简单参数；复杂键建议 `resolver`。

### Standard — JS (with resolver)
```js
function memoize(fn, resolver) {
  const cache = new Map();
  return function memoized(...args) {
    const key = resolver ? resolver.apply(this, args) : JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

### React-friendly — JS
```js
/**
 * For pure functions used in render pipelines, memoize can avoid recomputation.
 * Be mindful of memory growth; expose a .clear if needed.
 */
const expensive = (n) => (n <= 1 ? n : expensive(n-1) + expensive(n-2));
const fastFib = memoize(expensive);
```

### Standard — TS
```ts
export function memoize<T extends (...args: any[]) => any, K = string>(
  fn: T,
  resolver?: (...args: Parameters<T>) => K
) {
  const cache = new Map<K | string, ReturnType<T>>();
  return function memoized(
    this: ThisParameterType<T>,
    ...args: Parameters<T>
  ): ReturnType<T> {
    const key = (resolver ? resolver.apply(this, args) : JSON.stringify(args)) as K | string;
    if (cache.has(key)) return cache.get(key)!;
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

#### Usage Examples
```js
const add = (a, b) => a + b;
const addMemo = memoize(add, (a, b) => `${a}-${b}`);
addMemo(1, 2); // computes
addMemo(1, 2); // cached
```

#### Interview (English)
- “I key the arguments via a resolver or JSON; then return from cache if present.”  
- “It’s powerful for pure, deterministic functions; watch out for cache size.”

---

## 5. curry（柯里化 / Curry）
**定义**：将多参函数转化为可分批传参的链式调用。

### Standard — JS
```js
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...nextArgs) => curried.apply(this, args.concat(nextArgs));
  };
}
```

### Standard — TS
```ts
export function curry<T extends (...args: any[]) => any>(fn: T) {
  function curried(this: ThisParameterType<T>, ...args: any[]): any {
    return args.length >= fn.length
      ? fn.apply(this, args as Parameters<T>)
      : (...next: any[]) => curried.apply(this, [...args, ...next]);
  }
  return curried as any;
}
```

#### Usage Examples
```js
function sum3(a, b, c) { return a + b + c; }
const curriedSum3 = curry(sum3);
curriedSum3(1)(2)(3);  // 6
curriedSum3(1, 2)(3);  // 6
curriedSum3(1)(2, 3);  // 6
```

#### Interview (English)
- “I collect arguments until meeting `fn.length`, then execute the original function.”  
- “This enables parameter reuse and functional composition.”

---

## 6. compose / pipe（函数组合 / Function Composition）
**compose**：右→左；**pipe**：左→右。

### Standard — JS
```js
function compose(...fns) {
  return function composed(initialValue) {
    return fns.reduceRight((acc, fn) => fn(acc), initialValue);
  };
}
function pipe(...fns) {
  return function piped(initialValue) {
    return fns.reduce((acc, fn) => fn(acc), initialValue);
  };
}
```

### Standard — TS
```ts
export function compose<T>(...fns: Array<(x: T) => T>) {
  return (initial: T) => fns.reduceRight((acc, fn) => fn(acc), initial);
}
export function pipe<T>(...fns: Array<(x: T) => T>) {
  return (initial: T) => fns.reduce((acc, fn) => fn(acc), initial);
}
```

#### Usage Examples
```js
const add2 = (x) => x + 2;
const mul3 = (x) => x * 3;
const square = (x) => x * x;

const composed = compose(square, mul3, add2); // square(mul3(add2(x)))
composed(2); // 144
```

#### Interview (English)
- “`compose` executes right-to-left; each function receives the previous output.”  
- “`pipe` is the left-to-right counterpart, often easier to read for pipelines.”

---

## 7. deepClone（深拷贝 / Deep Clone） + WeakMap/WeakSet
**目标**：递归复制并处理特殊类型与循环引用。  
**为何 WeakMap**：键为对象且弱引用（不会阻止 GC），适合记录“已拷贝映射”。

### Standard — JS
```js
function deepClone(value, hash = new WeakMap()) {
  if (value === null || typeof value !== "object") return value;
  if (hash.has(value)) return hash.get(value);

  if (value instanceof Date) return new Date(value);
  if (value instanceof RegExp) return new RegExp(value);

  if (value instanceof Map) {
    const out = new Map();
    hash.set(value, out);
    value.forEach((v, k) => out.set(deepClone(k, hash), deepClone(v, hash)));
    return out;
  }
  if (value instanceof Set) {
    const out = new Set();
    hash.set(value, out);
    value.forEach(v => out.add(deepClone(v, hash)));
    return out;
  }

  const out = Array.isArray(value) ? [] : {};
  hash.set(value, out);

  for (const key in value) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      out[key] = deepClone(value[key], hash);
    }
  }
  return out;
}
```

### Standard — TS
```ts
export function deepClone<T>(value: T, hash = new WeakMap()): T {
  if (value === null || typeof value !== "object") return value;
  if (hash.has(value as any)) return hash.get(value as any);

  if (value instanceof Date) return new Date(value) as any;
  if (value instanceof RegExp) return new RegExp(value) as any;

  if (value instanceof Map) {
    const out = new Map();
    hash.set(value, out);
    value.forEach((v, k) => out.set(deepClone(k, hash), deepClone(v, hash)));
    return out as any;
  }
  if (value instanceof Set) {
    const out = new Set();
    hash.set(value, out);
    value.forEach(v => out.add(deepClone(v, hash)));
    return out as any;
  }

  const out: any = Array.isArray(value) ? [] : {};
  hash.set(value as any, out);
  for (const key in value as any) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      out[key] = deepClone((value as any)[key], hash);
    }
  }
  return out;
}
```

#### Usage Examples
```js
const a = { x: 1, y: { z: [1, 2] } };
a.self = a;
const b = deepClone(a);
console.log(b !== a, b.y !== a.y, b.self === b); // true true true
```

#### Interview (English)
- “I use a `WeakMap` to record already-cloned objects to break cycles and avoid leaks.”  
- “I branch for Date/RegExp/Map/Set and recursively copy arrays/objects.”

---

## 8. shallowClone（浅拷贝 / Shallow Clone）
### Standard — JS
```js
function shallowClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  const out = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      out[key] = obj[key];
    }
  }
  return out;
}
```

### Standard — TS
```ts
export function shallowClone<T extends object>(obj: T): T {
  if (obj === null || typeof obj !== "object") return obj;
  const out: any = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      out[key] = (obj as any)[key];
    }
  }
  return out;
}
```

#### Usage Examples
```js
const o = { a: 1, b: { c: 2 } };
const s = shallowClone(o);
console.log(s.b === o.b); // true (shared reference)
```

#### Interview (English)
- “Only the first level is copied; nested references are shared.”

---

## 9. deepEqual（深度比较 / Deep Equal）
### Standard — JS
```js
function deepEqual(a, b, visited = new WeakMap()) {
  if (a === b) return true;
  if (a === null || b === null || typeof a !== "object" || typeof b !== "object") return false;

  if (visited.has(a)) return visited.get(a) === b;
  visited.set(a, b);

  if (a.constructor !== b.constructor) return false;
  if (a instanceof Date) return a.getTime() === b.getTime();
  if (a instanceof RegExp) return a.toString() === b.toString();

  if (a instanceof Map) {
    if (a.size !== b.size) return false;
    for (const [k, v] of a) {
      if (!b.has(k) || !deepEqual(v, b.get(k), visited)) return false;
    }
    return true;
  }
  if (a instanceof Set) {
    if (a.size !== b.size) return false;
    for (const v of a) if (!b.has(v)) return false;
    return true;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  for (const k of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, k)) return false;
    if (!deepEqual(a[k], b[k], visited)) return false;
  }
  return true;
}
```

### Standard — TS
```ts
export function deepEqual(a: any, b: any, visited = new WeakMap()): boolean {
  if (a === b) return true;
  if (a === null || b === null || typeof a !== "object" || typeof b !== "object") return false;

  if (visited.has(a)) return visited.get(a) === b;
  visited.set(a, b);

  if (a.constructor !== b.constructor) return false;
  if (a instanceof Date) return a.getTime() === (b as Date).getTime();
  if (a instanceof RegExp) return a.toString() === (b as RegExp).toString();

  if (a instanceof Map) {
    if (a.size !== (b as Map<any, any>).size) return false;
    for (const [k, v] of a) {
      if (!(b as Map<any, any>).has(k) || !deepEqual(v, (b as Map<any, any>).get(k), visited)) return false;
    }
    return true;
  }
  if (a instanceof Set) {
    if (a.size !== (b as Set<any>).size) return false;
    for (const v of a) if (!(b as Set<any>).has(v)) return false;
    return true;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;
  for (const k of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, k)) return false;
    if (!deepEqual(a[k], b[k], visited)) return false;
  }
  return true;
}
```

#### Usage Examples
```js
deepEqual({a:1}, {a:1}); // true
deepEqual([1,[2]], [1,[2]]); // true
deepEqual(new Date(0), new Date(0)); // true
```

#### Interview (English)
- “I use `WeakMap` for cycle handling and branch on special types like Date/RegExp/Map/Set.”

---

## 10. instanceof 实现（Prototype Chain Check）
### Standard — JS
```js
function myInstanceof(obj, Ctor) {
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) return false;
  if (typeof Ctor !== "function") throw new TypeError("Right-hand side must be a function");
  let proto = Object.getPrototypeOf(obj);
  const target = Ctor.prototype;
  while (proto) {
    if (proto === target) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

### Standard — TS
```ts
export function myInstanceof(obj: any, Ctor: Function): boolean {
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) return false;
  if (typeof Ctor !== "function") throw new TypeError("Right-hand side must be a function");
  let proto = Object.getPrototypeOf(obj);
  const target = Ctor.prototype;
  while (proto) {
    if (proto === target) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

#### Usage Examples
```js
function Person() {}
const p = new Person();
myInstanceof(p, Person); // true
myInstanceof(p, Object); // true
myInstanceof(p, Array);  // false
```

#### Interview (English)
- “Walk up the prototype chain and check equality with `Ctor.prototype`.”

---

## 11. Object.create 实现
### Standard — JS
```js
function myCreate(proto) {
  if (proto !== null && typeof proto !== "object") {
    throw new TypeError("Object prototype may only be an Object or null");
  }
  function F() {}
  F.prototype = proto;
  return new F();
}
```

### Standard — TS
```ts
export function myCreate<T extends object>(proto: T | null): T {
  if (proto !== null && typeof proto !== "object") {
    throw new TypeError("Object prototype may only be an Object or null");
  }
  function F() {}
  (F as any).prototype = proto;
  return new (F as any)();
}
```

#### Usage Examples
```js
const base = { greet(){ console.log("hi"); } };
const o = myCreate(base);
o.greet(); // "hi"
```

#### Interview (English)
- “A temporary constructor’s prototype is set to `proto`, then `new` yields an object whose `[[Prototype]]` is `proto`.”

---

## 12. Promise.all 实现
### Standard — JS
```js
function myAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) return reject(new TypeError("Argument must be an array"));
    const results = [];
    let completed = 0;
    if (promises.length === 0) return resolve(results);
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        (val) => {
          results[i] = val;
          if (++completed === promises.length) resolve(results);
        },
        (err) => reject(err)
      );
    });
  });
}
```

### Standard — TS
```ts
export function myAll<T>(promises: Array<Promise<T> | T>): Promise<T[]> {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) return reject(new TypeError("Argument must be an array"));
    const results: T[] = [];
    let completed = 0;
    if (promises.length === 0) return resolve(results);
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        (val) => {
          results[i] = val as T;
          if (++completed === promises.length) resolve(results);
        },
        (err) => reject(err)
      );
    });
  });
}
```

#### Usage Examples
```js
myAll([Promise.resolve(1), 2, Promise.resolve(3)]).then(console.log); // [1,2,3]
```

#### Interview (English)
- “Wrap each input with `Promise.resolve`, store results by index, resolve when all fulfill; reject on the first error.”

---

## 13. Promise.race 实现
### Standard — JS
```js
function myRace(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) return reject(new TypeError("Argument must be an array"));
    if (promises.length === 0) return; // stays pending per spec
    for (const p of promises) {
      Promise.resolve(p).then(resolve, reject);
    }
  });
}
```

### Standard — TS
```ts
export function myRace<T>(promises: Array<Promise<T> | T>): Promise<T> {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) return reject(new TypeError("Argument must be an array"));
    if (promises.length === 0) return;
    for (const p of promises) {
      Promise.resolve(p).then(resolve, reject);
    }
  });
}
```

#### Usage Examples
```js
const a = new Promise(r => setTimeout(() => r("A"), 300));
const b = new Promise(r => setTimeout(() => r("B"), 100));
myRace([a, b]).then(console.log); // "B"
```

#### Interview (English)
- “The first settled promise decides the outcome; others are ignored.”

---

## 14. sleep（延迟 / Delay）
### Standard — JS
```js
function sleep(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}
```

### Standard — TS
```ts
export function sleep<T = void>(ms: number, value?: T): Promise<T> {
  return new Promise<T>((resolve) => setTimeout(() => resolve(value as T), ms));
}
```

#### Usage Examples
```js
(async () => {
  console.log("Start");
  await sleep(1000);
  console.log("After 1s");
})();
```

#### Interview (English)
- “`sleep` is `Promise + setTimeout`, often awaited to serialize async flows.”

---

## 15. EventEmitter（发布-订阅 / Pub-Sub）
### Standard — JS
```js
class EventEmitter {
  constructor() {
    this.events = {}; // { eventName: [listeners...] }
  }
  on(event, listener) {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
  }
  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter((l) => l !== listener);
  }
  emit(event, ...args) {
    if (!this.events[event]) return;
    [...this.events[event]].forEach((l) => l(...args));
  }
}
```

### Standard — TS
```ts
type Listener = (...args: any[]) => void;

export class EventEmitter {
  private events: Record<string, Listener[]> = {};
  on(event: string, listener: Listener): void {
    if (!this.events[event]) this.events[event] = [];
    this.events[event].push(listener);
  }
  once(event: string, listener: Listener): void {
    const wrapper: Listener = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
  off(event: string, listener: Listener): void {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter((l) => l !== listener);
  }
  emit(event: string, ...args: any[]): void {
    if (!this.events[event]) return;
    [...this.events[event]].forEach((l) => l(...args));
  }
}
```

#### Usage Examples
```js
const emitter = new EventEmitter();
const greet = (name) => console.log(`Hello, ${name}`);
emitter.on("hi", greet);
emitter.emit("hi", "Stan"); // Hello, Stan
emitter.off("hi", greet);
```

#### Interview (English)
- “Maintain a map from event names to arrays of listeners; `once` is implemented via a wrapper that auto-unsubscribes.”

---

## 16. flatten（数组扁平化 / Array Flatten）
### Standard — JS (recursive)
```js
function flatten(arr) {
  const out = [];
  for (const item of arr) {
    if (Array.isArray(item)) out.push(...flatten(item));
    else out.push(item);
  }
  return out;
}
```

### Standard — TS
```ts
export function flatten<T>(arr: any[]): T[] {
  const out: T[] = [];
  for (const item of arr) {
    if (Array.isArray(item)) out.push(...flatten<T>(item));
    else out.push(item as T);
  }
  return out;
}
```

#### Usage Examples
```js
flatten([1, [2, [3, 4]], 5]); // [1,2,3,4,5]
```

#### Interview (English)
- “Recursively expand nested arrays; an iterative stack variant avoids call-stack limits.”

---

## 17. unique（数组去重 / Array Unique）
### Standard — JS
```js
function unique(arr) {
  return [...new Set(arr)];
}
function uniqueBy(arr, key) {
  const seen = new Set();
  return arr.filter((item) => {
    const v = item[key];
    if (seen.has(v)) return false;
    seen.add(v);
    return true;
  });
}
```

### Standard — TS
```ts
export function unique<T>(arr: T[]): T[] {
  return [...new Set(arr)];
}
export function uniqueBy<T extends Record<string, any>, K extends keyof T>(arr: T[], key: K): T[] {
  const seen = new Set<T[K]>();
  return arr.filter((item) => {
    const v = item[key];
    if (seen.has(v)) return false;
    seen.add(v);
    return true;
  });
}
```

#### Usage Examples
```js
unique([1,2,2,3]); // [1,2,3]
uniqueBy([{id:1},{id:2},{id:1}], "id"); // [{id:1},{id:2}]
```

#### Interview (English)
- “Use `Set` for primitives; for object arrays, filter by a selected key with a `Set` of seen keys.”

---

## 18. LRUCache（最近最少使用缓存）
### Standard — JS (Map-based)
```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  get(key) {
    if (!this.cache.has(key)) return -1;
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  put(key, value) {
    if (this.cache.has(key)) this.cache.delete(key);
    if (this.cache.size >= this.capacity) {
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }
    this.cache.set(key, value);
  }
}
```

### Standard — TS (Doubly linked list + Map)
```ts
class Node<K, V> {
  key!: K;
  value!: V;
  prev: Node<K, V> | null = null;
  next: Node<K, V> | null = null;
  constructor(key?: K, value?: V) {
    if (key !== undefined) this.key = key;
    if (value !== undefined) this.value = value;
  }
}

export class LRUCache<K, V> {
  private capacity: number;
  private map = new Map<K, Node<K, V>>();
  private head = new Node<K, V>();
  private tail = new Node<K, V>();

  constructor(capacity: number) {
    this.capacity = capacity;
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }

  private add(node: Node<K, V>) {
    node.prev = this.tail.prev;
    node.next = this.tail;
    if (this.tail.prev) this.tail.prev.next = node;
    this.tail.prev = node;
  }
  private remove(node: Node<K, V>) {
    if (node.prev) node.prev.next = node.next;
    if (node.next) node.next.prev = node.prev;
    node.prev = node.next = null;
  }

  get(key: K): V | -1 {
    const node = this.map.get(key);
    if (!node) return -1;
    this.remove(node);
    this.add(node);
    return node.value;
  }

  put(key: K, value: V): void {
    if (this.map.has(key)) {
      const node = this.map.get(key)!;
      node.value = value;
      this.remove(node);
      this.add(node);
      return;
    }
    const node = new Node(key, value);
    this.add(node);
    this.map.set(key, node);
    if (this.map.size > this.capacity) {
      const oldest = this.head.next!;
      this.remove(oldest);
      this.map.delete(oldest.key);
    }
  }
}
```

#### Usage Examples
```js
const lru = new LRUCache(2);
lru.put(1, "A");
lru.put(2, "B");
lru.get(1);     // "A"; order becomes [2,1]
lru.put(3, "C"); // evicts 2
lru.get(2);     // -1
```

#### Interview (English)
- “ES6 Map preserves insertion order; delete+set refreshes recency. Algorithmic variant uses a hash map plus doubly linked list for O(1) ops.”

---

## 19. new 操作符实现（Simulate `new`）
### Standard — JS
```js
function myNew(Ctor, ...args) {
  if (typeof Ctor !== "function") throw new TypeError("myNew: constructor must be a function");
  const obj = Object.create(Ctor.prototype);
  const result = Ctor.apply(obj, args);
  return (result !== null && (typeof result === "object" || typeof result === "function")) ? result : obj;
}
```

### Standard — TS
```ts
export function myNew<T extends object, A extends any[]>(
  Ctor: new (...args: A) => T,
  ...args: A
): T {
  const obj = Object.create(Ctor.prototype);
  const result = (Ctor as any).apply(obj, args);
  return (result && (typeof result === "object" || typeof result === "function")) ? result : obj;
}
```

#### Usage Examples
```js
function Person(name){ this.name = name; }
const p = myNew(Person, "Stan");
p.name; // "Stan"
```

#### Interview (English)
- “Create an object linked to `Ctor.prototype`, call `Ctor` with `this=obj`, and return object result if provided.”

---

## 20. call / apply / bind 实现
### Standard — JS
```js
Function.prototype.myCall = function (context, ...args) {
  if (typeof this !== "function") throw new TypeError("myCall must be called on a function");
  const ctx = context == null ? globalThis : Object(context);
  const key = Symbol("fn");
  ctx[key] = this;
  const result = ctx[key](...args);
  delete ctx[key];
  return result;
};

Function.prototype.myApply = function (context, argsArray) {
  if (typeof this !== "function") throw new TypeError("myApply must be called on a function");
  const ctx = context == null ? globalThis : Object(context);
  const key = Symbol("fn");
  ctx[key] = this;
  let result;
  if (argsArray == null) result = ctx[key]();
  else result = ctx[key](...Array.from(argsArray));
  delete ctx[key];
  return result;
};

Function.prototype.myBind = function (context, ...presetArgs) {
  if (typeof this !== "function") throw new TypeError("myBind must be called on a function");
  const targetFn = this;
  function boundFn(...laterArgs) {
    const calledWithNew = this instanceof boundFn;
    const thisArg = calledWithNew ? this : (context == null ? globalThis : Object(context));
    return targetFn.apply(thisArg, [...presetArgs, ...laterArgs]);
  }
  if (targetFn.prototype) {
    boundFn.prototype = Object.create(targetFn.prototype);
  }
  return boundFn;
};
```

### Standard — TS (utility-style)
```ts
export function call<T extends (...args: any[]) => any>(fn: T, context: any, ...args: Parameters<T>) {
  const ctx = context == null ? globalThis : Object(context);
  const key = Symbol("fn");
  (ctx as any)[key] = fn;
  const res = (ctx as any)[key](...args);
  delete (ctx as any)[key];
  return res as ReturnType<T>;
}

export function apply<T extends (...args: any[]) => any>(fn: T, context: any, argsArray?: Parameters<T>) {
  const ctx = context == null ? globalThis : Object(context);
  const key = Symbol("fn");
  (ctx as any)[key] = fn;
  const res = argsArray ? (ctx as any)[key](...(argsArray as any)) : (ctx as any)[key]();
  delete (ctx as any)[key];
  return res as ReturnType<T>;
}

export function bind<T extends (...args: any[]) => any>(fn: T, context: any, ...preset: Parameters<T>) {
  function bound(this: any, ...later: any[]) {
    const isNew = this instanceof (bound as any);
    const thisArg = isNew ? this : (context == null ? globalThis : Object(context));
    return fn.apply(thisArg, [...preset, ...later]);
  }
  (bound as any).prototype = Object.create((fn as any).prototype);
  return bound as unknown as T;
}
```

#### Usage Examples
```js
function greet(g, name){ return `${g}, ${name}`; }
greet.myCall({x:1}, "Hi", "Stan"); // "Hi, Stan"
```

#### Interview (English)
- “Attach the function to the context using a `Symbol` key, invoke, then clean up; `bind` must support constructor usage via `instanceof` check.”

---

## 附录 A：重点术语中英双语速记 / Bilingual Cheat Sheet
- 防抖 Debounce；节流 Throttle；柯里化 Curry；组合 Compose/Pipe；记忆化 Memoize  
- 原型 Prototype；原型链 Prototype Chain；构造函数 Constructor  
- 异步并发组合器 Promise combinators (`all`/`race`)  
- 深拷贝 Deep Clone；浅拷贝 Shallow Clone；深度比较 Deep Equal  
- 事件总线 EventEmitter；最近最少使用 LRU Cache

---

## 附录 B：面试口述模板（英文） / Interview Spiels
> You can reuse these concise English explanations during interviews.

- **Debounce** — “I clear the previous timer and schedule a new one; the function fires on the trailing edge after inactivity. I preserve `this` using `apply` and memoize the debounced handler in React.”  
- **Throttle** — “I track the last invocation time and ensure at most one execution per interval. A trailing call improves UX by not missing the last event.”  
- **Once** — “A closure flag ensures only the first invocation executes; subsequent calls return the cached result.”  
- **Memoize** — “I map argument keys to results via a resolver or JSON; ideal for pure, deterministic functions.”  
- **Curry** — “I collect parameters until `fn.length` is satisfied, then execute; this enables parameter reuse and composition.”  
- **Compose/Pipe** — “Compose runs right-to-left; pipe runs left-to-right; each function consumes the previous output.”  
- **Deep Clone** — “Use a `WeakMap` to handle cycles and avoid memory leaks; branch for Date/RegExp/Map/Set.”  
- **Deep Equal** — “Compare constructors and recursively check keys; handle special types and cycles with `WeakMap`.”  
- **instanceof** — “Walk up the prototype chain comparing against `Ctor.prototype`.”  
- **Object.create** — “A temporary constructor’s prototype points to `proto`, so `new` yields the desired prototype linkage.”  
- **Promise.all** — “Normalize items with `Promise.resolve`, store results by index, resolve when all fulfill; reject on first failure.”  
- **Promise.race** — “The first settled promise determines outcome; the rest are ignored.”  
- **Sleep** — “Promise + `setTimeout`; `await` to serialize steps.”  
- **EventEmitter** — “Map event names to listener arrays; `once` uses a wrapper that auto-unsubscribes.”  
- **Flatten** — “Recursive expansion (or iterative stack) to linearize nested arrays.”  
- **Unique** — “Use a `Set` for primitives; `filter + Set` keyed for objects.”  
- **LRU Cache** — “Map preserves insertion order; or implement a hash map + doubly linked list for O(1) operations.”  
- **new** — “Create object with `Ctor.prototype`, `apply` constructor, and return the explicit object result if provided.”  
- **call/apply/bind** — “Attach via a `Symbol` property, invoke, and delete; `bind` handles constructor calls by checking `instanceof`.”

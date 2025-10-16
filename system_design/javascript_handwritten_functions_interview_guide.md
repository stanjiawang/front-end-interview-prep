
# JavaScript Handwritten Functions — Interview Guide (JS & TS)  
**前端手写函数面试大全（含 JS 与 TS 双版本 | 中文讲解 + 英文代码注释）**  
Author: You (prepared with GPT-5 Thinking)  
Date: 2025-10-16

---

## 使用说明 / How to Use
- **代码 Code**：全部为 **英文注释** + **meaningful naming**，JS 与 TS **双版本**。
- **讲解 Explanation**：中文为主，关键术语 **中英双语**。
- **目的 Purpose**：面试可直接书写；复习可系统理解；项目可直接复用。

> 目录（Table of Contents）
- [1. debounce（防抖）](#1-debounce防抖)
- [2. throttle（节流）](#2-throttle节流)
- [3. once（只执行一次）](#3-once只执行一次)
- [4. memoize（记忆化）](#4-memoize记忆化)
- [5. curry（柯里化）](#5-curry柯里化)
- [6. compose / pipe（函数组合）](#6-compose--pipe函数组合)
- [7. deepClone（深拷贝）+ WeakMap/WeakSet](#7-deepclone深拷贝-weakmapweakset)
- [8. shallowClone（浅拷贝）](#8-shallowclone浅拷贝)
- [9. deepEqual（深度比较）](#9-deepequal深度比较)
- [10. instanceof 实现](#10-instanceof-实现)
- [11. Objectcreate 实现](#11-objectcreate-实现)
- [12. Promiseall 实现](#12-promiseall-实现)
- [13. Promiserace 实现](#13-promiserace-实现)
- [14. sleep（延迟）](#14-sleep延迟)
- [15. EventEmitter（发布-订阅）](#15-eventemitter发布-订阅)
- [16. flatten（数组扁平化）](#16-flatten数组扁平化)
- [17. unique（数组去重）](#17-unique数组去重)
- [18. LRUCache（最近最少使用缓存）](#18-lrucache最近最少使用缓存)
- [19. new 操作符实现](#19-new-操作符实现)
- [20. call / apply / bind 实现](#20-call--apply--bind-实现)
- [附：面试口述模板合集](#附面试口述模板合集)

---

## 1. debounce（防抖）
**定义 Definition**：延迟执行（delay execution）直到**事件停止触发**一段时间（wait until user stops triggering）。  
**典型场景 Typical use cases**：输入框搜索 `input`、窗口 `resize`、表单验证、按钮防连击。  
**Why use `function` not arrow?** 为了**动态绑定 this**（arrow 的 this 是词法绑定）。

### JS
```js
function debounce(fn, delay = 300) {
  let timerId;
  return function (...args) {
    // Preserve dynamic `this` from caller (e.g., class method / event handler)
    const context = this;
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
}
```

### TS
```ts
export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay = 300
) {
  let timerId: ReturnType<typeof setTimeout> | null = null;
  return function (this: ThisParameterType<T>, ...args: Parameters<T>) {
    const context = this;
    if (timerId) clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
}
```

**重点（中英）**：  
- `this（调用时绑定 / dynamic binding）` vs `=>（词法绑定 / lexical binding）`  
- `clearTimeout + setTimeout`：只在**最后一次**触发后执行（only fire on the trailing edge）。

---

## 2. throttle（节流）
**定义**：固定时间间隔内最多执行一次（execute at most once per interval）。  
**场景**：`scroll`、`resize`、`mousemove`。  

### JS（leading-only）
```js
function throttle(fn, delay = 300) {
  let lastTime = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastTime >= delay) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

### TS（leading + trailing）
```ts
export function throttle<T extends (...args: any[]) => any>(
  fn: T,
  delay = 300
) {
  let lastTime = 0;
  let timerId: ReturnType<typeof setTimeout> | null = null;
  return function (this: ThisParameterType<T>, ...args: Parameters<T>) {
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
      // trailing invoke
      timerId = setTimeout(() => {
        lastTime = Date.now();
        timerId = null;
        fn.apply(context, args);
      }, remaining);
    }
  };
}
```

---

## 3. once（只执行一次）
**定义**：函数只在第一次调用时执行并缓存其结果（run only on the first call, cache result）。

### JS
```js
function once(fn) {
  let called = false;
  let result;
  return function (...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

### TS
```ts
export function once<T extends (...args: any[]) => any>(fn: T) {
  let called = false;
  let result: ReturnType<T>;
  return function (this: ThisParameterType<T>, ...args: Parameters<T>) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

---

## 4. memoize（记忆化）
**定义**：缓存函数输入->输出映射，避免重复计算（cache input→output）。

### JS（支持 resolver）
```js
function memoize(fn, resolver) {
  const cache = new Map();
  return function (...args) {
    const key = resolver ? resolver.apply(this, args) : JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

### TS
```ts
export function memoize<T extends (...args: any[]) => any, K = string>(
  fn: T,
  resolver?: (...args: Parameters<T>) => K
) {
  const cache = new Map<K | string, ReturnType<T>>();
  return function (
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

---

## 5. curry（柯里化）
**定义**：把多参数函数转换成可分步传参的链式调用（transform multi-arg fn to chained calls）。

### JS
```js
function curry(fn) {
  return function curried(...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : (...nextArgs) => curried.apply(this, args.concat(nextArgs));
  };
}
```

### TS
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

---

## 6. compose / pipe（函数组合）
**compose**：右到左（right-to-left），**pipe**：左到右（left-to-right）。

### JS
```js
function compose(...fns) {
  return function (initialValue) {
    return fns.reduceRight((acc, fn) => fn(acc), initialValue);
  };
}
function pipe(...fns) {
  return function (initialValue) {
    return fns.reduce((acc, fn) => fn(acc), initialValue);
  };
}
```

### TS
```ts
export function compose<T>(...fns: Array<(x: T) => T>) {
  return (initial: T) => fns.reduceRight((acc, fn) => fn(acc), initial);
}
export function pipe<T>(...fns: Array<(x: T) => T>) {
  return (initial: T) => fns.reduce((acc, fn) => fn(acc), initial);
}
```

---

## 7. deepClone（深拷贝） + WeakMap/WeakSet
**定义**：递归复制对象，处理循环引用（circular refs）与特殊类型（Date/RegExp/Map/Set）。  
**Why WeakMap?** 键为对象且**弱引用**，不阻止 GC；适合记录「已克隆对象」避免内存泄漏。

### JS
```js
function deepClone(value, hash = new WeakMap()) {
  // Base cases: primitives & functions
  if (value === null || typeof value !== "object") return value;
  if (hash.has(value)) return hash.get(value);

  if (value instanceof Date) return new Date(value);
  if (value instanceof RegExp) return new RegExp(value);

  if (value instanceof Map) {
    const result = new Map();
    hash.set(value, result);
    value.forEach((v, k) => result.set(deepClone(k, hash), deepClone(v, hash)));
    return result;
  }
  if (value instanceof Set) {
    const result = new Set();
    hash.set(value, result);
    value.forEach(v => result.add(deepClone(v, hash)));
    return result;
  }

  const result = Array.isArray(value) ? [] : {};
  hash.set(value, result);

  for (const key in value) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      result[key] = deepClone(value[key], hash);
    }
  }
  return result;
}
```

### TS
```ts
export function deepClone<T>(value: T, hash = new WeakMap()): T {
  if (value === null || typeof value !== "object") return value;
  if (hash.has(value as any)) return hash.get(value as any);

  if (value instanceof Date) return new Date(value) as any;
  if (value instanceof RegExp) return new RegExp(value) as any;

  if (value instanceof Map) {
    const result = new Map();
    hash.set(value, result);
    value.forEach((v, k) => result.set(deepClone(k, hash), deepClone(v, hash)));
    return result as any;
  }
  if (value instanceof Set) {
    const result = new Set();
    hash.set(value, result);
    value.forEach(v => result.add(deepClone(v, hash)));
    return result as any;
  }

  const result: any = Array.isArray(value) ? [] : {};
  hash.set(value as any, result);
  for (const key in value as any) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      result[key] = deepClone((value as any)[key], hash);
    }
  }
  return result;
}
```

**WeakMap vs Map（要点）**：  
- WeakMap 键必须是对象（keys must be objects），**弱引用**（garbage-collection friendly），不可枚举（non-enumerable）。  
- 适合：deepClone 防循环引用、私有数据封装、缓存不影响 GC。

**WeakSet（补充）**：只能存对象、弱引用、不可枚举，适合“访问标记”。

---

## 8. shallowClone（浅拷贝）
### JS
```js
function shallowClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  const result = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      result[key] = obj[key];
    }
  }
  return result;
}
```

### TS
```ts
export function shallowClone<T extends object>(obj: T): T {
  if (obj === null || typeof obj !== "object") return obj;
  const result: any = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      result[key] = (obj as any)[key];
    }
  }
  return result;
}
```

---

## 9. deepEqual（深度比较）
### JS
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
    for (const [k, v] of a) if (!b.has(k) || !deepEqual(v, b.get(k), visited)) return false;
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

### TS
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

---

## 10. instanceof 实现
### JS
```js
function myInstanceof(obj, Ctor) {
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) return false;
  if (typeof Ctor !== "function") throw new TypeError("Right-hand side of instanceof must be a function");
  let proto = Object.getPrototypeOf(obj);
  const target = Ctor.prototype;
  while (proto) {
    if (proto === target) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

### TS
```ts
export function myInstanceof(obj: any, Ctor: Function): boolean {
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) return false;
  if (typeof Ctor !== "function") throw new TypeError("Right-hand side of instanceof must be a function");
  let proto = Object.getPrototypeOf(obj);
  const target = Ctor.prototype;
  while (proto) {
    if (proto === target) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

---

## 11. Object.create 实现
### JS
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

### TS
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

---

## 12. Promise.all 实现
### JS
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

### TS
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

---

## 13. Promise.race 实现
### JS
```js
function myRace(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) return reject(new TypeError("Argument must be an array"));
    if (promises.length === 0) return; // pending forever (spec-aligned)
    for (const p of promises) {
      Promise.resolve(p).then(resolve, reject);
    }
  });
}
```

### TS
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

---

## 14. sleep（延迟）
### JS
```js
function sleep(ms, value) {
  return new Promise((resolve) => setTimeout(() => resolve(value), ms));
}
```

### TS
```ts
export function sleep<T = void>(ms: number, value?: T): Promise<T> {
  return new Promise<T>((resolve) => setTimeout(() => resolve(value as T), ms));
}
```

---

## 15. EventEmitter（发布-订阅）
### JS
```js
class EventEmitter {
  constructor() {
    this.events = {}; // { event: [listeners] }
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

### TS
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

---

## 16. flatten（数组扁平化）
### JS
```js
function flatten(arr) {
  const result = [];
  for (const item of arr) {
    if (Array.isArray(item)) result.push(...flatten(item));
    else result.push(item);
  }
  return result;
}
```

### TS
```ts
export function flatten<T>(arr: any[]): T[] {
  const result: T[] = [];
  for (const item of arr) {
    if (Array.isArray(item)) result.push(...flatten<T>(item));
    else result.push(item as T);
  }
  return result;
}
```

---

## 17. unique（数组去重）
### JS
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

### TS
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

---

## 18. LRUCache（最近最少使用）
### JS（简洁 Map 版）
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

### TS（双向链表 + Map 版）
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
  private head = new Node<K, V>(); // dummy head
  private tail = new Node<K, V>(); // dummy tail

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

---

## 19. new 操作符实现
### JS
```js
function myNew(Ctor, ...args) {
  if (typeof Ctor !== "function") throw new TypeError("myNew: constructor must be a function");
  const obj = Object.create(Ctor.prototype);
  const result = Ctor.apply(obj, args);
  return (result !== null && (typeof result === "object" || typeof result === "function")) ? result : obj;
}
```

### TS
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

---

## 20. call / apply / bind 实现
### JS
```js
Function.prototype.myCall = function (context, ...args) {
  if (typeof this !== "function") throw new TypeError("myCall must be called on a function");
  const ctx = context == null ? globalThis : Object(context);
  const fnKey = Symbol("fn");
  ctx[fnKey] = this;
  const result = ctx[fnKey](...args);
  delete ctx[fnKey];
  return result;
};

Function.prototype.myApply = function (context, argsArray) {
  if (typeof this !== "function") throw new TypeError("myApply must be called on a function");
  const ctx = context == null ? globalThis : Object(context);
  const fnKey = Symbol("fn");
  ctx[fnKey] = this;
  let result;
  if (argsArray == null) {
    result = ctx[fnKey]();
  } else {
    if (!Array.isArray(argsArray) && typeof argsArray[Symbol.iterator] !== "function") {
      if (!(typeof argsArray === "object" && "length" in argsArray)) {
        throw new TypeError("CreateListFromArrayLike called on non-object");
      }
      argsArray = Array.from(argsArray);
    }
    result = ctx[fnKey](...argsArray);
  }
  delete ctx[fnKey];
  return result;
};

Function.prototype.myBind = function (context, ...presetArgs) {
  if (typeof this !== "function") throw new TypeError("myBind must be called on a function");
  const targetFn = this;
  function boundFn(...laterArgs) {
    const isCalledWithNew = this instanceof boundFn;
    const thisArg = isCalledWithNew ? this : (context == null ? globalThis : Object(context));
    return targetFn.apply(thisArg, [...presetArgs, ...laterArgs]);
  }
  if (targetFn.prototype) {
    boundFn.prototype = Object.create(targetFn.prototype);
  }
  return boundFn;
};
```

### TS（声明合并需要在 .d.ts 中扩展，演示核心逻辑函数式版本）
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

---

## 附：面试口述模板合集（关键点中英）
- **debounce**：*“I clear the previous timer and start a new one, so the function fires only on the trailing edge after the user stops triggering.”*（清除前一次定时器、只在最后一次触发后执行）  
- **throttle**：*“I keep an execution timestamp and ensure the function runs at most once per interval.”*（使用时间戳，保证固定间隔）  
- **once**：*“Use a closure flag to run only on the first call and cache the result.”*（闭包标记 + 结果缓存）  
- **memoize**：*“Cache input→output; use resolver to build keys.”*（缓存入参到结果，可自定义 key）  
- **curry**：*“Collect args until reaching fn.length, then execute.”*（参数收集到位后执行）  
- **compose/pipe**：*“Right-to-left vs left-to-right function composition.”*（右到左 / 左到右组合）  
- **deepClone**：*“Use WeakMap to avoid cycles; handle Map/Set/Date/RegExp specially.”*（WeakMap 防环，特殊类型分支）  
- **deepEqual**：*“Check constructors and recursively compare keys; handle Map/Set/Date/RegExp.”*（构造器+递归键值）  
- **instanceof**：*“Walk the prototype chain to match Ctor.prototype.”*（沿原型链匹配）  
- **Object.create**：*“Temporary ctor whose prototype points to `proto`.”*（临时构造函数的原型指向传入原型）  
- **Promise.all**：*“Wrap each item with Promise.resolve, count fulfills, reject on first error.”*（统一包装、计数、出错即失败）  
- **Promise.race**：*“First settled decides the outcome.”*（谁先 settle 取谁）  
- **sleep**：*“Promise + setTimeout, await to pause.”*（Promise 封装定时器）  
- **EventEmitter**：*“Maintain event→listeners map; once via wrapper that auto-unsubscribes.”*（事件映射表，once 包装后自注销）  
- **flatten**：*“Recursive expansion; stack version for non-recursive.”*（递归/栈）  
- **unique**：*“Use Set; for objects use filter + Set by key.”*（基础用 Set，对象按 key）  
- **LRUCache**：*“Map keeps insertion order; delete+set to refresh; or use doubly linked list + map.”*（Map 顺序/链表+哈希）  
- **new**：*“Create object with Ctor.prototype, apply Ctor, return object result if provided.”*（三步 + 返回规则）  
- **call/apply/bind**：*“Attach fn via Symbol, invoke, cleanup; bind must support `new`.”*（Symbol 临时挂载；bind 需兼容构造）

---

> 祝你面试顺利！Good luck & have fun building! 🚀

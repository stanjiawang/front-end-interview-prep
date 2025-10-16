# 📘 JavaScript 手写函数面试题集 / JS Handwritten Utilities
---

## 📑 目录 / Table of Contents

1. [debounce（防抖）](#1-debounce防抖)
2. [throttle（节流）](#2-throttle节流)
3. [once（只执行一次）](#3-once只执行一次)
4. [memoize（记忆化）](#4-memoize记忆化)
5. [curry（函数柯里化）](#5-curry函数柯里化)
6. [compose（函数组合）](#6-compose函数组合)
7. [deepClone（深拷贝）](#7-deepclone深拷贝)
8. [call / apply / bind 实现](#8-call--apply--bind-实现)
9. [new 操作符实现](#9-new-操作符实现)
10. [instanceof 实现](#10-instanceof-实现)
11. [Promise.all 实现](#11-promiseall-实现)
12. [sleep（延迟函数）](#12-sleep延迟函数)
13. [flatten（数组扁平化）](#13-flatten数组扁平化)
14. [unique（数组去重）](#14-unique数组去重)
15. [EventEmitter（发布订阅）](#15-eventemitter发布订阅)

---

## 🧩 1. debounce（防抖）

```js
/**
 * debounce（防抖）
 * 在连续触发事件时，只执行最后一次。
 * Debounce: Execute only after continuous triggers stop.
 */
// ✅ 最适合函数组件 / hooks 环境
function debounce(fn, delay = 300) {
  let timerId;

  return (...args) => {
    clearTimeout(timerId);
    timerId = setTimeout(() => fn(...args), delay);
  };
}

// ✅ Example
function SearchBox() {
  const handleChange = debounce((e) => {
    console.log("Search:", e.target.value);
  }, 500);

  return <input onChange={handleChange} />;
}

// ✅ 通用写法，可安全用于对象方法 / 类组件
function debounce(fn, delay = 300) {
  let timerId;

  return function (...args) {
    const context = this; // 捕获调用者上下文
    clearTimeout(timerId);
    timerId = setTimeout(() => fn.apply(context, args), delay);
  };
}

const obj = {
  value: 42,
  sayHi() {
    console.log(this.value);
  },
};

obj.debouncedHi = debounce(obj.sayHi, 500);
obj.debouncedHi(); // ✅ 正常输出 42
```

---

## ⚙️ 2. throttle（节流）
什么是 Throttle（节流）

“节流（throttle）指的是无论事件触发多频繁，
只在固定时间间隔内执行一次函数。”

适用场景：scroll、resize、mousemove、window.onresize

高频但不希望过度执行的操作

```js
/**
 * throttle（节流）
 * 固定时间间隔内只执行一次。
 * Throttle: Execute at most once per given interval.
 */
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

// ✅ Example
window.addEventListener(
  "scroll",
  throttle(() => {
    console.log("Scroll event fired!");
  }, 1000)
);
```

---

## 🧠 3. once（只执行一次）

```js
/**
 * once（只执行一次）
 * 函数只会执行一次，以后调用无效。
 * Execute a function only once.
 */
const once = (fn) => {
  let called = false;
  return (...args) => {
    if (!called) {
      called = true;
      return fn(...args);
    }
  };
};

// ✅ Example
const init = once(() => console.log('Initialized'));
init(); // ✅
init(); // ❌ ignored
```

---

## 💾 4. memoize（记忆化）

```js
/**
 * memoize（记忆化函数）
 * 缓存函数结果以提高性能。
 * Cache function results to avoid recalculation.
 */
const memoize = (fn) => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
};

// ✅ Example
const slowAdd = (a, b) => { for (let i = 0; i < 1e6; i++); return a + b; };
const fastAdd = memoize(slowAdd);
console.log(fastAdd(1, 2)); // computed
console.log(fastAdd(1, 2)); // cached
```

---

## 🧩 5. curry（函数柯里化）

```js
/**
 * curry（柯里化）
 * 将多参数函数转化为链式调用。
 * Transform multi-argument function into chained calls.
 */
const curry = (fn) => {
  const curried = (...args) =>
    args.length >= fn.length ? fn(...args) : (...next) => curried(...args, ...next);
  return curried;
};

// ✅ Example
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
```

---

## 🔁 6. compose（函数组合）

```js
/**
 * compose（函数组合）
 * 从右到左执行多个函数。
 * Compose functions right to left.
 */
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

// ✅ Example
const double = (x) => x * 2;
const square = (x) => x ** 2;
console.log(compose(square, double)(3)); // (3*2)^2 = 36
```

---

## 🧱 7. deepClone（深拷贝）

```js
/**
 * deepClone（深拷贝）
 * 递归复制对象，避免引用共享。
 * Deep copy object recursively, handle circular refs.
 */
const deepClone = (obj, map = new WeakMap()) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (map.has(obj)) return map.get(obj);
  const clone = Array.isArray(obj) ? [] : {};
  map.set(obj, clone);
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) clone[key] = deepClone(obj[key], map);
  }
  return clone;
};
```

---

## ⚙️ 8. call / apply / bind 实现

```js
/**
 * 手写 call / apply / bind
 * Simulate Function.prototype.call/apply/bind
 */
Function.prototype.myCall = function (context, ...args) {
  context = context || globalThis;
  const key = Symbol();
  context[key] = this;
  const result = context[key](...args);
  delete context[key];
  return result;
};

Function.prototype.myApply = function (context, args) {
  return this.myCall(context, ...(args || []));
};

Function.prototype.myBind = function (context, ...preset) {
  return (...later) => this.myCall(context, ...preset, ...later);
};
```

---

## 🧩 9. new 操作符实现

```js
/**
 * new 操作符实现
 * Simulate the "new" keyword behavior.
 */
const myNew = (Ctor, ...args) => {
  const obj = Object.create(Ctor.prototype);
  const result = Ctor.apply(obj, args);
  return typeof result === 'object' && result !== null ? result : obj;
};
```

---

## 🧠 10. instanceof 实现

```js
/**
 * instanceof 实现
 * Check prototype chain manually.
 */
const myInstanceof = (obj, Ctor) => {
  let proto = Object.getPrototypeOf(obj);
  while (proto) {
    if (proto === Ctor.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
};
```

---

## 🧭 11. Promise.all 实现

```js
/**
 * Promise.all 实现
 * Resolve all promises, reject if any fails.
 */
const promiseAll = (promises) => {
  return new Promise((resolve, reject) => {
    const results = [];
    let count = 0;
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        (v) => {
          results[i] = v;
          if (++count === promises.length) resolve(results);
        },
        (e) => reject(e)
      );
    });
  });
};
```

---

## ⏳ 12. sleep（延迟函数）

```js
/**
 * sleep（延迟）
 * Delay execution by ms.
 */
const sleep = (ms) => new Promise((r) => setTimeout(r, ms));
```

---

## 🧩 13. flatten（数组扁平化）

```js
/**
 * flatten（数组扁平化）
 * Convert nested arrays into a single array.
 */
const flatten = (arr) => arr.reduce(
  (acc, cur) => acc.concat(Array.isArray(cur) ? flatten(cur) : cur),
  []
);
```

---

## 🧮 14. unique（数组去重）

```js
/**
 * unique（数组去重）
 * Remove duplicate elements.
 */
const unique = (arr) => [...new Set(arr)];
```

---

## 🔁 15. EventEmitter（发布订阅）

```js
/**
 * EventEmitter（发布订阅）
 * Simple implementation of event bus.
 */
class EventEmitter {
  constructor() { this.events = {}; }

  on(event, listener) {
    (this.events[event] ||= []).push(listener);
  }

  off(event, listener) {
    this.events[event] = (this.events[event] || []).filter(l => l !== listener);
  }

  emit(event, ...args) {
    (this.events[event] || []).forEach(l => l(...args));
  }
}
```

---

# ✅ End of File

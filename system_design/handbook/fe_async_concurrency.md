## 异步与并发处理：请求调度与事件循环

### 题干描述

**问题示例：**
> “JavaScript 在浏览器中是如何处理异步操作的？请解释事件循环机制，并举例说明在前端如何调度大量并发请求以避免阻塞。”

此类问题要求候选人深入讲解 **JavaScript 的异步并发模型（事件循环、任务队列等）**，以及在实际开发中如何有效管理多个异步任务或并发请求，比如控制请求并发数、处理竞态条件等。

---

### 背后考察意图

面试官希望通过此题考察候选人是否：

1. 深刻理解 **事件循环 (Event Loop)** 的运行机制，包括调用栈、宏任务、微任务、渲染阶段等。
2. 掌握 **异步编程模型**（Promise、async/await、回调、生成器）及其背后的执行原理。
3. 能够在实际业务中设计 **并发请求调度**、**竞态控制**、**错误处理** 等机制。
4. 理解浏览器的 **多队列任务模型** 和 UI 渲染优先级，能写出不卡顿的高性能异步代码。

---

### 结构化答题框架

#### 一、事件循环原理讲解（Event Loop Fundamentals）

**1. 单线程模型与异步需求**
- JavaScript 是单线程语言，所有代码都在一个主线程上执行。
- 若所有任务都同步执行，长耗时操作（I/O、网络请求）会阻塞 UI 响应。
- 因此浏览器引入了 **事件循环机制 (Event Loop)**，让异步任务在后台执行，待结果就绪后再调度执行回调。

**2. 调用栈 (Call Stack)**
- JS 引擎执行同步代码时，函数按 LIFO（后进先出）进入调用栈。
- 栈满时若任务执行时间过长，将导致页面“假死”或“卡顿”。

**3. 任务队列 (Task Queue)**
- 异步操作（定时器、网络请求、DOM 事件）完成后，其回调会被放入任务队列。
- 当调用栈清空后，事件循环会从任务队列取出一个任务压入栈执行。

**4. 宏任务与微任务**
- 任务分两类：
  - **宏任务 (Macro Task)：** script（整体代码）、setTimeout、setInterval、I/O、UI 事件。
  - **微任务 (Micro Task)：** Promise.then、MutationObserver、queueMicrotask。
- 事件循环执行规则：
  - 执行一个宏任务 → 清空所有微任务 → 浏览器渲染 → 再取下一个宏任务。
- 示例：
```js
console.log('start');
setTimeout(() => console.log('timeout'), 0);
Promise.resolve().then(() => console.log('promise'));
console.log('end');
// 输出顺序：start → end → promise → timeout
```

**5. 特殊队列与渲染阶段**
- `requestAnimationFrame`：在浏览器下一帧绘制前执行（适合动画逻辑）。
- `requestIdleCallback`：在浏览器空闲时执行（适合低优先级任务）。
- 浏览器渲染阶段穿插在事件循环中，主线程空闲时触发 UI 更新。

---

#### 二、实际应用：异步与并发控制

**1. 并发请求调度（Request Pooling / Concurrency Control）**
- 当需要发起大量请求时，应限制同时进行的请求数量。
- 例如同时最多 5 个请求在飞，其他排队等待。

实现思路：维护请求队列和计数器：
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
- 优点：避免浏览器连接数上限（通常同域名 6 个连接）。
- 应用场景：批量图片加载、批量 API 调用。

**2. 避免竞态条件（Race Condition）**
- 示例：搜索框实时联想，用户快速输入多次时发出多次请求。
- 解决方案：
  - **防抖 (debounce)：** 等用户停止输入一定时间后再请求。
  - **请求标识机制：** 只处理最后一次请求响应。
  - **AbortController：** 取消之前的 fetch 请求。

示例：
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

**3. 异步错误与超时处理**
- `Promise.allSettled()`：等待所有任务完成（成功或失败）。
- `Promise.race()`：用于超时控制。
```js
const timeout = new Promise((_, rej) => setTimeout(() => rej('Timeout'), 5000));
await Promise.race([fetchData(), timeout]);
```

**4. async/await 并发陷阱**
- `await` 会暂停当前 async 函数执行，相当于串行。
- 若两个请求可并行，应使用 Promise.all。
```js
// 串行（低效）
await fetchA();
await fetchB();
// 并行（高效）
await Promise.all([fetchA(), fetchB()]);
```

---

#### 三、事件循环与执行顺序示例

**经典面试题：**
```js
console.log('script start');
setTimeout(() => console.log('setTimeout'), 0);
Promise.resolve().then(() => console.log('promise1')).then(() => console.log('promise2'));
console.log('script end');
```
**输出顺序：**
```
script start
script end
promise1
promise2
setTimeout
```
**解析：**
- 同步代码先执行（script start, script end）。
- Promise.then 属于微任务，先于 setTimeout 执行。
- setTimeout 是宏任务，下一轮事件循环执行。

---

#### 四、Web Worker 与并行处理

- JavaScript 主线程是单线程，但可借助 **Web Worker** 实现多线程。
- 适用场景：计算密集型任务（视频转码、加密、AI 推理等）。
- Worker 与主线程通过消息机制通信（postMessage / onmessage）。
- 注意：Worker 无法访问 DOM，数据通信需序列化。

示例：
```js
// main.js
const worker = new Worker('worker.js');
worker.postMessage(data);
worker.onmessage = (e) => console.log('Result:', e.data);
```

---

#### 五、调度与节流机制

1. **防抖 (Debounce)**：在连续触发事件后，仅在最后一次触发后执行。
```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

2. **节流 (Throttle)**：保证一段时间内只执行一次。
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

#### 六、锁与同步控制

- 在复杂并发场景下，可能需要逻辑锁（mutex-like）。
- 前端常用“状态锁”方案：
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
- 避免同一逻辑同时多次触发，保证一致性。

---

#### 七、最佳实践与注意事项

1. **理解底层机制而非死记顺序**
   - 明白为何 Promise.then 先于 setTimeout，有助于编写正确逻辑。
2. **避免阻塞主线程**
   - 将重任务拆分，通过 setTimeout(fn, 0) 或 requestIdleCallback 分批执行。
3. **控制并发数量**
   - 合理设置请求上限，避免服务器过载或浏览器连接瓶颈。
4. **错误与超时处理**
   - 所有异步任务应具备超时与重试机制。
5. **统一异步编码风格**
   - 团队约定统一使用 async/await，避免混用回调与 Promise 链。
6. **持续优化与监控**
   - 利用 Performance API 分析任务执行时间，优化耗时操作。
7. **关注新标准与能力**
   - 了解 Promise.allSettled、AbortController、Atomics、SharedArrayBuffer 等。

---

### 总结

通过深入理解事件循环与异步并发模型：
- 能编写高性能、可控的异步代码。
- 优化并发请求调度，提高资源利用率。
- 避免 UI 阻塞、竞态问题与内存压力。

**核心结论：**
> 事件循环是 JavaScript 异步体系的核心，理解它不仅能写出更流畅的前端应用，更是进入大厂面试中展现深度思考与工程能力的关键。
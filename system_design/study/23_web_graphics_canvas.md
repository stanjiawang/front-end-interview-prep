# 23 — Modern Web Graphics & Canvas Rendering (学习级详细版 / Full Learning Guide)

---

## 🧠 Overview 概述

Modern front-end graphics go beyond static DOM. **Canvas**, **WebGL**, and **WebGPU** provide high-performance, low-level APIs for rendering complex visual experiences.

> 💡 中文：现代前端图形技术已从静态 DOM 发展到使用 Canvas、WebGL、WebGPU 等高性能图形 API 实现复杂渲染与动画。

---

## 1. Graphics Rendering in the Browser（浏览器渲染体系）

### 1.1 Rendering Layers
| Layer | API | Type | Use Case |
|--------|-----|------|-----------|
| **DOM** | HTML/CSS | Declarative | UI Layout |
| **Canvas 2D** | Canvas API | Imperative | Games, charts |
| **WebGL** | OpenGL ES 2.0 | GPU API | 3D graphics |
| **WebGPU** | Modern GPU API | Parallel compute | Next-gen rendering |

**Rendering Pipeline Diagram:**
```
JS → API Calls → GPU Driver → GPU Commands → Framebuffer → Screen
```

> 💡 中文：DOM 渲染依赖浏览器引擎，Canvas 与 WebGL 则直接与 GPU 通信，实现自定义渲染管线。

---

## 2. Canvas API & OffscreenCanvas（Canvas 与离屏渲染）

### 2.1 Canvas 2D Example
```html
<canvas id="canvas" width="400" height="300"></canvas>
<script>
const ctx = document.getElementById("canvas").getContext("2d");
ctx.fillStyle = "skyblue";
ctx.fillRect(50, 50, 200, 100);
ctx.strokeStyle = "black";
ctx.lineWidth = 4;
ctx.strokeRect(50, 50, 200, 100);
</script>
```

> 💡 中文：Canvas 提供逐像素绘制接口，常用于游戏、图表、图片处理等场景。

### 2.2 Animation Example
```js
let x = 0;
function draw() {
  ctx.clearRect(0, 0, 400, 300);
  ctx.beginPath();
  ctx.arc(x, 150, 20, 0, Math.PI * 2);
  ctx.fillStyle = "orange";
  ctx.fill();
  x = (x + 2) % 400;
  requestAnimationFrame(draw);
}
draw();
```

> 💡 中文：通过 requestAnimationFrame 可实现高帧率动画。

### 2.3 OffscreenCanvas (Web Worker)
```js
// main.js
const worker = new Worker("worker.js");
const canvas = document.querySelector("canvas");
const offscreen = canvas.transferControlToOffscreen();
worker.postMessage({ canvas: offscreen }, [offscreen]);
```

```js
// worker.js
onmessage = (e) => {
  const ctx = e.data.canvas.getContext("2d");
  function draw(t) {
    ctx.clearRect(0, 0, 300, 150);
    ctx.fillRect(Math.sin(t / 500) * 100 + 150, 50, 30, 30);
    requestAnimationFrame(draw);
  }
  draw(0);
};
```

> 💡 中文：OffscreenCanvas 可将渲染任务移动至 Worker 线程，避免阻塞主线程。

---

## 3. WebGL Fundamentals（WebGL 基础）

### 3.1 WebGL Context Setup
```js
const gl = canvas.getContext("webgl");
gl.clearColor(0.1, 0.2, 0.3, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);
```

### 3.2 Vertex Shader & Fragment Shader
```glsl
// vertex.glsl
attribute vec2 position;
void main() {
  gl_Position = vec4(position, 0.0, 1.0);
}
```

```glsl
// fragment.glsl
void main() {
  gl_FragColor = vec4(1.0, 0.5, 0.0, 1.0);
}
```

### 3.3 JS Program Binding
```js
const program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
gl.useProgram(program);
```

> 💡 中文：WebGL 使用 GLSL 着色器语言在 GPU 中运行程序，实现顶点与像素级渲染控制。

### 3.4 Geometry & Buffer
```js
const vertices = new Float32Array([
  -0.5, -0.5,
   0.5, -0.5,
   0.0,  0.5
]);
const buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
```

> 💡 中文：WebGL 通过缓冲区存储顶点数据并在 GPU 上绘制。

---

## 4. WebGPU & Modern Acceleration（WebGPU 与现代图形加速）

### 4.1 Why WebGPU?
| Feature | WebGL | WebGPU |
|----------|--------|--------|
| API Design | OpenGL style | Modern GPU pipeline |
| Performance | Limited compute | High parallelism |
| Language | GLSL | WGSL |
| Resource Control | Implicit | Explicit |

> 💡 中文：WebGPU 是下一代图形 API，提供显式的 GPU 控制与计算着色能力。

### 4.2 WebGPU Example
```js
const adapter = await navigator.gpu.requestAdapter();
const device = await adapter.requestDevice();
const context = canvas.getContext("webgpu");

const format = navigator.gpu.getPreferredCanvasFormat();
context.configure({ device, format });

const commandEncoder = device.createCommandEncoder();
const pass = commandEncoder.beginRenderPass({ colorAttachments: [] });
pass.end();
device.queue.submit([commandEncoder.finish()]);
```

> 💡 中文：WebGPU 提供显式的渲染管线定义，可实现并行计算与高级渲染。

---

## 5. Use Cases（实际应用）

| Scenario | Technology | Description |
|-----------|-------------|-------------|
| Data Visualization | Canvas / WebGL | D3.js + GPU rendering |
| Game Development | WebGL / WebGPU | 3D engine (Three.js, Babylon.js) |
| Image Processing | OffscreenCanvas | Filters, resizing |
| Machine Learning | WebGPU | Tensor computations |
| Simulation | WebGL / WebGPU | Physics / particle systems |

> 💡 中文：Canvas 与 WebGPU 已广泛用于数据可视化、AI 推理与 3D 游戏。

---

## 6. Interview-Oriented Section（面试导向）

### 6.1 Key Question
**“Explain the difference between Canvas, WebGL, and WebGPU.”**

**Answer Outline:**
1. Canvas 2D: CPU-based pixel rendering.  
2. WebGL: GPU-based 3D API using shaders.  
3. WebGPU: Modern explicit GPU pipeline, compute shaders, better performance.  

### 6.2 Trade-off Table
| API | Pros | Cons |
|-----|------|------|
| Canvas 2D | Simple, wide support | CPU bound |
| WebGL | GPU acceleration | Complex setup |
| WebGPU | Best performance | Limited browser support |

---

## 🧩 Summary 总结

| Layer | Focus | Example |
|-------|--------|----------|
| Canvas | 2D Rendering | Charts, games |
| WebGL | 3D GPU Rendering | Three.js |
| WebGPU | Compute + Render | AI / Physics |
| OffscreenCanvas | Parallel rendering | Workers |

> 💡 中文总结：从 Canvas 到 WebGPU，前端图形渲染正在迈向高性能并行时代。掌握渲染管线与 GPU 编程是下一代前端工程师的关键能力。

---

📘 **Next Chapter → 24. Mobile Web & Responsive Design**

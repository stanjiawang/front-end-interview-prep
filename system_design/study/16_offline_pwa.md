# 16 — Offline & Progressive Web Applications (学习级详细版 / Full Learning Guide)

---

## 🧠 Overview 概述

A **Progressive Web Application (PWA)** combines the reach of the web with the reliability and performance of native apps.  
It’s fast, resilient, and installable — even under flaky or no-network conditions.

> 💡 中文：PWA 是具备离线能力、原生体验与快速性能的现代 Web 应用。核心技术包括 Service Worker、Web Manifest 和缓存策略。

---

## 1. PWA Fundamentals（基础原理）

### 1.1 Core Components
| Component | Description |
|------------|--------------|
| **Service Worker** | Background script that intercepts network requests and manages caching |
| **Web App Manifest** | JSON file describing app metadata (icons, theme, name, start URL) |
| **HTTPS** | Required security context |
| **App Shell** | Cached minimal UI to render instantly offline |

### 1.2 Architecture Diagram
```
   ┌───────────────┐
   │    Browser    │
   └──────┬────────┘
          │
          ▼
   ┌───────────────┐
   │ ServiceWorker │ ←─ Intercepts requests
   ├───────────────┤
   │ Cache Storage │ ←─ Stores assets & data
   ├───────────────┤
   │ IndexedDB     │ ←─ Stores structured data
   └───────────────┘
```

---

## 2. Service Worker（核心）

### 2.1 Registration
```js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js").then(() => {
    console.log("Service Worker registered!");
  });
}
```

### 2.2 Lifecycle
| Phase | Description | Event |
|--------|-------------|--------|
| **Install** | Download & cache assets | `install` |
| **Activate** | Cleanup old caches | `activate` |
| **Fetch** | Intercept network requests | `fetch` |

```js
self.addEventListener("install", (e) => {
  e.waitUntil(
    caches.open("v1").then((cache) => cache.addAll(["/", "/index.html", "/main.css", "/app.js"]))
  );
});

self.addEventListener("fetch", (e) => {
  e.respondWith(caches.match(e.request).then((r) => r || fetch(e.request)));
});
```

> 💡 中文：Service Worker 拦截请求，可优先返回缓存或重新拉取网络资源，从而实现离线体验。

---

## 3. Caching Strategies（缓存策略）

| Strategy | Description | Example Use |
|-----------|--------------|--------------|
| **Cache First** | Use cached data, fallback to network | Static assets (logo, fonts) |
| **Network First** | Try network, fallback to cache | API responses |
| **Stale-While-Revalidate** | Return cache, update in background | News feeds |
| **Network Only** | Always fetch | Admin dashboards |
| **Cache Only** | Strict offline | Static documentation |

### Example: Stale-While-Revalidate
```js
self.addEventListener("fetch", (e) => {
  e.respondWith(
    caches.open("dynamic").then(async (cache) => {
      const cached = await cache.match(e.request);
      const network = fetch(e.request).then((resp) => {
        cache.put(e.request, resp.clone());
        return resp;
      });
      return cached || network;
    })
  );
});
```

> 💡 中文：SW 策略应根据资源类型选择，避免全局缓存带来的数据陈旧。

---

## 4. Background Sync & Deferred Actions（后台同步与延迟操作）

### 4.1 Background Sync
```js
self.addEventListener("sync", (event) => {
  if (event.tag === "sync-posts") event.waitUntil(uploadQueuedPosts());
});
```

### 4.2 Queued POSTs Example
```js
async function uploadQueuedPosts() {
  const posts = await readFromIndexedDB();
  for (const post of posts) {
    await fetch("/api/posts", { method: "POST", body: JSON.stringify(post) });
    await removeFromIndexedDB(post.id);
  }
}
```

> 💡 中文：后台同步可在离线时暂存请求，联网后自动重放，提升可靠性。

---

## 5. Web App Manifest

**manifest.json**
```json
{
  "name": "My PWA App",
  "short_name": "PWAApp",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

> 💡 中文：Manifest 文件定义了应用的图标、名称、启动路径和主题色，是 PWA 可安装的关键。

---

## 6. Workbox Implementation（Workbox 实战）

**Installation:**
```bash
npm install workbox-cli --save-dev
```

**workbox-config.js**
```js
module.exports = {
  globDirectory: "dist/",
  globPatterns: ["**/*.{html,js,css,png,svg}"],
  swDest: "dist/sw.js",
  runtimeCaching: [
    {
      urlPattern: ({ request }) => request.destination === "image",
      handler: "CacheFirst",
      options: { cacheName: "images", expiration: { maxEntries: 50 } },
    },
  ],
};
```

**Build Command:**
```bash
npx workbox generateSW workbox-config.js
```

> 💡 中文：Workbox 自动生成 SW 脚本，实现资源预缓存、运行时缓存与更新策略。

---

## 7. Offline UX Design（离线体验设计）

| Pattern | Description | Implementation |
|----------|--------------|----------------|
| **App Shell** | Load cached UI instantly | Cache core HTML/CSS/JS |
| **Skeleton** | Visual placeholder | `<Skeleton />` component |
| **Offline Banner** | Notify user of network loss | `navigator.onLine` events |
| **Deferred Actions** | Queue user operations | IndexedDB + Background Sync |
| **Cached Fallback** | Show last data snapshot | SW + IndexedDB |

### Example Offline Banner
```tsx
const [online, setOnline] = useState(navigator.onLine);
useEffect(() => {
  const update = () => setOnline(navigator.onLine);
  window.addEventListener("online", update);
  window.addEventListener("offline", update);
  return () => {
    window.removeEventListener("online", update);
    window.removeEventListener("offline", update);
  };
}, []);
return !online && <div className="banner">You are offline</div>;
```

---

## 8. Testing & Debugging PWA

### 8.1 Chrome DevTools
- Application → Service Workers → “Offline” checkbox.  
- Lighthouse → PWA audit section.  
- Network throttling simulation.  

### 8.2 Common Pitfalls
- Not unregistering old SWs.  
- Cache invalidation missing version keys.  
- Fetch handler blocking async code.  

```js
self.skipWaiting(); // activate immediately
```

> 💡 中文：调试时需手动清理缓存并确保 Service Worker 正确更新。

---

## 9. Interview-Oriented Section

### 9.1 Key Question
**“How would you design a reliable offline-first web application?”**

**Answer Outline:**
1. Cache App Shell via Service Worker.  
2. Use Stale-While-Revalidate for data freshness.  
3. Queue POST requests using Background Sync.  
4. Store structured data in IndexedDB.  
5. Provide offline UX fallback (banner, skeleton).

### 9.2 Trade-off Table
| Option | Pros | Cons |
|---------|------|------|
| Cache-First | Fast, reliable | May serve stale data |
| Network-First | Fresh data | Slower, requires network |
| Background Sync | Reliable uploads | Complex logic |
| IndexedDB | Structured offline data | Larger footprint |

---

## 🧩 Summary 总结

| Concept | Focus | Tool |
|----------|--------|------|
| **Service Worker** | Request interception | Native API / Workbox |
| **Caching Strategy** | Offline resilience | Cache API |
| **Manifest** | Installable PWA | manifest.json |
| **Background Sync** | Deferred actions | SyncManager |
| **UX Design** | Offline awareness | Skeleton / Banner |

> 💡 中文总结：PWA 是现代 Web 的可靠性架构核心。通过 Service Worker 缓存、离线交互与后台同步，用户体验不再受网络波动影响。

---

📘 **Next Chapter → 17. Real-Time Collaboration & WebSockets**

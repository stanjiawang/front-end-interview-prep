## 网络与缓存：CDN 策略与缓存一致性

### 题干描述

**问题示例：**
> “如果让你为 TikTok 这样的大型 Web 应用设计前端网络请求和缓存方案，你会怎么做？尤其请说明如何利用 CDN 加速静态资源，以及如何保证缓存数据的一致性。”

此类问题要求候选人阐述在前端架构中如何有效利用 CDN、浏览器缓存等提高网络性能，并讨论在多层缓存下保持数据同步与一致性的策略。

---

### 背后考察意图

面试官希望考察候选人是否：

1. 理解 **CDN（内容分发网络）** 的原理与部署策略。
2. 熟悉 **HTTP 缓存机制**（强缓存、协商缓存、ETag、Cache-Control 等）。
3. 掌握 **缓存一致性**（Cache Invalidation）与失效控制方法。
4. 能够设计 **多层缓存架构**（浏览器缓存、CDN 缓存、服务端缓存）并平衡性能与数据正确性。
5. 对 **大规模、高流量应用（如 TikTok）** 的网络性能优化有实战经验与全局视角。

---

### 结构化答题框架

#### 一、CDN 加速策略（Content Delivery Network）

**1. CDN 原理与作用**
- CDN 是一种内容分发网络，将静态资源（如 JS、CSS、图片、视频封面等）分发至全球各地的边缘节点。
- 用户请求资源时会自动访问距离最近的节点（通过 DNS 解析），减少网络延迟与源站压力。

**2. CDN 部署与命名策略**
- 将静态文件托管到独立域名（例如 `static.tiktokcdn.com`）。
- 部署流程中添加 **内容哈希（Content Hash）** 到文件名中，如：
  ```
  main.9af1e3c.js
  styles.aa82df.css
  ```
- 内容变化 → 文件哈希变化 → 新 URL → 自动避开旧缓存。

**3. CDN 缓存配置**
- 设置长时间缓存：`Cache-Control: public, max-age=31536000, immutable`。
- 通过内容哈希解决一致性问题：新版本发布时直接引用新 URL。
- CDN 缓存命中率高达 95%+ 时，可显著降低带宽与源站负载。
- 配置预热（Prefetching）或预取（Preloading），确保热门资源在 CDN 节点提前加载。

**4. CDN 回源与多级冗余**
- CDN 节点缓存未命中时会向源站请求资源（称为“回源”）。
- 可使用多源站部署或多 CDN 提供商（Multi-CDN）架构，增强容灾能力。
- 大型应用如 TikTok 通常采用 **多区域源站 + 动态路由调度系统**（如基于延迟或带宽选择最优路径）。

**5. CDN 示例方案**
> “例如，TikTok Web 的主 JS/CSS 文件托管在 CDN 上，并配置 1 年长缓存。每次构建发布时，Webpack 自动生成带 hash 的文件名，新版本资源的 URL 不会与旧版冲突。HTML 页面则使用短缓存（几分钟）或 no-cache，以便引用最新的静态资源 URL。这样用户既享受 CDN 缓存带来的极速加载，又不会遇到版本不一致问题。”

---

#### 二、浏览器缓存与网络策略

**1. 浏览器缓存类型**
- **Memory Cache**：存在于内存中，关闭标签即失效，适用于短时复用。
- **Disk Cache**：存储在硬盘中，可跨会话复用。
- **Service Worker Cache**：可自定义缓存逻辑，支持离线访问与版本控制。

**2. 强缓存与协商缓存**
- **强缓存 (Strong Cache)**：浏览器根据 `Cache-Control` 或 `Expires` 决定是否直接使用本地资源。
  ```
  Cache-Control: public, max-age=86400
  ```
  在有效期内不请求服务器。
- **协商缓存 (Conditional Request)**：缓存过期后，浏览器带上 `If-None-Match`（ETag）或 `If-Modified-Since`，服务器返回 304 表示内容未变。
  ```
  ETag: "v1.2.3"
  ```
  当文件内容变化时，服务器更新 ETag 值，触发重新下载。

**3. 离线与预缓存（Service Worker）**
- 使用 Service Worker 实现前端层缓存控制：
  - 在 `install` 阶段缓存必要资源（HTML shell、核心 JS）。
  - 在 `fetch` 阶段优先返回缓存内容，提高离线或弱网体验。
  - 通过版本号机制（Cache Version）在新版本发布时更新缓存。
- 示例：
  ```js
  const CACHE_NAME = 'app-cache-v2';
  self.addEventListener('install', event => {
    event.waitUntil(caches.open(CACHE_NAME).then(cache => cache.addAll(['/index.html', '/main.js'])));
  });
  self.addEventListener('fetch', event => {
    event.respondWith(caches.match(event.request).then(res => res || fetch(event.request)));
  });
  ```

**4. 动态数据缓存策略**
- 对 API 请求结果进行客户端缓存。
- 使用 Apollo Client / React Query 维护缓存层：
  - 设定数据 TTL（Time-To-Live）。
  - 缓存过期后自动重新请求。
- 对冷数据可使用 `localStorage` 或 `IndexedDB`。
- 支持后台静默刷新：加载时先展示缓存数据，再异步请求更新最新内容。

---

#### 三、缓存一致性与失效控制

**1. 静态资源一致性：文件指纹策略**
- 对于静态文件（JS/CSS/图片等），使用内容哈希版本号解决一致性问题。
- 每次构建生成新的 hash → HTML 引用新 URL → 浏览器/CDN 自动区分版本。
- HTML 本身不宜使用长缓存，应配置：
  ```
  Cache-Control: no-cache, must-revalidate
  ```
  这样每次都会验证是否有新版本。

**2. 动态数据一致性**
- 前端缓存的 API 数据可能过期，例如用户粉丝数或视频点赞数。
- 常见策略：
  - **TTL（时间驱动）**：设定缓存有效期（如 60s）。过期自动刷新。
  - **主动刷新**：后端通过 WebSocket 或 SSE 推送最新数据，前端更新缓存。
  - **协商缓存 (ETag)**：服务端返回版本号或哈希值，客户端验证是否变化。

**3. CDN 缓存失效与刷新机制**
- 发布新版本时，可通过：
  - **文件指纹**（推荐）：自动失效旧缓存。
  - **主动清除 (Purge API)**：调用 CDN 的刷新接口清除旧缓存。
  - **短 TTL 策略**：为关键资源设置较短缓存周期。

**4. 多终端一致性问题**
- 多 Tab 或多设备可能各自缓存不同版本数据。
- 可用 `BroadcastChannel` 或 `StorageEvent` 实现跨标签页同步更新：
  ```js
  const channel = new BroadcastChannel('cache-sync');
  channel.postMessage('invalidate-user-cache');
  ```
- 打开页面时强制刷新关键数据，确保最终一致性（Eventual Consistency）。

---

#### 四、安全性与边缘情况处理

**1. 鉴权数据不可缓存**
- 对涉及用户隐私的请求添加：
  ```
  Cache-Control: private, no-store
  ```
- 避免 CDN 或共享缓存误缓存敏感内容。

**2. 缓存容量与清理**
- 浏览器本地缓存有限制（通常几十 MB），需定期清理。
- 可采用 LRU（Least Recently Used）策略删除旧缓存。

**3. CDN 异常与回源策略**
- 监控 CDN 命中率与节点延迟。
- CDN 故障时自动回源访问主站。
- 大型系统如 TikTok 使用多 CDN 切换（Akamai + Cloudflare + 自建）。

**4. 数据完整性验证**
- 对关键资源使用 **Subresource Integrity (SRI)** 校验：
  ```html
  <script src="main.js" integrity="sha384-abc123" crossorigin="anonymous"></script>
  ```
- 防止 CDN 被篡改导致前端执行异常脚本。

---

#### 五、缓存策略分类与一致性模型

**1. 缓存模式分类**
- **Cache-Aside（旁路缓存）**：先查缓存，无则访问源并回填缓存。
- **Write-Through**：写入数据时同时更新缓存。
- **Write-Back（延迟写回）**：先写缓存，后异步同步源数据。

**2. 一致性级别**
- **强一致性 (Strong Consistency)**：每次读取都是最新数据（代价高）。
- **最终一致性 (Eventual Consistency)**：允许短暂不同步，最终达到一致（多数 CDN 与浏览器缓存采用）。

**3. 三种缓存失效策略**
- 主动失效（Write-Invalidate）
- 更新缓存（Write-Update）
- 定期过期（TTL）

> 前端常采用 TTL 策略（例如 React Query 中的 stale-while-revalidate 模式），因为客户端无法实时获知数据变化。

---

#### 六、最佳实践与注意事项

1. **按数据类型分类缓存策略**
   - 静态资源：长缓存 + 内容哈希。
   - 动态接口：根据实时性分级缓存（数秒 / 数分钟 / 不缓存）。
2. **监控与可视化**
   - 监控 CDN 缓存命中率、回源率、304 响应比例。
   - 若命中率低，检查 URL 版本策略是否失效。
3. **防止过度缓存**
   - 严格控制缓存更新流程，发布流程应自动刷新相关版本资源。
   - 提供用户手动刷新（如“点击刷新”按钮清除本地缓存）。
4. **前后端协同策略**
   - 统一缓存版本号与失效机制。
   - 后端可支持 `forceRefresh=true` 参数以允许前端强制刷新缓存。
5. **持续优化与验证**
   - 通过 Real User Monitoring (RUM) 分析资源加载时间。
   - 定期回顾 CDN 节点性能与用户地理分布调整策略。

---

### 总结

一个优秀的前端网络与缓存架构应做到：

- **高性能：** CDN 边缘节点 + 本地缓存 + 合理 TTL。
- **高一致性：** 哈希版本控制 + ETag 协商验证。
- **高可靠性：** 多 CDN 冗余 + 回源策略 + SRI 校验。

> TikTok 级别的 Web 应用在缓存设计上追求“性能与一致性的平衡”：让静态资源 100% 命中 CDN，同时确保数据更新不滞后，是前端系统设计的关键能力。


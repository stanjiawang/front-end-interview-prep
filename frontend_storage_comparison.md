# 🌐 前端存储机制全解析

## 一、浏览器端存储全景概览

| 类型 | 名称 | 说明 | 容量 | 是否持久化 | 可被JS访问 | 典型用途 |
|------|------|------|--------|---------------|---------------|-------------|
| **基础键值对存储** | `localStorage` | 永久保存在浏览器的键值对 | ~5–10 MB | ✅ 是 | ✅ 是 | 用户设置、缓存数据 |
|  | `sessionStorage` | 仅在当前标签页会话有效 | ~5 MB | ❌ 否（关闭即失效） | ✅ 是 | 临时状态、表单缓存 |
| **HTTP级存储** | `cookie` | 浏览器和服务器之间自动传输的小数据 | ~4 KB | ✅（可设过期） | ✅（除非HttpOnly） | 登录状态、追踪 |
| **结构化存储** | `IndexedDB` | 面向对象数据库，可存储结构化大数据 | 数百 MB–几 GB | ✅ 是 | ✅ 是 | 离线应用、缓存大对象 |
| **文件存储** | `File System Access API` | 允许 Web 应用直接访问用户本地文件系统（需授权） | 受限于磁盘 | ✅ 是 | ✅（需授权） | 离线编辑器、IDE、PWA |
| **缓存型存储** | `Cache Storage` (Service Worker) | 专用于网络请求响应缓存 | 无严格上限（约数百 MB） | ✅ 是 | ✅ 是 | 离线访问、PWA缓存 |
| **数据库式存储（旧）** | `WebSQL` *(已废弃)* | 早期SQL型浏览器数据库 | ~50 MB | ✅ 是 | ✅ 是 | ❌ 不再推荐使用 |
| **会话级内存存储** | `window.name` | 数据绑定在窗口标签的生命周期内 | 几百 KB | ❌ 否 | ✅ 是 | 页面间简单通信 |
| **存储 API 封装** | `StorageManager` | 查询和管理配额、持久化权限 | N/A | ✅ 是 | ✅ 是 | 控制持久化策略 |
| **应用级缓存** | `Service Worker + Cache API` | 离线资源、预缓存、动态缓存 | 取决于系统配额 | ✅ 是 | ✅ 是 | PWA、离线访问 |

---

## 二、核心原理与机制

### 1. localStorage / sessionStorage
- **底层机制：** 基于浏览器内部的 `Storage` 接口，以键值对形式存储在磁盘或内存中。
- **隔离性：** 同源策略控制（协议 + 域名 + 端口）。
- **生命周期：**
  - `localStorage`：永久保存，手动或清除缓存时删除。
  - `sessionStorage`：随标签页关闭自动删除。
- **实现机制：**
  - 浏览器在打开网页时，会为当前源初始化一个内存对象（`Storage` 实例）。
  - 所有操作都在此对象上进行（同步API）。
  - 数据最终序列化为字符串存储在本地磁盘中。

### 2. Cookie
- **底层机制：** 属于 HTTP 协议一部分，通过 `Set-Cookie` 响应头设置。
- **自动传输：** 每次请求自动随 HTTP 头部发送至服务器（除非 `HttpOnly` 禁止 JS 访问）。
- **生命周期：**
  - 默认浏览器会话结束即清除。
  - 可通过 `Expires` 或 `Max-Age` 指定持久化时间。
- **安全机制：**
  - `Secure`：仅HTTPS传输。
  - `HttpOnly`：防止JS读取，抵御XSS。
  - `SameSite`：防止跨站请求伪造（CSRF）。

### 3. IndexedDB
- **底层机制：** 异步的 NoSQL 数据库，基于事务和对象存储（object store）。
- **数据结构：** 支持索引、游标、事务。
- **用途：** 存储大体量 JSON、Blob、ArrayBuffer 数据。
- **API特征：** 异步操作（事件驱动），比 Storage API 更复杂。

### 4. Cache Storage (Service Worker)
- **机制：** 由 Service Worker 管理的响应缓存系统。
- **可缓存内容：** HTTP 请求及响应。
- **优势：** 离线访问、静态资源预加载。
- **应用：** PWA 应用的离线访问核心。

### 5. File System Access API
- **原理：** 调用原生文件系统接口（需用户授权）。
- **机制：**
  - 调用 `window.showOpenFilePicker()` 或 `showSaveFilePicker()`。
  - 获取文件句柄（FileHandle）后，可读写本地文件。
- **安全限制：** 需 HTTPS + 用户操作触发。

---

## 三、容量与性能差异

| 存储类型 | 容量 | 性能 | 适用场景 |
|-----------|--------|-----------|-------------|
| Cookie | < 4 KB | 慢（每次请求附带） | 登录、认证 |
| Session Storage | ~5 MB | 快（内存中） | 临时页面状态 |
| Local Storage | ~5–10 MB | 中（同步API） | 用户配置缓存 |
| IndexedDB | 100MB–几GB | 高（异步API） | 离线大数据 |
| Cache Storage | 数百MB–几GB | 高（异步） | 离线缓存资源 |
| File System Access | 磁盘大小 | 取决于IO | 文件操作类应用 |

---

## 四、安全性对比

| 等级 | 类型 | 风险 | 防护 |
|-------|--------|--------|--------|
| ⭐⭐⭐⭐ | Cookie (HttpOnly + Secure) | 中 | 防XSS防CSRF |
| ⭐⭐⭐ | IndexedDB / Cache | 低 | 沙箱隔离 |
| ⭐⭐ | localStorage / sessionStorage | 中高 | 易受XSS影响 |
| ⭐ | window.name | 高 | 不推荐 |

---

## 五、应用建议

| 场景 | 推荐存储 | 理由 |
|------|------------|-------|
| 登录状态 | HttpOnly Cookie | 安全、防XSS |
| 用户主题/语言 | localStorage | 简单持久 |
| 临时表单 | sessionStorage | 页面级缓存 |
| 离线Web应用 | IndexedDB + Cache Storage | 支持大数据离线 |
| 离线文件编辑 | File System Access API | 本地读写 |
| 统计与埋点 | Cookie | 跨页面持久 |

---

## 六、前端存储体系结构总结

> 浏览器存储体系可类比成金字塔：

```
      ┌──────────────────────────┐
      │ Cookie (认证、跨域)      │
      ├──────────────────────────┤
      │ localStorage / session   │
      ├──────────────────────────┤
      │ IndexedDB / CacheStorage │
      ├──────────────────────────┤
      │ FileSystem / PWA Cache   │
      └──────────────────────────┘
```

**一句话总结：**
> 小数据存储用 Storage，大数据缓存用 IndexedDB，认证用 Cookie，离线用 Cache。

---

## 七、常见面试问答

1. **为什么不能在 localStorage 存放 token？**
   - 因为容易被 XSS 读取，安全风险高。
   - 推荐放在 `HttpOnly Cookie`。

2. **localStorage 和 sessionStorage 的底层区别？**
   - 都基于 `Storage` 接口，但生命周期不同（磁盘 vs 内存）。

3. **IndexedDB 和 Cache Storage 的区别？**
   - IndexedDB 存数据结构化信息。
   - Cache Storage 存HTTP响应。

4. **Service Worker 与存储有什么关系？**
   - 它控制缓存的生命周期与策略，实现真正的离线支持。

---

# ✅ 总结

| 分类 | 技术 | 核心用途 | 特点 |
|------|------|-----------|------|
| 认证级 | Cookie | 登录状态 | 自动随请求发送 |
| 状态级 | SessionStorage | 页面状态 | 随标签失效 |
| 偏持久 | LocalStorage | 用户偏好 | 永久存储 |
| 离线级 | IndexedDB / Cache | 离线缓存 | 异步高容量 |
| 文件级 | File System Access | 文件操作 | 需授权 |

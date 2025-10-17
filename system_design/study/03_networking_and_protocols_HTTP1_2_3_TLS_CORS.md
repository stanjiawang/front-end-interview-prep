# 03 — Networking and Protocols (HTTP/1.1 → HTTP/2 → HTTP/3, TLS, CORS)
# 网络与协议详解（HTTP/1.1 → HTTP/2 → HTTP/3、TLS、安全与跨域机制）

> **中英双语完整版 | Detailed Bilingual Guide**  
> This document explains how browser networking works — from DNS to HTTP/3 and CORS — in both English and Chinese.  
> 本文深入讲解浏览器的网络机制：从 DNS 到 HTTP/3，再到 CORS 跨域与缓存系统。  
> Each section contains: English explanation, Chinese explanation, diagrams, and interview templates.  
> 每一节都包含：英文原理、中文解释、图示、面试口述模板。

---

## Table of Contents
1. Browser Networking Overview 浏览器网络总览  
2. DNS, TCP, and TLS 基础协议层：DNS、TCP 与 TLS  
3. HTTP/1.1 — Mechanism & Limitations 工作机制与局限  
4. HTTP/2 — Multiplexing & Header Compression 多路复用与头部压缩  
5. HTTP/3 — QUIC over UDP 基于 UDP 的 QUIC 协议  
6. TLS — Encryption and Handshake 安全传输与握手过程  
7. Same-Origin Policy & CORS 同源策略与跨域资源共享  
8. Web Caching Hierarchy 缓存体系结构（Memory → HTTP → CDN）  
9. Content Delivery Network (CDN) 内容分发网络  
10. Performance Optimization Strategies 性能优化策略  
11. Interview Questions & Speaking Templates 面试问题与口述模板

---

## 1. Browser Networking Overview  
### 浏览器网络总览

**English Explanation:**  
The browser networking stack handles everything from DNS resolution to rendering resources on screen. Each page load involves multiple layers — application (HTTP), transport (TCP/UDP), network (IP), and link layers.

**中文解释：**  
浏览器的网络栈负责从域名解析到最终页面渲染的所有步骤。整个过程跨越多个层级：应用层（HTTP 协议）、传输层（TCP/UDP）、网络层（IP）、链路层（物理传输）。  
理解这一流程是前端系统设计面试的核心，因为它直接影响性能与安全。

**Simplified flow (简化流程图)：**  
```
User → Browser → DNS → TCP/TLS → HTTP → Response → Rendering (DOM/CSSOM)
```

**Key takeaway:** Every optimization at the front end ultimately affects one of these layers.  
前端的每一次性能优化，最终都作用于这些网络层中的某一个。

---

## 2. DNS, TCP, and TLS  
### DNS、TCP 与 TLS 的协作原理

**English Explanation:**  
1. **DNS Resolution:** Translates a domain name into an IP address.  
2. **TCP Handshake:** Establishes a reliable connection (3-way handshake).  
3. **TLS Handshake:** Adds encryption and identity verification.  

**中文解释：**  
1. **DNS 解析**：将域名转换为 IP 地址。浏览器先查本地缓存、本机 DNS、再到权威服务器。  
2. **TCP 握手**：通过三次握手建立可靠连接。  
   - SYN → SYN-ACK → ACK  
3. **TLS 握手**：在 TCP 之上添加安全层，完成加密算法协商与服务器身份认证。  

**Diagram (流程图)：**  
```
Browser → DNS → IP  
Browser → TCP Handshake → Server  
Browser → TLS Handshake → Secure Channel  
Browser → HTTP Request/Response  
```

**面试口述模板 (Interview Speaking Template):**  
> “When a user enters a URL, the browser performs DNS resolution, then a TCP handshake followed by TLS negotiation. Once the secure channel is established, the HTTP request is sent.”  
> 中文口述：当用户输入 URL，浏览器先进行 DNS 解析，然后是 TCP 三次握手，再进行 TLS 安全握手，最后才发送 HTTP 请求。

---

## 3. HTTP/1.1 — Mechanism & Limitations  
### HTTP/1.1 的工作机制与局限

**English:**  
- Uses text-based requests and responses.  
- Each request/response pair travels in sequence per connection.  
- Persistent connections via `Connection: keep-alive` reduce TCP setup cost.  
- Major issues: head-of-line (HOL) blocking and limited concurrency.

**中文：**  
HTTP/1.1 是基于文本的请求-响应协议。每个请求都必须等待上一个完成才能继续发送。虽然 `keep-alive` 可以复用连接，但浏览器仍限制同域并发连接数（通常 6 个）。  
这种串行化导致了性能瓶颈——即 **头阻塞 (Head-of-Line Blocking)**。

**Example:**  
```http
GET /index.html HTTP/1.1
Host: example.com
Connection: keep-alive
```

**面试问答：**  
**Q:** Why is HTTP/1.1 inefficient?  
**A:** Because multiple requests on the same connection must wait sequentially, leading to head-of-line blocking.  
**中文回答：** 因为同一连接上的请求必须依次处理，导致前一个响应阻塞后续请求，降低并发效率。

---

## 4. HTTP/2 — Multiplexing & Header Compression  
### HTTP/2：多路复用与头部压缩

**English:**  
HTTP/2 replaces text with binary framing. Multiple requests share one TCP connection using independent streams identified by stream IDs. It adds **multiplexing**, **header compression (HPACK)**, and optional **server push**.

**中文：**  
HTTP/2 引入了二进制帧层（Binary Framing Layer），多个请求可以通过同一个 TCP 连接并行发送，每个请求都有独立的流 ID，从而解决了应用层的头阻塞问题。头部通过 **HPACK 算法** 压缩，大幅减少重复字段。

**ASCII Diagram:**  
```
TCP Connection
 ├── Stream 1: HTML
 ├── Stream 2: CSS
 ├── Stream 3: JS
 └── Stream 4: Image
```

**面试问答：**  
**Q:** Does HTTP/2 completely eliminate head-of-line blocking?  
**A:** Not entirely. It removes it at the HTTP layer, but since all streams share one TCP connection, a lost packet still blocks all streams.  
**中文回答：** 没有完全消除。HTTP/2 解决了应用层的阻塞，但由于所有流仍依赖一个 TCP 连接，一旦丢包，TCP 的可靠机制会阻塞所有请求。

---

## 5. HTTP/3 — QUIC over UDP  
### HTTP/3：基于 UDP 的 QUIC 协议

**English:**  
HTTP/3 uses QUIC, built over UDP. QUIC integrates transport and TLS, providing faster connection setup and independent streams. It supports connection migration and 0-RTT handshakes.

**中文：**  
HTTP/3 基于 UDP 构建 QUIC 协议，将传输层与 TLS 加密整合在一起。  
它避免了 TCP 的队头阻塞（每个流独立传输），还能在网络切换时保持连接（如 Wi-Fi 切到 4G），并支持 0-RTT 握手（无需重复往返）。

**面试模板：**  
> “HTTP/3 integrates TLS 1.3 into QUIC, running over UDP. It removes transport-level HOL blocking and speeds up connection setup.”  
> 中文口述：HTTP/3 在 UDP 上运行，将 TLS1.3 集成入 QUIC，消除了传输层的队头阻塞并加快连接建立。

---

## 6. TLS — Encryption & Handshake  
### TLS 加密与握手

**English:**  
TLS (Transport Layer Security) ensures confidentiality, integrity, and authenticity.  
The handshake negotiates encryption algorithms and verifies certificates.

**中文：**  
TLS（传输层安全协议）提供数据加密、完整性与身份验证。握手阶段会协商加密算法并验证服务器证书。  
TLS 1.3 将握手从 4 次往返缩减至 1 次，极大提高了连接速度。

**Diagram:**  
```
ClientHello → ServerHello → Certificate → Finished
```
**面试提示：**  
> TLS ensures that even if packets are intercepted, the content remains unreadable and tamper-proof.  
> TLS 确保数据即使被拦截也无法被读取或篡改。

---

## 7. Same-Origin Policy & CORS  
### 同源策略与跨域资源共享

**English:**  
The **Same-Origin Policy (SOP)** restricts scripts from interacting with resources from a different origin (protocol + domain + port).  
**CORS (Cross-Origin Resource Sharing)** provides a controlled relaxation via server-set headers.

**中文：**  
**同源策略** 限制网页脚本访问不同源的资源（源指协议+域名+端口组合）。  
**CORS** 机制允许服务器通过特定响应头授权跨域访问。浏览器在实际请求前发送 **预检请求（OPTIONS）** 检查权限。

**Example:**  
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Credentials: true
```

**面试模板：**  
> “CORS works through a preflight OPTIONS request; the browser ensures the server explicitly allows the origin and method.”  
> 中文口述：CORS 通过预检 OPTIONS 请求实现，浏览器在正式请求前确认服务器是否允许该源与方法。

---

## 8. Web Caching Hierarchy  
### 浏览器缓存层级

```
Memory Cache → Service Worker Cache → HTTP Cache → CDN → Origin
```

**English:**  
Browsers use multiple caching layers. HTTP caching uses headers like `Cache-Control`, `ETag`, and `Last-Modified` to validate resources.

**中文：**  
浏览器缓存分为多层：内存缓存、Service Worker 缓存、HTTP 缓存、CDN 缓存。  
HTTP 缓存通过 `Cache-Control`, `ETag` 等头部字段控制资源是否重新请求。

```http
Cache-Control: max-age=31536000, immutable
ETag: "v1.0.3"
```

---

## 9. CDN — Content Delivery Network  
### 内容分发网络

**English:**  
CDNs store cached assets at edge servers globally, reducing latency and offloading traffic from origin servers.

**中文：**  
CDN 在全球的边缘节点缓存静态资源（图片、脚本等），使用户就近访问，降低延迟、减轻源站压力。  
现代 CDN 还支持边缘计算（Edge Rendering）。

---

## 10. Performance Optimization Strategies  
### 网络层性能优化策略

| Category | Strategy | 中文解释 |
|-----------|-----------|-----------|
| **Connection** | Reuse TCP/TLS (keep-alive, session resumption) | 复用连接，减少握手成本 |
| **Compression** | Gzip/Brotli | 压缩响应体 |
| **Caching** | CDN + ETag + Service Worker | 多层缓存减少请求 |
| **Preloading** | `<link rel="preload">` / `<dns-prefetch>` | 提前加载关键资源 |
| **HTTP/2+3** | Multiplexing & QUIC | 并发传输与低延迟 |
| **Security** | Strict CSP & HSTS | 保证安全传输 |

---

## 11. Interview Questions & Speaking Templates  
### 面试问题与口述模板

**Q1. Explain the difference between HTTP/1.1, HTTP/2, and HTTP/3.**  
**A (EN):** HTTP/1.1 uses text and sequential requests; HTTP/2 introduces multiplexing; HTTP/3 uses QUIC over UDP to remove transport-level blocking.  
**A (中文):** HTTP/1.1 是文本串行传输，HTTP/2 增加多路复用解决应用层阻塞，HTTP/3 基于 UDP 的 QUIC 消除了传输层阻塞。

---

**Q2. What is TLS and why is it important?**  
**A (EN):** TLS encrypts communication, prevents eavesdropping, and authenticates the server.  
**A (中文):** TLS 提供数据加密、防止窃听，并验证服务器身份，是安全通信的基础。

---

**Q3. What happens during a DNS lookup?**  
**A (EN):** Browser checks local cache → OS → DNS resolver → authoritative servers → returns IP.  
**A (中文):** 浏览器先查本地缓存，再查系统解析器，最后向权威 DNS 服务器查询，获得 IP 地址。

---

**Q4. How does CORS ensure security while allowing flexibility?**  
**A (EN):** It only allows cross-origin access if the server explicitly grants permission via headers.  
**A (中文):** 浏览器默认禁止跨域，只有服务器明确返回允许头时才放行，实现安全与灵活性的平衡。

---

**Q5. What are caching validation mechanisms?**  
**A (EN):** ETag, Last-Modified with If-None-Match / If-Modified-Since.  
**A (中文):** 使用 ETag 与 Last-Modified 头验证资源是否更新，未更新则返回 304 节省流量。

---

**Q6. How does HTTP/3 improve mobile performance?**  
**A (EN):** QUIC supports connection migration and faster recovery from packet loss.  
**A (中文):** QUIC 支持连接迁移（切网不断连）并在丢包时更快恢复，因此对移动端延迟更友好。

---

**Q7. How can front-end engineers influence network performance?**  
**A (EN):** By minimizing blocking assets, using preloading, caching, compression, and monitoring CWV.  
**A (中文):** 通过减少阻塞资源、使用预加载、多层缓存与压缩，并实时监控核心 Web 指标。

---

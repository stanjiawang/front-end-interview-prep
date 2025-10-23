## Network and Caching: CDN Strategy and Cache Consistency

### Problem Description

**Example Question:**
> “If you were to design a front-end network and caching strategy for a large-scale web application like TikTok, how would you do it? Please explain how you would leverage a CDN to accelerate static assets and ensure cache consistency.”

This question evaluates a candidate’s ability to design an efficient front-end network architecture that utilizes CDNs and caching to improve performance, while maintaining data consistency across multiple cache layers.

---

### Interviewer’s Intent

The interviewer is testing whether the candidate:

1. Understands **how CDNs (Content Delivery Networks)** work and how to configure them for optimal static asset delivery.
2. Is familiar with **HTTP caching mechanisms** — strong caching, conditional requests, `ETag`, and `Cache-Control` headers.
3. Can handle **cache invalidation and consistency**, ensuring users always receive up-to-date data.
4. Can design a **multi-layer caching strategy** (browser cache, CDN cache, backend cache) that balances performance and correctness.
5. Has **practical experience** optimizing high-traffic applications (like TikTok) for global delivery.

---

### Structured Answer Framework

#### 1. CDN Acceleration Strategy

**1.1 CDN Overview**
- A CDN (Content Delivery Network) distributes static assets (JS, CSS, images, videos) across geographically distributed edge nodes.
- When a user requests a resource, it is served from the nearest node via DNS routing — minimizing latency and offloading traffic from the origin server.

**1.2 CDN Deployment and Versioning**
- Host static resources on a dedicated domain, e.g., `static.tiktokcdn.com`.
- Use **content hashes** for versioning:
  ```
  main.9af1e3c.js
  styles.aa82df.css
  ```
- When the content changes, the hash (and thus URL) changes — ensuring automatic cache busting.

**1.3 CDN Cache Configuration**
- Apply long-term caching headers:
  ```
  Cache-Control: public, max-age=31536000, immutable
  ```
- Use versioned filenames to ensure old assets don’t conflict.
- Pre-warm CDN caches (prefetch popular resources before deployment).
- Target >95% CDN cache hit rate to reduce origin load.

**1.4 Multi-CDN and Failover Strategy**
- If a CDN edge misses or fails, it performs an **origin fetch** from the main server.
- Use **multi-CDN architecture** (Akamai, Cloudflare, Fastly) for redundancy.
- Implement **real-time routing** to select the fastest CDN region based on latency.

**1.5 CDN Example (TikTok Scenario)**
> “In TikTok Web, static assets like JS, CSS, and icons are deployed to a CDN with long-lived cache headers. Each build includes a unique content hash in filenames. The HTML entry file has a short cache lifetime (or no-cache) to always fetch the latest references. This combination achieves both fast load speed and version consistency.”

---

#### 2. Browser Caching and Network Strategies

**2.1 Browser Cache Types**
- **Memory Cache:** Temporary, cleared when the tab closes.
- **Disk Cache:** Persistent, reused across sessions.
- **Service Worker Cache:** Controlled via JavaScript for offline or PWA support.

**2.2 Strong and Conditional Caching**
- **Strong Cache:** Uses `Cache-Control` or `Expires` to skip revalidation.
  ```
  Cache-Control: public, max-age=86400
  ```
- **Conditional Cache:** Uses `ETag` or `Last-Modified` to validate freshness.
  ```
  ETag: "v1.2.3"
  ```
  When the content changes, the server returns a new ETag; otherwise, a 304 response confirms the cached version is still valid.

**2.3 Service Worker Caching**
- Use Service Workers to cache key assets for offline access or faster reloads.
  ```js
  const CACHE_NAME = 'app-cache-v2';
  self.addEventListener('install', event => {
    event.waitUntil(caches.open(CACHE_NAME).then(cache => cache.addAll(['/index.html', '/main.js'])));
  });
  self.addEventListener('fetch', event => {
    event.respondWith(caches.match(event.request).then(res => res || fetch(event.request)));
  });
  ```
- Implement versioned cache names to invalidate old versions automatically.

**2.4 Dynamic Data Caching**
- Use libraries like **Apollo Client** or **React Query** for API data caching with TTLs.
- Persist non-critical data (e.g., user preferences, viewed videos) in `localStorage` or `IndexedDB`.
- Employ background updates: show cached data instantly, then refresh with newer data in the background.

---

#### 3. Cache Consistency and Invalidation

**3.1 Static Resource Consistency**
- Use **content hashing** to guarantee static asset freshness.
- HTML files (which reference JS/CSS) should use:
  ```
  Cache-Control: no-cache, must-revalidate
  ```
  to ensure they always fetch the latest version.

**3.2 Dynamic Data Consistency**
- Cached API data (e.g., user followers count) must refresh regularly.
- Techniques:
  - **TTL (Time-To-Live):** Define expiration times (e.g., 60s).
  - **Active Refresh:** Server pushes updates via WebSocket/SSE.
  - **ETag Validation:** Verify if the resource changed before serving cached data.

**3.3 CDN Invalidation Mechanisms**
- **Content Hashing:** Preferred automatic invalidation approach.
- **Purge API:** Use CDN’s REST API to manually invalidate outdated files.
- **Short TTL:** Apply shorter cache times for critical or fast-changing resources.

**3.4 Multi-Tab and Multi-Device Consistency**
- Use `BroadcastChannel` or `StorageEvent` for tab-to-tab synchronization.
  ```js
  const channel = new BroadcastChannel('cache-sync');
  channel.postMessage('invalidate-user-cache');
  ```
- For mobile/web parity, always refresh key resources on app start.

---

#### 4. Security and Edge Considerations

**4.1 Private and Authenticated Data**
- Mark sensitive responses with:
  ```
  Cache-Control: private, no-store
  ```
- Prevent CDN or shared caches from storing user-specific data.

**4.2 Cache Size and Eviction**
- Browsers impose local cache limits (usually a few hundred MB per domain).
- Use **LRU (Least Recently Used)** to manage custom caches efficiently.

**4.3 CDN Failover and Monitoring**
- Track CDN hit ratio, latency, and error rates.
- Implement automatic origin fallback if CDN edge nodes fail.

**4.4 Data Integrity and Validation**
- Use **Subresource Integrity (SRI)** to protect against CDN tampering.
  ```html
  <script src="main.js" integrity="sha384-abc123" crossorigin="anonymous"></script>
  ```

---

#### 5. Cache Architecture Patterns and Consistency Models

**5.1 Common Caching Models**
- **Cache-Aside:** Check cache first; if miss, fetch from origin and populate cache.
- **Write-Through:** Update cache at the same time as writing to source.
- **Write-Back:** Write to cache first, then asynchronously update origin.

**5.2 Consistency Levels**
- **Strong Consistency:** Always up-to-date but costly (frequent validation).
- **Eventual Consistency:** Allows short-term staleness; caches synchronize over time (used by CDNs and browsers).

**5.3 Cache Invalidation Strategies**
- **Write-Invalidate:** Explicitly invalidate affected cache on updates.
- **Write-Update:** Update cache immediately after a data change.
- **TTL Expiry:** Let cached data expire naturally after a defined lifetime.

> Front-end systems commonly adopt TTL-based caching (e.g., React Query’s `stale-while-revalidate` pattern) for simplicity and acceptable freshness.

---

#### 6. Best Practices and Observability

1. **Tailor Caching by Data Type**  
   - Static resources → Long-lived CDN cache + versioned URLs.  
   - Dynamic APIs → Shorter TTLs based on data sensitivity.  
2. **Monitor Performance and Cache Efficiency**  
   - Track CDN hit ratios, origin fetch rates, and HTTP 304 responses.  
   - Low hit ratios often indicate poor versioning or cache-busting issues.  
3. **Avoid Over-Caching**  
   - Ensure deployment scripts handle cache invalidation automatically.  
   - Provide manual refresh options (e.g., a “force reload” button).  
4. **Backend and Frontend Coordination**  
   - Agree on cache headers, TTLs, and invalidation rules.  
   - Allow clients to trigger force-refresh requests when necessary.  
5. **Continuous Optimization**  
   - Use RUM (Real User Monitoring) to track real-world latency and cache effectiveness.  
   - Adjust CDN routing and TTLs dynamically based on analytics.

---

### Summary

A well-designed front-end network and caching system should achieve:

- **High Performance:** CDN edge delivery + local cache reuse.
- **High Consistency:** Versioned assets + ETag validation.
- **High Reliability:** Multi-CDN failover + SRI verification.

> For TikTok-scale systems, the goal is balance — maximize performance via aggressive caching, while ensuring users always see fresh, correct data through intelligent invalidation and version management.
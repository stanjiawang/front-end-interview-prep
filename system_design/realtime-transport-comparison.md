# Realtime Transport on the Web — Comparative Analysis

> A practical, engineering-focused comparison of **Polling**, **Long Polling**, **SSE**, **WebSocket**, **WebRTC DataChannel**, **WebTransport (HTTP/3/QUIC)**, plus common **framework layers** (Socket.IO, SignalR, GraphQL Subscriptions, gRPC-Web streams). Includes scenarios, trade-offs, and code sketches (JS/TS).

---

## 0) TL;DR Decision Matrix

| Use Case | Best First Choice | Good Alternatives | Rationale |
|---|---|---|---|
| **Chat / Collaboration (bi‑directional, low latency)** | **WebSocket** | WebRTC DC (p2p), WebTransport (when available) | Full duplex, low overhead, widespread support; easy server fanout |
| **News Feed / Notifications (mostly server→client)** | **SSE** | Long polling (fallback), WebSocket (if already used elsewhere) | One‑way push, lightweight; auto‑reconnect; great for “read‑mostly” |
| **Live Telemetry / Logs Streaming** | **SSE** | WebSocket | Text/event-stream fits; flow control and incremental rendering |
| **Media/Data P2P (file send, multiplayer p2p)** | **WebRTC DataChannel** | WebSocket relay | NAT traversal, SCTP/QUIC‑like reliability modes; no relay cost if p2p works |
| **Ultra‑low latency / QUIC features / unreliable datagrams** | **WebTransport** | WebSocket | HTTP/3/QUIC streams & datagrams, modern congestion control (availability varies) |
| **IoT over web clients** | **WebSocket** | SSE | MQTT-over-WebSocket common; binary frames; server brokers |
| **Purely public SEO pages** | **SSE** | Long polling | Simpler infra, fits CDN/proxies better |
| **When infra already standardizes on a framework** | **Socket.IO / SignalR** | Native WS/SSE | Auto‑fallbacks, rooms, reconnection, auth helpers |

> Heuristic: **Start simple** (SSE for one‑way, WS for two‑way). Add **fallbacks** for enterprise proxies. Consider **WebTransport** experimentally for next‑gen needs.

---

## 1) Core Protocols Overview

### 1.1 Polling
Periodic `fetch()` to ask “anything new?”

**Pros:** simplest, works everywhere; easy caching; no stateful connections.  
**Cons:** wasteful (empty responses), higher latency (poll interval), handshake overhead.  
**When:** very low update frequency, prototypes, or as **last‑resort fallback**.

**Code (TS)**
```ts
async function poll(url: string, intervalMs = 5000, onData: (d: any)=>void) {
  let stop = false;
  while (!stop) {
    try {
      const r = await fetch(url, { cache: "no-store" });
      if (r.ok) onData(await r.json());
    } catch {}
    await new Promise(r => setTimeout(r, intervalMs));
  }
}
```

---

### 1.2 Long Polling
Client issues a request that **hangs** until server has data; returns immediately when data available, then client re‑issues.

**Pros:** lower latency than periodic polling; still HTTP‑friendly; works through most proxies.  
**Cons:** each response requires new request; many **hanging** requests on server; still not duplex.  
**When:** you need near‑realtime but can’t use WS/SSE (restrictive networks).

**Code**
```ts
async function longPoll(url: string, onData: (d: any)=>void) {
  for (;;) {
    const ac = new AbortController();
    try {
      const res = await fetch(url, { signal: ac.signal });
      if (res.ok) onData(await res.json());
    } catch {}
    // immediately reissue
  }
}
```

---

### 1.3 Server‑Sent Events (SSE)
HTTP/1.1 persistent connection, MIME `text/event-stream`. **Server→Client only** with automatic reconnection and **event IDs** for resume.

**Pros:** tiny overhead; easy to stream text; native reconnection (`EventSource`); great for **read‑mostly** flows.  
**Cons:** **one‑way**; text only; some proxies time out; fewer enterprise features than WS frameworks.  
**When:** feeds, notifications, logs, metrics, progress streams.

**Code**
```ts
const es = new EventSource("/events");
es.onmessage = (e) => console.log("data", e.data);
es.addEventListener("custom", (e) => { /* e.data */ });
es.onerror = () => { /* auto-reconnect by browser */ };
```

---

### 1.4 WebSocket (WS)
HTTP upgrade to a **full‑duplex** TCP‑based channel. Binary or text frames; app‑level protocol up to you.

**Pros:** **bi‑directional**, low latency; binary support; broad ecosystem; rooms/broadcast via libs.  
**Cons:** Requires stateful infra and WS‑aware load balancers; custom reconnection/heartbeats unless using a lib.  
**When:** chat, collaborative editing, game lobbies, interactive dashboards, generic event bus to clients.

**Code**
```ts
const ws = new WebSocket("wss://example.com/ws");
ws.onopen = () => ws.send(JSON.stringify({ type: "hello" }));
ws.onmessage = (e) => console.log(JSON.parse(e.data));
// heartbeat
setInterval(() => ws.readyState === ws.OPEN && ws.send('{"type":"ping"}'), 30000);
```

---

### 1.5 WebRTC DataChannel
Peer‑to‑peer, negotiated via SDP/ICE over a signaling channel (often WS). SCTP over DTLS; **ordered/unordered, reliable/partial‑reliable** modes.

**Pros:** p2p (cuts relay costs, lower latency), flexible reliability; ideal for real‑time collab and file/share between peers.  
**Cons:** complicated signaling, NAT traversal may fail → need TURN relay; not ideal for many‑to‑one server fanout.  
**When:** p2p data, multiplayer, large file share between participants, co‑editing cursors.

**Code (very simplified)**
```ts
const pc = new RTCPeerConnection();
const dc = pc.createDataChannel("chat", { ordered: true });
dc.onmessage = (e) => console.log(e.data);
// exchange SDP via signaling (e.g., WS) then setLocal/RemoteDescription, addIceCandidate...
```

---

### 1.6 WebTransport (HTTP/3 / QUIC)
Modern web API on top of QUIC: **unidirectional/bidirectional streams + unreliable datagrams**. Designed to replace some WS/WebRTC DC use cases with better congestion control.

**Pros:** multiplexed streams, head‑of‑line blocking avoidance, unreliable datagrams for ultra‑low latency.  
**Cons:** **Availability varies** (browser/server), TLS/ALPN/HTTP/3 infra needed; server stacks still maturing.  
**When:** future‑facing apps needing QUIC features (gaming, high‑freq telemetry, partial reliability).

**Code (experimental)**
```ts
const wt = new WebTransport("https://example.com/transport");
await wt.ready;
const stream = await wt.createBidirectionalStream();
const writer = stream.writable.getWriter();
writer.write(new TextEncoder().encode("hello"));
```

---

## 2) Framework / Abstraction Layers

### 2.1 Socket.IO
Built atop WS + fallbacks (long polling). Rooms, acks, reconnection, namespaces, sticky sessions.

**Pros:** batteries included, works across flaky networks; easy rooms/broadcast.  
**Cons:** extra framing; not a raw WS; server must be socket.io‑aware.  
**When:** product needs robust realtime quickly with minimal plumbing.

### 2.2 SignalR (ASP.NET)
Similar goals to Socket.IO for .NET ecosystems (WS/SSE/LP fallbacks).

### 2.3 GraphQL Subscriptions
Typically implemented over **WebSocket** (graphql‑ws, subscriptions‑transport‑ws). One channel, many ops with IDs.

**Pros:** typed schema; integrates with GraphQL clients; multiplexed ops.  
**Cons:** needs GraphQL server + WS infra.  
**When:** GraphQL‑centric stacks needing realtime results.

### 2.4 gRPC‑Web (streams)
Unary + server‑streaming over HTTP/1.1 or HTTP/2 with proxies.

**Pros:** strong contracts, codegen; server‑streaming fits telemetry.  
**Cons:** browsers don’t support raw gRPC; need Envoy or grpc‑web proxy; no client‑streaming in browser.  
**When:** org standardized on gRPC backend; streaming telemetry to web clients.

### 2.5 MQTT over WebSocket
Pub/Sub semantics; widely used in IoT.

**Pros:** topic‑based, light headers; retained messages, QoS.  
**Cons:** need broker; topic ACLs; adds protocol layer.  
**When:** IoT dashboards, device fleets.

---

## 3) Comparison Table (Capabilities)

| Capability | Polling | Long Poll | SSE | WebSocket | WebRTC DC | WebTransport |
|---|---:|---:|---:|---:|---:|---:|
| Direction | C→S | C→S (S→C via response) | S→C | **C↔S** | **P2P** | **C↔S** |
| Latency | ★ | ★★ | ★★★ | **★★★★** | **★★★★** | **★★★★** |
| Binary | ✔ (fetch arrayBuffer) | ✔ | ✖ (text) | **✔** | **✔** | **✔** |
| Multiplexing | ✖ | ✖ | ✖ | (by app) | ✔ | **✔ (streams)** |
| Unreliable datagrams | ✖ | ✖ | ✖ | ✖ | ✔ (partial reliable) | **✔** |
| Proxies/CDN friendly | **✔** | **✔** | ◐ | ◐ | ◐ | ◐ |
| Server complexity | ★ | ★★ | ★★ | **★★★** | **★★★★** | **★★★★** |
| Browser support | **★★★★★** | **★★★★★** | **★★★★** | **★★★★★** | **★★★★** | ◐ (evolving) |

(★ more stars = better; ◐ = depends on environment.)

---

## 4) Architecture Patterns

### 4.1 “Simple feed + notifications”
- **SSE** for new‑item hints and counters.  
- **REST** for pagination (`cursor`‑based).  
- **Long polling** fallback for corporate proxies.

### 4.2 “Chat + feed + presence” (collaboration suite)
- **WebSocket** single connection carrying: chat messages, presence, feed updates, badges.  
- **REST** for history/backfill and uploads.  
- Optional: SSE for public announcements page.

### 4.3 “P2P collaboration (Figma‑style cursors)”
- **WebRTC DC** for peer cursor/ops when possible; fallback to **WS relay**.  
- Server orchestrates room membership & history snapshot.

### 4.4 “Next‑gen realtime (gaming/telemetry)”
- **WebTransport** for datagrams (unreliable) + streams (reliable), with fallback to **WS**.

---

## 5) Reliability, Ordering, Backpressure

- **Ordering**: WS is ordered; WebRTC DC can be unordered/partial‑reliable; WebTransport supports independent streams to avoid head‑of‑line blocking.  
- **Backpressure**: Streams APIs (WebTransport) expose backpressure; WS needs app‑level flow control (rate limit, buffer sizes).  
- **Reconnect**: SSE auto‑reconnect with `Last-Event-ID`; WS use exponential backoff + heartbeats; frameworks help.  
- **Idempotence**: include `eventId`/`clientNonce` to dedupe on reconnect.  
- **Security**: all via TLS; apply **auth tokens** (short‑lived), **CSP**, and sanitize payloads before rendering.

---

## 6) Example: Unified Realtime Client (WS primary, SSE fallback)

```ts
type Handler = (evt: any) => void;

export class RealtimeClient {
  private ws?: WebSocket;
  private es?: EventSource;
  private handlers = new Map<string, Set<Handler>>();

  on(type: string, h: Handler) {
    if (!this.handlers.has(type)) this.handlers.set(type, new Set());
    this.handlers.get(type)!.add(h);
  }

  private emit(type: string, evt: any) {
    this.handlers.get(type)?.forEach(h => h(evt));
  }

  connect() {
    try {
      this.ws = new WebSocket("wss://example.com/rt");
      this.ws.onmessage = (e) => this.emit(JSON.parse(e.data).type, JSON.parse(e.data));
      this.ws.onclose = () => this.fallbackToSSE();
    } catch {
      this.fallbackToSSE();
    }
  }

  private fallbackToSSE() {
    this.es = new EventSource("/events");
    this.es.onmessage = (e) => this.emit("message", { type: "message", data: e.data });
  }

  send(data: any) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    } else {
      // enqueue or POST as fallback
      fetch("/api/commands", { method: "POST", body: JSON.stringify(data) });
    }
  }
}
```

---

## 7) Cost & Ops Considerations

- **Stateful connections** (WS/SSE) need sticky sessions or shared pub/sub (Redis, NATS, Kafka) behind stateless edges.  
- **Autoscaling**: connection count as a metric; shard by tenant/region.  
- **Enterprise networks**: some proxies terminate idle connections—implement ping/keepalive and **fallbacks**.  
- **Battery & mobile**: avoid frequent wakeups; prefer server push (SSE/WS) over short‑interval polling.

---

## 8) Choosing Guide (Flow)

```
Need client→server low-latency? ── yes ──► WebSocket
  │
  no
  │
Server→client only? ───────────── yes ──► SSE
  │                                  └─► Long Poll (if SSE blocked)
  no
  │
P2P between browsers? ─────────── yes ──► WebRTC DataChannel
  │
Need QUIC features? ───────────── yes ──► WebTransport (with WS fallback)
  │
Default: start with SSE/WS based on directionality
```

---

## 9) Quick Reference (When to Use What)

- **SSE**: server‑only push (feeds, counters, logs).  
- **WebSocket**: bidirectional (chat, presence, dashboards).  
- **WebRTC DC**: p2p data (files, multiplayer), with TURN fallback.  
- **WebTransport**: QUIC streams/datagrams (advanced, watch support).  
- **Long polling**: strict networks or legacy fallback.  
- **Socket.IO / SignalR**: productivity boost with auto‑fallbacks.  
- **GraphQL Subscriptions / gRPC‑Web**: if your org already centers on those stacks.

---

*Appendix tip:* In interviews, explain **why your first choice fits the traffic pattern**, and mention **fallbacks** + **enterprise proxy constraints**. Name **backpressure, ordering, idempotence**, and **security** (auth, XSS) for bonus points.

# 22 — Analytics & User Behavior Tracking (学习级详细版 / Full Learning Guide)

---

## 🧠 Overview 概述

**Analytics systems** help teams understand *what users do, where they struggle, and how features perform*.  
This chapter explains how front-end analytics pipelines collect, process, and visualize behavioral data responsibly.

> 💡 中文：前端分析系统通过事件数据帮助团队理解用户行为、功能表现与转化路径。

---

## 1. Web Analytics Fundamentals（网页分析基础）

### 1.1 Purpose
- Track **user interactions** (clicks, scrolls, navigation).  
- Measure **business metrics** (conversion, engagement).  
- Support **product experiments** (A/B testing).

### 1.2 Types of Analytics
| Type | Description | Example Tools |
|------|--------------|----------------|
| **Page Analytics** | Page views, time-on-page | Google Analytics, Mixpanel |
| **Event Analytics** | Button clicks, feature usage | Amplitude, Segment |
| **Funnel Analytics** | Conversion flow | Heap, PostHog |
| **Custom BI** | SQL-based dashboards | Snowflake + Looker |

**Data Flow Diagram**
```
Browser → SDK (collect) → Collector API → Stream (Kafka) → Data Warehouse → Dashboard
```

> 💡 中文：分析系统核心链路包括采集、传输、存储与可视化。

---

## 2. Instrumentation & Event Architecture（埋点与事件架构）

### 2.1 Tracking Models
| Model | Description | Pros / Cons |
|--------|-------------|-------------|
| **Manual Tracking** | Developer inserts `track()` manually | Precise but laborious |
| **Auto Tracking** | DOM-based auto capture | Fast but noisy |
| **Hybrid** | Manual + auto | Best practice |

### 2.2 Event Schema Design
| Field | Example | Description |
|--------|----------|-------------|
| `event_name` | "button_click" | Action name |
| `category` | "checkout" | Feature group |
| `user_id` | "uid_123" | Anonymized ID |
| `timestamp` | ISO string | Event time |
| `properties` | { "label": "Buy" } | Context |

**Example JSON:**
```json
{
  "event_name": "product_view",
  "user_id": "anon_123",
  "timestamp": "2025-10-17T09:00:00Z",
  "properties": { "product_id": "P001", "category": "shoes" }
}
```

> 💡 中文：事件 Schema 需统一定义字段，确保数据可聚合与兼容。

---

## 3. Data Pipeline & SDK Design（数据管线与 SDK 设计）

### 3.1 Core SDK Structure
```js
class Tracker {
  constructor(endpoint) {
    this.endpoint = endpoint;
    this.queue = [];
  }
  track(eventName, properties = {}) {
    const data = {
      event_name: eventName,
      properties,
      timestamp: new Date().toISOString(),
    };
    this.queue.push(data);
    this.flush();
  }
  flush() {
    if (navigator.sendBeacon) {
      navigator.sendBeacon(this.endpoint, JSON.stringify(this.queue));
    } else {
      fetch(this.endpoint, { method: "POST", body: JSON.stringify(this.queue) });
    }
    this.queue = [];
  }
}

const tracker = new Tracker("/collect");
tracker.track("page_view", { url: location.href });
```

> 💡 中文：SDK 负责采集并批量上报事件数据，常用 sendBeacon 实现异步非阻塞传输。

### 3.2 Batch & Retry Strategy
- Buffer events until 10 items or 5 seconds elapsed.  
- Retry failed uploads (exponential backoff).  
- Compress payload using GZIP.  

### 3.3 Real-Time vs Batch Upload
| Mode | Advantage | Example |
|------|------------|----------|
| **Real-time** | Instant updates | User session analytics |
| **Batch** | Reduced network load | Periodic telemetry |

---

## 4. Privacy & Compliance（隐私与合规）

### 4.1 Key Regulations
| Law | Region | Key Point |
|------|--------|-----------|
| **GDPR** | EU | Consent, right to forget |
| **CCPA** | US (California) | Opt-out, transparency |
| **PDPA** | Singapore | Explicit consent |

### 4.2 Anonymization Techniques
| Method | Description |
|--------|-------------|
| Hashing | Replace identifiers (SHA-256) |
| Masking | Hide sensitive fields |
| Sampling | Collect only partial users |

### 4.3 Consent Management
```js
if (getCookie("consent") === "yes") tracker.track("page_view");
```

> 💡 中文：采集前需获得用户同意；所有标识符应脱敏、哈希或采样。

---

## 5. Visualization & A/B Testing（可视化与实验分析）

### 5.1 Dashboard Tools
| Tool | Usage |
|------|--------|
| **Mixpanel / Amplitude** | Behavioral analytics |
| **Looker / Tableau** | BI dashboards |
| **Grafana / Metabase** | Open-source visualization |

**Example Metrics:**
- DAU / MAU ratio  
- Feature adoption rate  
- Conversion funnel (add to cart → checkout → purchase)

### 5.2 A/B Testing Workflow
1. Split users randomly (50/50).  
2. Assign variant (A or B).  
3. Track conversion events.  
4. Analyze statistically (p-value, lift).

**Implementation:**
```js
const variant = Math.random() < 0.5 ? "A" : "B";
tracker.track("experiment_assign", { exp: "new_checkout", variant });
```

> 💡 中文：A/B 实验通过事件追踪对比用户行为差异，以验证产品假设。

---

## 6. Interview-Oriented Section（面试导向）

### 6.1 Key Question
**“How would you design a scalable analytics system for a front-end app?”**

**Answer Outline:**
1. Define event schema & SDK interface.  
2. Collect via beacon + batching.  
3. Stream events to message broker (Kafka).  
4. Store in data warehouse.  
5. Visualize & analyze metrics.  
6. Apply GDPR-compliant anonymization.

### 6.2 Trade-off Table
| Approach | Pros | Cons |
|-----------|------|------|
| Real-time tracking | Instant updates | High bandwidth |
| Batch upload | Efficient | Slight delay |
| Manual tagging | Control | Dev overhead |
| Auto-capture | Fast rollout | Noise, data inflation |

---

## 🧩 Summary 总结

| Layer | Focus | Tools |
|-------|--------|-------|
| Collection | Event SDK | Custom, Segment |
| Transport | Stream / Queue | Kafka, Kinesis |
| Storage | Warehouse | Snowflake, BigQuery |
| Visualization | Dashboard | Looker, Grafana |
| Compliance | Privacy & consent | Cookie banner, hashing |

> 💡 中文总结：用户行为分析系统通过事件采集、管线传输与可视化实现数据驱动决策。其关键在于数据一致性与隐私合规。

---

📘 **Next Chapter → 23. Modern Web Graphics & Canvas Rendering**

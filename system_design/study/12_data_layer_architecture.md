# 12 — Frontend Data Layer Design (Part 1 of 4)
*(Sections 1–2 | Data Layer Fundamentals & Architecture — Enhanced Version)*

---

## 1. Understanding the Frontend Data Layer

A **Data Layer** abstracts the way front-end apps fetch, cache, synchronize, and mutate data across components and servers.

> 💡 **中文解释：** 数据层（Data Layer）是前端应用中用于抽象「数据获取、缓存、同步与更新」的体系，让 UI 与数据源解耦。

### 1.1 Why Do We Need a Data Layer?

| Problem | Without Data Layer | With Data Layer |
|----------|--------------------|-----------------|
| **Redundant Requests** | Multiple components call same API | Shared cache & deduplication |
| **Inconsistent Data** | Different states in components | Central data coordination |
| **Complex Error Handling** | Scattered try/catch blocks | Unified retry/revalidation |
| **Poor Offline UX** | No data when offline | Stale cache & optimistic UI |

> 💡 **中文解释：** 数据层的核心价值在于统一管理远程数据的生命周期，从而提高性能与一致性。

---

## 2. Data Flow Architecture Overview

A well-structured front-end data system separates **UI**, **State**, and **Data Operations** layers.

```
+------------------------------+
|        UI Components         |
|  (React / Next.js Pages)     |
+-------------▲----------------+
              |
              ▼
+------------------------------+
|     State Management Layer    |
|  (Zustand / Redux / Context) |
+-------------▲----------------+
              |
              ▼
+------------------------------+
|        Data Layer            |
| (React Query / SWR / Apollo) |
+-------------▲----------------+
              |
              ▼
+------------------------------+
|         APIs / GraphQL       |
|   (REST, RPC, Streaming)     |
+------------------------------+
```

> 💡 **中文解释：** UI 通过状态层访问数据层；数据层与后端 API 交互并维护缓存。

---

## 3. REST vs GraphQL vs RPC — Conceptual Overview

| Protocol | Query Model | Caching | Strength | Limitation |
|-----------|--------------|----------|------------|-------------|
| **REST** | Resource-based (GET/POST/PUT/DELETE) | URL-based caching | Simplicity, ubiquity | Over-fetching, multiple requests |
| **GraphQL** | Declarative query for data graph | Field-level normalization | Flexible, fewer requests | Complex caching, query cost |
| **RPC (Remote Procedure Call)** | Function-based endpoints | Per-method caching | Performance, binary protocols (gRPC) | Tightly coupled with backend schema |

> 💡 **中文解释：** REST 简单易用；GraphQL 灵活且减少请求；RPC 性能高但强依赖服务定义。

**Example — REST:**
```js
// Fetch REST endpoint
const res = await fetch("/api/products");
const data = await res.json();
```

**Example — GraphQL:**
```js
const query = `
  query GetProducts {
    products {
      id
      name
      price
    }
  }
`;
const res = await fetch("/graphql", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query }),
});
const { data } = await res.json();
```

**Example — gRPC (pseudo client stub):**
```ts
const client = new ProductService("https://grpc.example.com");
const response = await client.GetProduct({ id: 1 });
```


# 12 — Frontend Data Layer Design (Part 2 of 4)
*(Sections 3–4 | React Data Layer: SWR & React Query — Enhanced Version)*

---

## 4. React Data Layer Frameworks

### 4.1 SWR — Stale-While-Revalidate

SWR follows the **HTTP caching strategy** “stale-while-revalidate”: serve stale data first, then revalidate in background.

```js
import useSWR from "swr";
const fetcher = url => fetch(url).then(res => res.json());

function Profile() {
  const { data, error, isLoading } = useSWR("/api/user", fetcher);
  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error!</p>;
  return <h1>Hello {data.name}</h1>;
}
```

> 💡 **中文解释：** SWR 会先返回缓存数据（stale），同时后台触发请求刷新（revalidate），从而兼顾性能与实时性。

**Flow:**
```
Cache hit → UI render → background re-fetch → cache update → re-render
```

**Key Options:**
```js
useSWR(key, fetcher, {
  revalidateOnFocus: true,
  dedupingInterval: 2000,
  refreshInterval: 0
});
```

---

### 4.2 React Query — Caching + Invalidation

React Query focuses on **server state** with intelligent caching and invalidation.

```js
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

function Todos() {
  const queryClient = useQueryClient();
  const { data: todos } = useQuery(["todos"], () => fetch("/api/todos").then(r => r.json()));

  const mutation = useMutation(
    newTodo => fetch("/api/todos", { method: "POST", body: JSON.stringify(newTodo) }),
    { onSuccess: () => queryClient.invalidateQueries(["todos"]) }
  );

  return (
    <>
      {todos?.map(t => <p key={t.id}>{t.text}</p>)}
      <button onClick={() => mutation.mutate({ text: "New Task" })}>Add</button>
    </>
  );
}
```

> 💡 **中文解释：** React Query 通过 `invalidateQueries` 实现精确缓存失效与自动刷新，是复杂异步场景的首选。

**Cache & Staleness:**
```js
useQuery(["users"], fetchUsers, {
  staleTime: 60_000,     // 1 min considered fresh
  cacheTime: 5 * 60_000, // GC after 5 min unused
  retry: 2
});
```

**Prefetch & Hydration (Next.js):**
```ts
// server
await queryClient.prefetchQuery(["post", id], () => fetchPost(id));
const dehydrated = dehydrate(queryClient);

// client
const { data } = useQuery(["post", id], () => fetchPost(id), { initialData: dehydrated });
```

---

### 4.3 Comparison: SWR vs React Query

| Feature | SWR | React Query |
|----------|-----|-------------|
| API Simplicity | Minimal | Rich API |
| Mutation Handling | Manual | Built-in (useMutation) |
| DevTools | No official | ✅ Excellent DevTools |
| Prefetch / Hydration | Manual | First-class (SSR/SSG) |
| Window Refocus Revalidate | Yes | Yes |

> 💡 **中文解释：** SWR 轻量，适合静态内容；React Query 功能全面，适合复杂异步交互。


# 12 — Frontend Data Layer Design (Part 3 of 4)
*(Sections 5–6 | GraphQL Client Architecture — Apollo & Relay)*

---

## 5. GraphQL Client Architecture

GraphQL clients maintain a **normalized in-memory cache** and track dependencies at field level.

> 💡 **中文解释：** GraphQL 客户端（如 Apollo、Relay）通过字段级缓存管理数据依赖，实现最小化数据更新。

### 5.1 Apollo Client Example

```js
import { ApolloClient, InMemoryCache, gql } from "@apollo/client";

const client = new ApolloClient({
  uri: "/graphql",
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          product: {
            keyArgs: ["id"]
          }
        }
      }
    }
  }),
});

client
  .query({
    query: gql`
      query GetUser {
        user(id: 1) {
          id
          name
          email
        }
      }
    `,
  })
  .then(result => console.log(result.data));
```

**Cache Normalization:**
```
{
  "User:1": { id: 1, name: "Alice" },
  "User:2": { id: 2, name: "Bob" }
}
```

**Mutation & Cache Update:**
```js
client.mutate({
  mutation: gql`mutation Rename($id: ID!, $name: String!) {
    renameUser(id: $id, name: $name) { id name }
  }`,
  variables: { id: "1", name: "Alicia" },
  update(cache, { data }) {
    cache.modify({
      id: cache.identify({ __typename: "User", id: "1" }),
      fields: { name() { return data.renameUser.name; } }
    });
  }
});
```

> 💡 **中文解释：** Apollo 通过 `cache.modify` 精准更新变更对象，避免全量 refetch。

---

### 5.2 Relay — Declarative Data Dependencies

Relay enforces **colocation**: components declare exactly what data they need.

```js
graphql`
  fragment UserInfo_user on User {
    id
    name
  }
`;

function UserInfo({ user }) {
  return <p>{user.name}</p>;
}
```

**Pagination Container:**
```js
const connectionConfig = {
  direction: 'forward',
  getConnectionFromProps(props) {
    return props.user && props.user.friends;
  },
  getVariables(props, { count, cursor }) {
    return { id: props.user.id, count, cursor };
  },
  query: graphql`
    query FriendsPaginationQuery($id: ID!, $count: Int!, $cursor: String) {
      user(id: $id) { friends(first: $count, after: $cursor) @connection(key: "Friends_friends") { edges { node { id name } } } }
    }`
};
```

> 💡 **中文解释：** Relay 通过片段（fragment）与连接（connection）模式在大型项目中提供高可维护性与精准缓存。


# 12 — Frontend Data Layer Design (Part 4 of 4)
*(Sections 7–8 | Consistency, Offline, and Interview Q&A — Enhanced)*

---

## 6. Data Consistency & Offline Sync

### 6.1 Optimistic UI Updates (React Query)

```js
const mutation = useMutation(updateTodo, {
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries(["todos"]);
    const previousTodos = queryClient.getQueryData(["todos"]);
    queryClient.setQueryData(["todos"], old => [...(old ?? []), newTodo]);
    return { previousTodos };
  },
  onError: (_err, _newTodo, context) => {
    if (context?.previousTodos) queryClient.setQueryData(["todos"], context.previousTodos);
  },
  onSettled: () => queryClient.invalidateQueries(["todos"])
});
```

> 💡 **中文解释：** 乐观更新在网络未完成时先更新 UI，失败再回滚，提高交互流畅度。

### 6.2 Offline-First Pattern (Service Worker + IndexedDB)

| Strategy | Description |
|-----------|--------------|
| **IndexedDB Cache** | Persist query results for offline |
| **Background Sync** | Resync when connection restored |
| **Workbox** | Route API calls via SW and cache |
| **Conflict Resolution** | Merge local & remote changes |

**Service Worker Example:**
```js
self.addEventListener("fetch", (event) => {
  if (event.request.url.includes("/api/")) {
    event.respondWith(
      caches.open("data-cache").then(cache =>
        fetch(event.request)
          .then(resp => {
            cache.put(event.request.url, resp.clone());
            return resp;
          })
          .catch(() => cache.match(event.request))
      )
    );
  }
});
```

> 💡 **中文解释：** 通过 SW 缓存数据请求可保证弱网/离线时的可用性，网络恢复后再对账同步。

---

## 7. Interview-Oriented Summary

### 7.1 “How would you design a data layer for a large-scale React app?”

**Answer Framework:**
1. **Server State:** React Query/SWR with staleTime & cacheTime.  
2. **Client State:** Zustand/Redux for UI domain logic.  
3. **GraphQL:** Apollo/Relay with normalized caching for complex graphs.  
4. **Consistency:** Optimistic UI, background revalidation, conflict resolution.  
5. **Observability:** Cache hit/miss, latency, retry rates, error budgets.

### 7.2 “What’s the difference between server state and client state?”

| Aspect | Client State | Server State |
|---------|---------------|--------------|
| Ownership | Local | Remote (API) |
| Update Model | Immediate | Network roundtrip |
| Persistence | In-memory | Persistent store |
| Tools | Zustand, Recoil | React Query, SWR |

### 7.3 Key Takeaways

1. Data layer = orchestration of **fetching, caching, revalidation**.  
2. REST, GraphQL, RPC each have strengths — choose per domain.  
3. Use optimistic UI + offline sync for resilience.  
4. Measure cache performance and invalidation cost.

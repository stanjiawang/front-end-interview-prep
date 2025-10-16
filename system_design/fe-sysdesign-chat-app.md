# Front-End System Design Deep Dive — Chat / Messaging (Front-End)

> A production-oriented blueprint for **real-time messaging UI**: cache patching via WebSocket, optimistic UI, offline outbox, read receipts, and virtualized message lists in **React + TypeScript**.

---

## 1. Requirements
- 1:1 & group channels; threads.
- Send text/emoji/files; edit/delete; reactions.
- Delivery states: sending/sent/delivered/read.
- Presence/typing; read receipts.
- History pagination; deep link to message.
- Offline compose & retry; reconnect recovery.

---

## 2. High-Level Architecture
```
App Shell
 ├─ ChannelList
 └─ ChannelView
     ├─ Header (title, members, search)
     ├─ MessagePane (virtualized)
     ├─ Composer (drafts, uploads, emoji)
     └─ ThreadDrawer
Global
 ├─ Query cache (RTK Query / React Query)
 ├─ WebSocketClient + EventBus
 ├─ Outbox (IndexedDB + Background Sync)
 ├─ UploadManager (chunked, retries)
 └─ Telemetry (send→ack latency, errors)
```

---

## 3. Data Model (Simplified)
```ts
export interface Message {
  id: string;
  channelId: string;
  senderId: string;
  body: string;
  createdAt: string;
  editedAt?: string;
  status?: "sending" | "sent" | "delivered" | "read" | "failed";
  clientNonce?: string;
  replyToId?: string;
}
```

---

## 4. Realtime & Cache Patching
- WebSocket events: `message.created`, `message.updated`, `reaction.added`, `read.receipt`, `typing`.
- Idempotent reducers: dedupe by `eventId`; last-writer-wins by server timestamp.
- Reconnect recovery: `GET /events?since=<ts>` to backfill.

**Reducer sketch**
```ts
function applyEvent(state: ChannelState, event: WsEvent) {
  if (state.applied.has(event.eventId)) return;
  switch (event.type) {
    case "message.created": insertInOrder(state.messages, event.message); break;
    case "message.updated": upsert(state.messages, event.message); break;
    case "read.receipt": state.readByUser.set(event.userId, event.messageId); break;
  }
  state.applied.add(event.eventId);
}
```

---

## 5. Optimistic Send & Outbox
- On send: create temp message `{id: "temp:"+nonce, status:"sending"}`.
- On ACK: replace by real id + `status:"sent"`; on error: mark `failed` with retry button.
- Outbox in IndexedDB + Background Sync flushes when online.

**RTK Query mutation (sketch)**
```ts
sendMessage: b.mutation<Message, { channelId: string; body: string; clientNonce: string }>({
  query: ({ channelId, ...payload }) => ({ url: `/channels/${channelId}/messages`, method: "POST", body: payload }),
  async onQueryStarted({ channelId, body, clientNonce }, { dispatch, queryFulfilled }) {
    const temp: Message = { id: `temp:${clientNonce}`, channelId, senderId: "me", body, createdAt: new Date().toISOString(), status: "sending", clientNonce };
    const patch = dispatch(api.util.updateQueryData("listMessages", { channelId }, (draft) => { draft.messages.push(temp); }));
    try { const { data } = await queryFulfilled;
      dispatch(api.util.updateQueryData("listMessages", { channelId }, (draft) => {
        const i = draft.messages.findIndex(m => m.clientNonce === clientNonce);
        if (i >= 0) draft.messages[i] = { ...data, status: "sent" };
      }));
    } catch { dispatch(api.util.updateQueryData("listMessages", { channelId }, (d) => { const i = d.messages.findIndex(m => m.clientNonce === clientNonce); if (i >= 0) d.messages[i].status = "failed"; })); }
    finally { patch.undo?.(); }
  }
})
```

---

## 6. Virtualized Message List
- Use `react-window`/`react-virtualized` with dynamic row heights.
- Anchor-based prepend (for older history) to avoid scroll jumps.
- Image `onload` → debounced remeasure; reserve height to minimize CLS.

---

## 7. A11y & Security
- `role="list"`/`listitem"`; `aria-live="polite"` for new messages.
- Keyboard: jump to latest, navigate by message, focus composer after send.
- Sanitize markdown on server; escape at render; CSP; never trust HTML from users.

---

## 8. Testing
- Unit: reducers, cache patching idempotence.
- Integration: send→ACK flow; reconnect replay.
- E2E: offline outbox; large channel scroll.

---

## 9. Interview Soundbite
> “WebSocket events patch the query cache idempotently, optimistic UI bridges latency, and IndexedDB powers an outbox. Virtualization sustains smooth scrolling at scale.”

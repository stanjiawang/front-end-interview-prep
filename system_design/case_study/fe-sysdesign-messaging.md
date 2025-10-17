# Front‑End System Design — Messaging for a Collaboration Platform

> Scope: Design the **front end** for the messaging module of a collaboration platform (with meetings + messaging). Focus on system‑design thinking, trade‑offs, performance, accessibility, testing, security, and operations. Answers are framed like interview responses with pragmatic implementation detail (React + TypeScript bias, but framework‑agnostic).

---

## 0) Executive Summary (Elevator Pitch)

We’re building a **real‑time, resilient, accessible, and scalable** messaging UI that supports **1:1 DMs and group channels**, **threads**, **rich media**, **reactions**, **mentions**, **read receipts**, **presence/typing**, **search**, **message retention/pinning**, and **meeting hand‑offs** (jump from chat to meeting). The front end emphasizes **performance** (virtualized lists, code‑splitting, image optimization), **data consistency** (RTK Query/React Query cache + websocket events), **offline/spotty‑network UX** (service worker, optimistic UI, retry queues), and **operability** (telemetry, feature flags, error boundaries).

---

## 1) Clarifying Questions (What I’d ask in the interview)

1. **Traffic & scale**: Peak concurrent users per channel? Max channel size? Expected daily messages? Typical message length & attachment sizes?
2. **Message features**: Threads? Reactions? Mentions/hashtags? Editing/deleting? Quoting/reply‑to? Pinned messages? Read receipts (per‑message vs per‑channel)?
3. **Realtime**: WebSockets, SSE, or polling? Latency SLO (e.g., P95 < 500ms from server broadcast to UI render)?
4. **Offline**: Is full offline read/browse required? Offline compose + send on reconnect?
5. **Retention & compliance**: Legal holds, retention windows, eDiscovery, export? Message redaction? Deletion policies?
6. **Security**: E2EE vs TLS‑only? DLP, malware scanning for files? SSO/OIDC? CSP policy?
7. **Platforms**: Web only vs desktop (Electron) vs mobile web/PWA? Internationalization (RTL, CJK, emojis) and theming?
8. **Integration with Meetings**: Deep links from messages to recorded meetings? “Escalate to call” entry points?
9. **Search**: Server‑side search with pagination, filters (sender, date, has:link/file), highlight snippets?
10. **Accessibility**: WCAG 2.2 AA? Screen reader flow for virtualized list? Keyboard interaction model?

---

## 2) Functional & Non‑Functional Requirements

### Functional
- 1:1 and group channels, **threads**, **replies**, **reactions**, **mentions** (@user, @here, @channel).
- **Typing & presence**, **read receipts**, **message states** (sending, sent, delivered, read, failed).
- **Rich content**: text, emoji, code blocks, files/images, link preview, polls (extensible).
- **Search & filters** with result highlighting, jump‑to‑message + smooth scroll/anchor.
- **Moderation**: delete message, redact, freeze channel (role‑based).
- **Notifications**: per‑channel mute, mentions‑only, do‑not‑disturb, unread counts, toasts.
- **Meeting hand‑off**: “Start meeting” from channel; show live meeting badge.

### Non‑Functional
- **Performance**: P75 LCP < 2.5s on mid‑tier devices; smooth 60fps scroll; stable FPS with 100k+ messages via virtualization.
- **Reliability**: Graceful on disconnect/reconnect; **exactly‑once UI** for events (idempotent reducers).
- **Accessibility**: WCAG 2.2 AA, full keyboard navigation, screen reader compatible virtualization.
- **Security & Privacy**: XSS safe rendering, CSP, sanitized rich text, principle of least privilege.
- **Observability**: Web Vitals, error tracking, user journey traces, real‑time health indicators.
- **Maintainability**: Module boundaries, type‑safe APIs, well‑tested, feature‑flag‑driven rollout.

---

## 3) High‑Level Front‑End Architecture

```
+-------------------------------------------- App Shell --------------------------------------------+
|  Nav  |  ChannelSidebar  |                    MainArea (Router Outlet)                            |
|       |                  |  ----------------------------------------------------------------------------------
|       |                  |  ChannelView:                                                            
|       |                  |   ├── Header (channel title, members, search box, meeting badge)                       
|       |                  |   ├── MessagePane (VirtualizedList)  <--- subscribes to query + websocket stream
|       |                  |   │     ├── MessageItem (text, media, reactions, read receipts)           
|       |                  |   │     └── DateDividers / NewMessageAnchor                               
|       |                  |   ├── Composer (drafts, uploads, emoji, @mentions, slash commands)       
|       |                  |   └── ThreadDrawer (optional side panel)                                 
|       |                  |                                                                           
|       |                  |  Global:                                                                  
|       |                  |   ├── WebsocketClient / EventBus                                          
|       |                  |   ├── Data Fetching (RTK Query / React Query)                             
|       |                  |   ├── Cache & Normalized Store (messages, users, channels)                
|       |                  |   ├── Service Worker (offline, push, background sync)                     
|       |                  |   ├── UploadManager (chunked uploads, retries, progress)                  
|       |                  |   └── Telemetry (logger, metrics, feature flags)                          
+-----------------------------------------------------------------------------------------------------+
```

**Routing & State**  
- URL structure encodes location: `/c/:channelId`, `/c/:channelId/m/:messageId` (deep link), `/thread/:rootMessageId`.  
- **Normalized cache** (by `messageId`, `channelId`, `userId`).  
- **Query library** (RTK Query / React Query) for server cache + **websocket patching** to keep lists live.  
- **Event dedupe** by `eventId` & **monotonic clocks** (server timestamp authority) to prevent out‑of‑order UI.

---

## 4) Data Model (Front‑End Types)

```ts
export type ID = string;

export interface User {
  id: ID;
  displayName: string;
  avatarUrl?: string;
  presence: "online" | "away" | "dnd" | "offline";
}

export interface Channel {
  id: ID;
  name: string;
  type: "dm" | "group";
  memberIds: ID[];
  lastReadAt?: string; // ISO
}

export interface Reaction {
  emoji: string;
  userIds: ID[];
}

export interface Attachment {
  id: ID;
  kind: "image" | "file" | "link" | "audio" | "video";
  url: string;
  name?: string;
  sizeBytes?: number;
  previewUrl?: string;
  width?: number;
  height?: number;
}

export interface Message {
  id: ID;
  channelId: ID;
  senderId: ID;
  body: string;           // sanitized rich text (markdown-ish) from server
  createdAt: string;      // server time
  editedAt?: string;
  replyToId?: ID;         // thread root or parent
  reactions?: Reaction[];
  attachments?: Attachment[];
  status?: "sending" | "sent" | "delivered" | "read" | "failed";
  clientNonce?: string;   // for optimistic sends
}
```

---

## 5) Fetching, Realtime, and Caching Strategy

**Hybrid approach**:  
- **Initial load**: paginated REST/GraphQL queries (server‑side search index; return `messages[], nextCursor`).  
- **Realtime updates**: WebSocket stream of events `{type, payload, eventId, ts}`.  
- **Cache**: RTK Query/React Query as the **authoritative source**, **patched** by websocket events (insert/update by `messageId`, merge reactions, set delivery/read states).  
- **Optimistic UI** for sends/reactions; map `clientNonce → server messageId` upon ack; reconcile failures.  
- **Backfill** on reconnect with `sinceTs` to recover missed events.  
- **Image/file** via CDN with signed URLs; use `<img loading="lazy">`, responsive `srcset`, object‑fit.

**Pagination**:  
- **Bidirectional infinite scroll** (up for older, down for newer); maintain anchors to avoid scroll jumps.  
- Virtualized list (e.g., `react-window`/`react-virtualized`) with **dynamic row heights** and **sticky "New" divider**.  
- Preserve scroll position when new items are prepended/appended.

---

## 6) UI/UX Details That Interviewers Love

1. **Composer**: draft persistence per channel, keyboard shortcuts (Enter to send, Shift+Enter newline), upload via paste/drag, emoji picker, mention autocomplete, slash commands.  
2. **Message item**: collapsible quoted reply, copy link/ID, edit/delete, hover actions; show delivery/read ticks; inline code & fenced blocks with copy button.  
3. **Threading**: side drawer with count badge; clicking a reply in main list opens thread at that root.  
4. **Anchored “Jump to Latest / New Messages”** with smooth scroll and ARIA live region announcing jumps.  
5. **Presence/typing**: batched updates to avoid thrash (coalesce typing events within 1s).  
6. **Empty and error states** with recovery actions (retry, diagnostics link).

---

## 7) Performance Plan (Web Vitals + Runtime)

- **Code splitting**: split editor (rich‑text) and heavy pickers; lazy‑load thread drawer and search panel.  
- **Virtualization**: never render thousands of DOM nodes; dynamic size caching; window overscan tuned for perf.  
- **Memoization**: `React.memo`, `useMemo`, `useCallback`; carefully scoped to avoid prop‑drill re‑renders.  
- **Image perf**: responsive images, lazy loading, decode hints, prefetch avatars.  
- **Minimize layout shifts**: reserve heights (skeleton rows), use `content-visibility: auto`.  
- **Network**: HTTP/2 or HTTP/3, compression (Brotli), ETag/If‑None‑Match, long‑lived immutable caching for static bundles.  
- **Measure**: field data via web‑vitals + RUM dashboards; budget regressions with CI perf checks.

---

## 8) Accessibility (WCAG 2.2 AA)

- **Semantic roles**: list/listitem for messages, `role="feed"` optional; `aria-live="polite"` for new message announcements.  
- **Keyboard**: tab order predictable; arrow navigation within list; shortcuts with help sheet (`?`).  
- **Focus management**: return focus to composer after actions; focus trap in modals; visible focus ring.  
- **Color & contrast**: 4.5:1 for text; do not rely on color alone for state (add icons/labels).  
- **Screen reader**: announce “N new messages” and “Jumped to message at HH:MM”.  
- **RTL**: mirror layout, bidi isolation for mentions/inline code.

---

## 9) Security & Privacy (Front End)

- **Sanitize** all rich text (server‑side) + **escape** render; forbid inline HTML that can inject scripts.  
- **CSP**: `default-src 'self'; img-src cdn; connect-src api ws; object-src 'none'...`  
- **XSS**: no `dangerouslySetInnerHTML` unless through a vetted sanitizer.  
- **Clickjacking**: `X-Frame-Options/SameSite` enforced server‑side; front end avoids target=“_blank” w/o `rel="noopener"`.  
- **Auth**: OIDC/OAuth; short‑lived access tokens + silent refresh; store in memory when possible.  
- **PII handling**: redact in logs; toggle debug modes; gating via feature flags & RBAC.  
- **Uploads**: show file‑type warnings, size limits, server malware‑scan status; do not execute blobs inline.

---

## 10) Offline & Failure Modes

- **Service Worker**: cache last N pages of history & avatars; **Background Sync** to flush outbox.  
- **Outbox**: queue messages with `clientNonce`; retries with exponential backoff; conflict resolution on reconnect.  
- **Connectivity banner**: shows offline/online; keep local scroll & drafts usable.  
- **Degraded mode**: fall back to polling if websocket unavailable.  
- **Idempotence**: dedupe on `eventId` and `(channelId, messageId)` pairs.

---

## 11) Testing & Quality

- **Unit**: formatters, reducers, selectors, hooks.  
- **Component**: render message item/virtual list with jest‑dom/react‑testing‑library.  
- **Contract tests**: validate API DTOs with generated TypeScript types (OpenAPI/GraphQL codegen).  
- **E2E**: Playwright: joins channel, sends message w/ attachment, verifies optimistic → acked state, scroll restore.  
- **Performance**: scripted Lighthouse; Playwright trace view; synthetic scroll tests for jank.  
- **Accessibility**: axe‑core CI, keyboard nav tests, snapshots with high‑contrast themes.

**Playwright sketch**:
```ts
test('send message optimistic → delivered', async ({ page }) => {
  await page.goto('/c/eng');
  await page.getByRole('textbox', { name: /message/i }).fill('Hello world');
  await page.keyboard.press('Enter');
  await expect(page.getByText('Hello world')).toBeVisible();
  await expect(page.getByText('Hello world')).toHaveAttribute('data-status', 'sent'); // after ws ack
});
```

---

## 12) Example API Contracts (DTOs)

**REST/GraphQL‑ish** (simplified):

```
GET /channels/:id/messages?cursor=<ts>&limit=50&dir=backward|forward
→ { messages: Message[], nextCursor?: string }

POST /channels/:id/messages
{ clientNonce, body, attachments? }
→ { message: Message }  // server assigns id, createdAt

WS EVENT: message.created { message }
WS EVENT: message.updated { message }
WS EVENT: reaction.added { messageId, emoji, userId }
WS EVENT: read.receipt { channelId, messageId, userId, ts }
WS EVENT: typing { channelId, userId, isTyping }
```

---

## 13) State Management Options & Trade‑Offs

| Option | Pros | Cons | Good Fit |
|---|---|---|---|
| **RTK Query** | Batteries‑included cache, mutations, optimistic updates, TS‑friendly | Less flexible than DIY query cache | Large apps; unified store; Redux ecosystem |
| **React Query** | Powerful cache, mutations, devtools | Separate from Redux slice state | Apps not otherwise using Redux |
| **SWR** | Very simple hooks, lightweight | Less control for complex cache graphs | Simpler views |
| **Signals/MobX/Zustand** | Small boilerplate, good perf | DIY cache, more custom work | Perf‑critical UI bits |

**Pattern**: query library for **server cache** + thin local UI state (drafts, open panels). Normalize by IDs; keep selectors fast.

---

## 14) Search UX & Deep Linking

- Server returns **snippets** with hit highlights; front end shows context and **“jump to message”** with pinned anchor.  
- Keyboard navigation between results; result list is separate virtualized view.  
- **Deep link** `/c/:id/m/:messageId` loads around anchor via `around` API: `?centerId=<messageId>&radius=50`.

---

## 15) Telemetry & Ops

- **RUM**: LCP, FID, CLS, TTI, First Input Delay; user flows (open channel → send → ack).  
- **Errors**: capture stack traces + breadcrumbs, tag by channel size & device.  
- **Feature flags**: gradual rollout for threads, read receipts, and rich‑text editor.  
- **Debug panel** (hidden): last WS events, queue length, reconnect attempts.

---

## 16) Interview‑Style Q&A (Focused on Messaging)

**Q1. How do you design a virtualized chat list that supports images and threads?**  
- Use `react-window` with dynamic row measuring, cache heights; maintain anchor index when prepending; image `onload` re‑measure; avoid reflow storms via debounced remeasure. Threads open in a side drawer to avoid reflowing main list.

**Q2. How do you keep message state in sync with server (edits, reactions, read receipts)?**  
- Use WebSocket events to **patch** the query cache; reducers are idempotent by `eventId`. For edits: replace `body` and `editedAt`. For reactions: merge sets by `(messageId, emoji, userId)`. Read receipts: store map `messageId → Set(userId)` and project latest read per user.

**Q3. How do you support offline send?**  
- Outbox in IndexedDB; queue `{clientNonce, payload}`; SW Background Sync posts when online; UI shows “sending…” with retry/cancel. On ack, map nonce → real `id` and reconcile. On conflict/duplicate, discard by nonce mapping.

**Q4. What’s your approach to mentions and presence at scale?**  
- Mention autocomplete uses debounced query and **local LRU cache** of recent contacts; presence/typing updates are **coalesced** (e.g., batch every 1s) to prevent render storms. Use a separate WS channel for high‑churn presence events; backoff under load.

**Q5. How do you defend against XSS in rich messages?**  
- Sanitize on the server (e.g., allowlist markdown → sanitized HTML), escape at render, block inline styles/scripts, content‑security‑policy, and never render untrusted HTML without a sanitizer.

**Q6. How do you handle large channels (50k+ members)?**  
- Paginate member lists; lazy‑load avatars; window presence list; server‑side fanout throttled; front end filters locally once base list is cached. Mentions query server with prefix matching + paging.

**Q7. How do you keep scroll position stable as new messages arrive?**  
- If scrolled near bottom, append and auto‑scroll; else hold position and show “N new messages” with sticky CTA. Use **anchor‑based virtualization** to compute offsets and avoid layout jumps.

**Q8. SSR vs SPA for messaging?**  
- SSR not critical for closed/auth app; SPA for fast navigation and persistent WS connection. Optionally SSR shell for faster TTFB, but hydrate WS only client‑side.

**Q9. What would you log/measure in production?**  
- WS connect time, reconnect count, message send→ack latency, error counts by type, scroll jank %, LCP/CLS, failed uploads, cache hit rate, search latency and zero‑result rate.

**Q10. How to migrate safely when schemas change?**  
- Versioned DTOs; feature flags gating new fields; tolerant readers; dual‑write server for a period; front end with defensive parsing and fallback UI.

---

## 17) Minimal React + RTK Query Sketch (Illustrative)

```tsx
// api/messagesApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { Message } from './types';

export const messagesApi = createApi({
  reducerPath: 'messagesApi',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Messages'],
  endpoints: (b) => ({
    listMessages: b.query<{ messages: Message[]; nextCursor?: string }, { channelId: string; cursor?: string }> ({
      query: ({ channelId, cursor }) => ({ url: `/channels/${channelId}/messages`, params: { cursor, limit: 50 } }),
      providesTags: (_res, _err, { channelId }) => [{ type: 'Messages', id: channelId }],
    }),
    sendMessage: b.mutation<Message, { channelId: string; body: string; clientNonce: string }> ({
      query: ({ channelId, ...body }) => ({ url: `/channels/${channelId}/messages`, method: 'POST', body }),
      async onQueryStarted({ channelId, body, clientNonce }, { dispatch, queryFulfilled }) {
        // Optimistic insert
        const temp: Message = {
          id: `temp:${clientNonce}`, channelId, senderId: 'me', body, createdAt: new Date().toISOString(),
          status: 'sending', clientNonce
        };
        const patch = dispatch(messagesApi.util.updateQueryData('listMessages', { channelId }, (draft) => {
          draft.messages.push(temp);
        }));
        try {
          const { data } = await queryFulfilled;
          dispatch(messagesApi.util.updateQueryData('listMessages', { channelId }, (draft) => {
            const idx = draft.messages.findIndex(m => m.clientNonce === clientNonce);
            if (idx >= 0) draft.messages[idx] = { ...data, status: 'sent' };
          }));
        } catch {
          // mark failed
          dispatch(messagesApi.util.updateQueryData('listMessages', { channelId }, (draft) => {
            const idx = draft.messages.findIndex(m => m.clientNonce === clientNonce);
            if (idx >= 0) draft.messages[idx].status = 'failed';
          }));
        } finally {
          patch.undo?.();
        }
      },
    }),
  }),
});
```

```tsx
// components/MessagePane.tsx (simplified)
import { FixedSizeList as List } from 'react-window';
import { useListMessagesQuery } from '../api/messagesApi';

export function MessagePane({ channelId }: { channelId: string }) {
  const { data } = useListMessagesQuery({ channelId });
  const items = data?.messages ?? [];
  return (
    <List height={600} width={'100%'} itemCount={items.length} itemSize={72}>
      {({ index, style }) => <div style={style}>{items[index].body}</div>}
    </List>
  );
}
```

---

## 18) Rollout Plan

- **Phase 1**: core DM + group chat, text only, optimistic send, basic presence, virtualization.  
- **Phase 2**: attachments, reactions, mentions, read receipts.  
- **Phase 3**: threads + search + meeting hand‑offs.  
- **Phase 4**: offline outbox, PWA install, push notifications.  
- **Always‑on**: telemetry, a11y audits, perf budgets, feature flags.

---

## 19) Common Pitfalls (and how we avoid them)

- **Janky scrolling** from dynamic message heights → cache measurements, defer image layout shifts.  
- **Event storms** (typing/presence) → coalesce and throttle; render only visible participants.  
- **Duplicate messages** on reconnect → idempotent reducers keyed by messageId + eventId.  
- **Broken deep links** → server “load around” endpoint + front‑end anchor restoration.  
- **XSS via markdown** → strict sanitizer + render allowlist.

---

## 20) Quick Checklist (Interview Lightning Round)

- Clarify requirements & SLOs (latency, offline).  
- Pick data‑fetching & realtime strategy.  
- Normalize cache; plan for optimistic UI & reconciliation.  
- Virtualize long lists; design scroll/anchor behavior.  
- A11y & i18n from day 1.  
- Security: sanitize + CSP.  
- Test pyramid + perf + telemetry.  
- Rollout via flags; plan failure modes.

---

*Feel free to adapt code snippets to your stack. This document is intentionally front‑end centric and designed to be pasted into an engineering design repo.*

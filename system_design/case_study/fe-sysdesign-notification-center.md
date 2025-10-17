# Front-End System Design Deep Dive — Notification Center

> Design a **real-time notification bell** with unread counts, grouping, and persistence using **React + TypeScript + WebSocket**.

---

## 1. Requirements
- Bell icon with unread count.
- Dropdown list of notifications.
- Real-time updates.
- Mark as read individually or all.
- Persist read state locally.

---

## 2. Architecture
```
NotificationProvider
 ├─ WebSocketClient
 ├─ Cache (React Query)
 ├─ NotificationBell
 └─ NotificationList
```

---

## 3. Tech Choices

| Layer | Tech | Why | Alternatives |
|-------|------|-----|--------------|
| Real-time | **WebSocket** | Bidirectional; multiple event types | SSE (one‑way) |
| Cache | **React Query** | Auto refetch + background sync | Redux Toolkit |
| UI | **Headless UI Popover** | A11y‑ready dropdown | Custom CSS |
| Storage | **IndexedDB** | Offline persistence | localStorage |

---

## 4. Code Example

```tsx
function useNotifications() {
  const { data, setData } = useState<Notif[]>([]);
  useEffect(() => {
    const ws = new WebSocket("wss://api.example.com/notifications");
    ws.onmessage = (e) => setData(d => [...d, JSON.parse(e.data)]);
    return () => ws.close();
  }, []);
  return { data };
}

export function NotificationBell() {
  const { data } = useNotifications();
  const unread = data.filter(n => !n.read).length;
  return <button>{`🔔 ${unread}`}</button>;
}
```

---

## 5. Considerations
- Batch incoming events.
- Persist latest 100 notifications.
- Avoid over-fetching on reconnect.

---

## 6. Interview Soundbite
> “WebSocket events append to a cached list; React Query maintains consistency; IndexedDB provides offline persistence.”

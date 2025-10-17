# Front-End System Design Deep Dive — Online Whiteboard / Collaboration Canvas

> Real-time multi-user canvas with shapes, drawing, selection, panning/zooming, and presence cursors. Uses React + TS + Canvas/SVG with WebSocket (or WebRTC DC for P2P).

---

## 1. Requirements
- Draw shapes (rect, line, freehand), select/transform.
- Real-time presence cursors; multi-user edits.
- Infinite canvas (pan/zoom); undo/redo.
- Persistence and conflict resolution.

---

## 2. Architecture
```
WhiteboardApp
 ├─ CanvasLayer (HTML Canvas or SVG)
 ├─ ToolPanel (select, draw, text, erase)
 ├─ PresenceLayer (cursors)
 ├─ useBoardState() (React state + history)
 └─ Realtime: WS broadcast ops (CRDT/OT optional)
```

---

## 3. Tech Choices
| Area | Choice | Why | Alternatives |
|---|---|---|---|
| Rendering | **Canvas** for perf; **SVG** for simple DOM ops | Canvas better for freehand | WebGL for very large scenes |
| Realtime | **WebSocket** bus | Server fanout reliable | WebRTC DC (p2p) |
| Consistency | **CRDT/OT** (optional) | Avoid conflicts | Last-write-wins with server auth |

---

## 4. Implementation Sketch

```tsx
function Whiteboard() {
  const canvasRef = React.useRef<HTMLCanvasElement>(null);
  React.useEffect(() => {
    const ctx = canvasRef.current!.getContext("2d")!;
    // draw loop, handle pan/zoom
  }, []);
  return <canvas ref={canvasRef} width={1200} height={800} />;
}
```

Realtime op:
```ts
// { type:"addShape", shape:{ id, kind, points, color } }
// Server validates then broadcasts to room
```

---

## 5. Performance & UX
- Batch strokes; requestAnimationFrame for drawing.
- Spatial index for hit testing.
- Zoom levels; infinite canvas tiling.
- Predictive cursors to mask latency.

---

## 6. Testing
- Multi-client sync; conflict scenarios.
- Large stroke performance; pan/zoom precision.

---

## 7. Interview Summary
> “Canvas rendering with WS-broadcast ops; optional CRDT for conflict-free merges; RAF batching and spatial hit-testing for performance.”

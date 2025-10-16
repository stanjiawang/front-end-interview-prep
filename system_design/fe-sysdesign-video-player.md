# Front-End System Design Deep Dive — Video / Audio Streaming Player

> Build a robust **media player** (VOD + live) with buffering, adaptive bitrate, captions, and analytics in **React + TypeScript**. Compare **HTMLMediaElement**, **HLS/DASH via MSE**, **hls.js**, **Shaka**, **dash.js**, and why/when to choose each.

---

## 1. Requirements
**Functional**
- Play/pause/seek, volume, mute, picture-in-picture.
- Subtitles/closed captions (WebVTT), multiple audio tracks.
- Quality selection (auto + manual), playback rate.
- VOD and Live (DVR window).
- Error handling (stall, network fail) and recovery.

**Non-Functional**
- Fast start time; smooth playback (low rebuffer ratio).
- Adaptive bitrate under varying networks.
- Cross-browser compatibility; mobile support.
- Telemetry (startup time, rebuffer count, watch time).

---

## 2. Architecture
```
MediaPlayer (container)
 ├─ <video> element (HTMLMediaElement)
 ├─ Engine: HLS/DASH library (hls.js / Shaka / dash.js) or native HLS on Safari
 ├─ UI Controls: Play, SeekBar, Volume, CC, Quality
 ├─ Captions Renderer: <track> or TextTrack API
 └─ Analytics: hooks for events
```

---

## 3. Tech Stack: Options & Rationale

| Concern | Option | Why | Trade-offs / Why not others |
|---|---|---|---|
| Protocol | **HLS** | Ubiquitous, great CDN support | Segment-based latency vs LL-HLS |
| Protocol | **DASH** | Standardized, powerful features | Safari HLS-first ecosystem |
| Player lib | **hls.js** | Simple, HLS→MSE in Chrome/Firefox | HLS only; for DASH need another lib |
| Player lib | **Shaka Player** | HLS + DASH + DRM, robust | Larger API surface; heavier |
| Player lib | **dash.js** | Solid DASH support | Need separate flow for HLS |
| Native | **Safari HLS native** | Zero lib for iOS/macOS | Feature parity differences |
| DRM | **EME** (via Shaka) | Widevine/FairPlay/PlayReady | Complexity, license handling |

**Recommendation**: If you need **both HLS & DASH & DRM**, use **Shaka** for a unified stack; if **HLS-only** and want a small footprint, **hls.js** is excellent.

---

## 4. Implementation (React + TS)

```tsx
// VideoPlayer.tsx
import React, { useEffect, useRef, useState } from "react";

type Props = { src: string; type?: "hls" | "dash" | "mp4"; poster?: string };
export function VideoPlayer({ src, type = "hls", poster }: Props) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [ready, setReady] = useState(false);

  useEffect(() => {
    const video = videoRef.current!;
    async function setup() {
      if (type === "hls") {
        // Safari: native HLS
        if ((video as any).canPlayType("application/vnd.apple.mpegurl")) {
          video.src = src;
          setReady(true);
        } else {
          const Hls = (await import("hls.js")).default;
          if (Hls.isSupported()) {
            const hls = new Hls();
            hls.loadSource(src);
            hls.attachMedia(video);
            hls.on(Hls.Events.MANIFEST_PARSED, () => setReady(true));
          }
        }
      } else if (type === "mp4") {
        video.src = src;
        setReady(true);
      }
    }
    setup();
  }, [src, type]);

  return (
    <div className="player">
      <video ref={videoRef} poster={poster} controls playsInline />
      {!ready && <div className="loading">Loading…</div>}
    </div>
  );
}
```

---

## 5. Performance & Resilience
- Prefer **auto ABR**; expose manual override.
- For Live: enable **Low-Latency HLS** if available.
- Handle visibility change (pause analytics, reduce timers).
- Retry strategy: network errors → backoff; stall → seek workaround.

---

## 6. Accessibility & UX
- Keyboard shortcuts; focusable controls; visible focus ring.
- Captions via `<track kind="subtitles" srcLang="en">`.
- High-contrast UI; screen-reader labels.

---

## 7. Analytics
- `startupTime`, `rebufferCount/Duration`, `bitrate switches`, `seek count`, `watch time`.
- Error taxonomy with codes.

---

## 8. Interview Summary
> “Choose hls.js for HLS-only or Shaka for HLS/DASH/DRM. I wrap the engine behind a React component, collect playback analytics, and implement stall recovery and ABR controls.”

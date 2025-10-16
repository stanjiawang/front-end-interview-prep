# Front-End System Design Deep Dive — File Upload System

> Implement a **robust file upload** module supporting drag‑and‑drop, progress, cancel, retry, and resumable uploads using **React + TypeScript**.

---

## 1. Requirements
- Multi‑file drag & drop and file input.
- Progress bar and cancel per file.
- Retry on failure, resume from checkpoint.
- Handle large files (GB‑scale).

---

## 2. Architecture
```
UploadManager
 ├─ useUploader() hook
 ├─ FileList (progress + controls)
 ├─ FileItem (state: queued/uploading/success/error)
 ├─ UploadWorker (chunked POSTs + retry)
 └─ Service: /api/upload, /api/upload/chunk, /api/upload/complete
```

---

## 3. Tech Stack & Choices

| Concern | Tech | Reason | Alternative |
|----------|------|--------|-------------|
| Transport | **XHR / Fetch** with `FormData` | Simpler; progress via XHR | WebSocket stream, WebTransport |
| Concurrency | **Promise pool** | Control # of parallel uploads | Web Workers |
| Resume | **Chunked upload** (5 MB parts) | Reliable retry for large files | Multipart upload (S3 API) |
| State | **React Query** | Mutation + optimistic states | Redux Toolkit |
| UI | **Tailwind / CSS Grid** | Responsive progress cards | MUI |

---

## 4. Implementation

```tsx
function useUploader() {
  const [files, setFiles] = useState<FileState[]>([]);

  const uploadFile = async (f: File) => {
    const chunkSize = 5 * 1024 * 1024;
    for (let offset = 0; offset < f.size; offset += chunkSize) {
      const chunk = f.slice(offset, offset + chunkSize);
      await fetch("/api/upload", { method: "POST", body: chunk });
    }
  };

  return { files, uploadFile };
}
```

---

## 5. Performance / Resilience
- Limit parallel uploads (3‑5).
- Show ETA and retry after exponential backoff.
- Persist state to IndexedDB for crash recovery.

---

## 6. Interview Soundbite
> “I use chunked, resumable uploads with controlled concurrency and retry logic to ensure resilience under poor network conditions.”

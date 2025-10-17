# Next.js Front-End Interview Q&A (App Router, 2025)

A compact, **practical** set of Next.js questions with **minimal example code** using the App Router (`/app`). Built to be copy‑paste‑ready for interviews.

---

## 1) What is the App Router? How is it different from the Pages Router?
**Answer:**  
- **App Router (`/app`)** (Next.js 13+): React Server Components by default, nested layouts, streaming, route handlers, advanced routing (`parallel`, `intercepting`), and file‑system conventions for data/metadata.  
- **Pages Router (`/pages`)**: Client components by default, `getServerSideProps` / `getStaticProps` APIs, older data‑fetching model.

**Key takeaway:** App Router enables **server-first UI**, **automatic code-splitting**, and **granular rendering** (SSR/SSG/ISR) with **server actions** and **RSC**.

---

## 2) Server Components vs Client Components
**Answer:**  
- **Server Components (default):** Rendered on server, no JS sent to client; ideal for data fetching, heavy logic, secrets.  
- **Client Components:** Opt-in using `'use client'`; needed for interactivity, hooks like `useState`, `useEffect`.

```tsx
// app/users/page.tsx  (Server Component by default)
import { Suspense } from 'react'
import Users from './Users'

export default function Page() {
  return (
    <Suspense fallback={<p>Loading…</p>}>
      <Users />
    </Suspense>
  )
}
```

```tsx
// app/users/UserList.tsx
export default async function Users() {
  const res = await fetch('https://api.example.com/users', { cache: 'no-store' })
  const users = await res.json()
  return <pre>{JSON.stringify(users, null, 2)}</pre>
}
```

```tsx
// app/users/Search.tsx  (needs interactivity)
'use client'
import { useState } from 'react'
export default function Search() {
  const [q, setQ] = useState('')
  return <input value={q} onChange={(e) => setQ(e.target.value)} />
}
```

---

## 3) Data Fetching: SSR / SSG / ISR (App Router)
**Answer:**  
- **SSR:** `fetch(..., { cache: 'no-store' })` or `revalidate: 0`  
- **SSG:** default `fetch` with full caching  
- **ISR:** set `next: { revalidate: N }` seconds

```tsx
// SSR (no cache)
const res = await fetch('https://api.example.com/posts', { cache: 'no-store' })

// ISR (revalidate every 60s)
const res2 = await fetch('https://api.example.com/posts', { next: { revalidate: 60 } })

// SSG (default). Use stable URLs + default caching.
```

**Route segment revalidation:**  
```ts
export const revalidate = 120 // revalidate this route every 2 minutes
```

---

## 4) Route Handlers vs API Routes
**Answer:**  
- **Route Handlers (`app/api/**/route.ts`)** replace Pages Router **API routes**.  
- Support **Edge** / **Node** runtimes, streaming, and `Request/Response` Web APIs.

```ts
// app/api/hello/route.ts
export async function GET() {
  return new Response(JSON.stringify({ msg: 'hello' }), { headers: { 'content-type': 'application/json' } })
}

export async function POST(req: Request) {
  const body = await req.json()
  return Response.json({ ok: true, body })
}
```

---

## 5) Navigation, Link, and `useRouter`
```tsx
import Link from 'next/link'
export default function Product({ params }: { params: { id: string } }) {
  return (
    <div>
      <h1>Product {params.id}</h1>
      <Link href="/cart">Go to Cart</Link>
    </div>
  )
}
```

```tsx
'use client'
import { useRouter } from 'next/navigation'
export default function Btn() {
  const router = useRouter()
  return <button onClick={() => router.push('/checkout')}>Checkout</button>
}
```

---

## 6) Layouts, Templates, Error & Loading UI
```tsx
// app/layout.tsx  (root layout)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

```tsx
// app/dashboard/layout.tsx  (nested layout)
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return <section className="dash-shell">{children}</section>
}
```

```tsx
// app/dashboard/error.tsx
'use client'
export default function Error({ error }: { error: Error }) {
  return <p>Something broke: {error.message}</p>
}
```

---

## 7) Dynamic, Catch-All, Optional Routes
```tsx
// app/blog/[slug]/page.tsx
export default function Page({ params }: { params: { slug: string } }) {
  return <h1>{params.slug}</h1>
}
```

---

## 8) Metadata and SEO
```ts
export const metadata = {
  title: 'Blog Post',
  description: 'Details of the post',
}
```

---

## 9) Caching & Revalidation
```ts
// app/actions.ts
'use server'
import { revalidatePath, revalidateTag } from 'next/cache'

export async function refreshProducts() {
  revalidateTag('products')
  revalidatePath('/products')
}
```

---

## 10) Server Actions
```tsx
'use server'
export async function createTodo(formData: FormData) {
  const title = formData.get('title') as string
  return { ok: true }
}
```

---

## 11) Middleware
```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(req: NextRequest) {
  if (req.nextUrl.pathname.startsWith('/admin')) {
    return NextResponse.redirect(new URL('/login', req.url))
  }
  return NextResponse.next()
}

export const config = { matcher: ['/admin/:path*'] }
```

---

## 12) Dynamic Import & Performance
```tsx
import dynamic from 'next/dynamic'
const Heavy = dynamic(() => import('./HeavyChart'), { ssr: false })
export default function Page() { return <Heavy /> }
```

---

## 13) Common Gotchas
1. RSC vs Client components — use `'use client'` carefully.
2. Default `fetch` is cached — use `{ cache: 'no-store' }` for SSR.
3. Server actions must have `'use server'` at the top.
4. Route handler vs pages API route differences.

---

## 14) Final Tips
- Clarify SSR vs SSG vs ISR in interviews.
- Mention Edge runtime trade-offs.
- Know the file conventions: `layout.tsx`, `page.tsx`, `error.tsx`, `loading.tsx`, `route.ts`.

# Front-End System Design Deep Dive — E‑Commerce: PLP + PDP

> A pragmatic design for **Product Listing Page (PLP)** + **Product Detail Page (PDP)** using **React + Next.js (App Router) + TypeScript** with SSR/SSG, responsive images, and interactive widgets.

---

## 1. Requirements
- PLP: filters (facets), sort, pagination/infinite scroll, SEO-friendly URLs.
- PDP: gallery, variants, reviews, related products, add-to-cart, availability.
- Performance: LCP < 2.5s, responsive images, code splitting.
- SEO: SSR/SSG + structured data; canonical URLs.

---

## 2. Architecture
```
Next.js (App Router)
 ├─ /products (server component page: PLP)
 ├─ /products/[slug] (server component page: PDP)
 └─ client components: Filters, ProductGrid, Gallery, AddToCart, ReviewList
Data: server fetch (RSC) + client hydration (React Query) for interactions
Images: CDN with AVIF/WebP and width-based srcset
```

---

## 3. URL & Filters
- Encode filters in query params, e.g. `/products?brand=nike&size=10&price=50-100&sort=popular`.
- Use server-side filtering to keep SEO & consistent pagination.

---

## 4. Code Sketches

**PLP server page**
```tsx
// app/products/page.tsx
import ProductGrid from "./ProductGrid";
import { fetchProducts } from "@/lib/api";

export default async function ProductsPage({ searchParams }: { searchParams: Record<string,string> }) {
  const initial = await fetchProducts(searchParams); // runs on server
  return <ProductGrid initial={initial} />; // client component for hydration + further interactions
}
```

**Client ProductGrid with React Query**
```tsx
"use client";
import { useQuery } from "@tanstack/react-query";

export default function ProductGrid({ initial }: { initial: any }) {
  const { data } = useQuery({
    queryKey: ["products", initial.queryKey],
    queryFn: () => fetch(`/api/products?${initial.query}`).then(r => r.json()),
    initialData: initial,
    keepPreviousData: true,
  });
  return (
    <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
      {data.items.map((p: any) => (
        <article key={p.id} className="border rounded p-2">
          <img src={p.imageUrl} alt={p.name} loading="lazy" className="w-full aspect-square object-cover" />
          <div className="mt-2 font-medium">{p.name}</div>
          <div>${p.price}</div>
        </article>
      ))}
    </div>
  );
}
```

**PDP Gallery (responsive)**
```tsx
export function Gallery({ images }: { images: { url: string; w: number }[] }) {
  const srcset = images.map(i => `${i.url} ${i.w}w`).join(", ");
  return (
    <picture>
      <source srcSet={srcset} sizes="(min-width:768px) 50vw, 100vw" />
      <img src={images[0].url} alt="" loading="lazy" className="w-full object-contain" />
    </picture>
  );
}
```

---

## 5. Performance & SEO
- Server render HTML; cache at CDN with vary by query.
- Defer heavy widgets (reviews) via dynamic import; prefetch PDP on hover from PLP.
- Add Product JSON-LD; ensure meaningful alt text; stable layout to avoid CLS.

---

## 6. Testing
- Lighthouse budgets; synthetic tests for CLS/LCP.
- Playwright flows: filter, sort, add-to-cart.
- Snapshot schema markup.

---

## 7. Interview Soundbite
> “I combine Next.js server components for initial HTML with React Query for client interactions. Responsive images and code splitting keep LCP low while preserving SEO.”

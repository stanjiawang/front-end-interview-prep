# Front-End System Design Deep Dive — SPA vs SSR vs SSG vs RSC

> Choose the right rendering strategy for different pages, balancing SEO, TTFB, interactivity, and ops complexity.

---

## 1. Options at a Glance
| Mode | Rendered | Pros | Cons | Best For |
|---|---|---|---|---|
| **SPA (CSR)** | Client | App-like UX, persistent state | Slow first paint, SEO weak | Auth apps, dashboards |
| **SSR** | Server per request | Great SEO, fast TTFB | Server cost, hydration | Blogs, product pages |
| **SSG** | Build time | Lightning fast, CDN | Stale content | Docs, marketing |
| **ISR** | Revalidated | Fresh-ish + CDN | Complexity | Catalogs, news |
| **RSC** | Server + client islands | Less JS shipped | New model | Mixed apps |

---

## 2. Decision Guide
- SEO-critical? → SSR/SSG/ISR.
- Authenticated/internal? → SPA/RSC.
- Highly interactive? → SPA with client islands; or RSC for server data + small clients.
- Data freshness? → SSR or ISR window tuned to updates.

---

## 3. Examples

**Next.js SSR page**
```tsx
// app/article/[slug]/page.tsx
export default async function Page({ params }) {
  const article = await fetchArticle(params.slug);
  return <Article article={article} />;
}
```

**SPA route (Vite/React)**
```tsx
export default function Dashboard() {
  const [data, setData] = useState<any>(null);
  useEffect(() => { fetch("/api/metrics").then(r => r.json()).then(setData); }, []);
  return data ? <Charts data={data} /> : <Spinner />;
}
```

---

## 4. Trade-offs Table
| Aspect | SPA | SSR | SSG | ISR | RSC |
|---|---|---|---|---|---|
| TTFB | Slow | Fast | Instant | Fast | Fast |
| SEO | Poor | Excellent | Excellent | Excellent | Excellent |
| Server Cost | Low | High | Low | Medium | Medium |
| Complexity | Low | Medium | Medium | High | High |
| Personalization | Client | Server | Limited | Some | Mixed |

---

## 5. Interview Soundbite
> “I mix SSR/SSG for public surfaces and SPA/RSC for app-like flows. I avoid over-hydration by isolating interactive islands.”

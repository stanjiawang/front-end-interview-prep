# React Coding Problems — With Bootstrap

> Each exercise includes: **Goal → Constraints → Hints → Solution (React + Bootstrap)**. You can copy-paste solutions into a Vite React app and add Bootstrap via CDN or npm.

## Setup (Vite + Bootstrap)

```bash
npm create vite@latest react-bootstrap-lab -- --template react
cd react-bootstrap-lab
npm i
# Option A: CDN in index.html
# <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
# <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
# Option B: npm
# npm i bootstrap
# in main.jsx: import 'bootstrap/dist/css/bootstrap.min.css'; import 'bootstrap/dist/js/bootstrap.bundle.min.js';
```

---

## 1) Filterable Product Table (Search + Category + Stock Badge)

**Goal:** Build a table with search box, category filter, and an **in-stock** badge.  
**Constraints:** Client-side filter; highlight match; accessible.  
**Hints:** Debounce search; memoize filtered rows.

```jsx
import React from "react";

const PRODUCTS = [
  { id: 1, name: "Apple", category: "Fruit", price: 1.5, stock: 30 },
  { id: 2, name: "Banana", category: "Fruit", price: 1.2, stock: 0 },
  { id: 3, name: "Carrot", category: "Vegetable", price: 0.9, stock: 10 },
];

function useDebounce(value, delay = 300) {
  const [v, setV] = React.useState(value);
  React.useEffect(() => {
    const id = setTimeout(() => setV(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return v;
}

export default function ProductTable() {
  const [query, setQuery] = React.useState("");
  const [category, setCategory] = React.useState("All");
  const dq = useDebounce(query);

  const categories = React.useMemo(
    () => ["All", ...new Set(PRODUCTS.map(p => p.category))],
    []
  );

  const rows = React.useMemo(() => {
    const q = dq.toLowerCase();
    return PRODUCTS.filter(p => (category === "All" || p.category === category) &&
      p.name.toLowerCase().includes(q));
  }, [dq, category]);

  return (
    <div className="container py-4">
      <div className="row g-2">
        <div className="col-sm">
          <input className="form-control" placeholder="Search" value={query}
                 onChange={e => setQuery(e.target.value)} />
        </div>
        <div className="col-sm">
          <select className="form-select" value={category} onChange={e => setCategory(e.target.value)}>
            {categories.map(c => <option key={c}>{c}</option>)}
          </select>
        </div>
      </div>

      <table className="table table-hover mt-3">
        <thead><tr><th>Name</th><th>Category</th><th>Price</th><th>Stock</th></tr></thead>
        <tbody>
          {rows.map(p => (
            <tr key={p.id} className={p.stock === 0 ? "table-warning" : ""}>
              <td>{p.name}</td>
              <td>{p.category}</td>
              <td>${p.price.toFixed(2)}</td>
              <td>
                {p.stock > 0
                  ? <span className="badge text-bg-success">In stock ({p.stock})</span>
                  : <span className="badge text-bg-secondary">Out</span>}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 2) Paginated, Sortable Table

**Goal:** Generic table with client-side sorting and pagination.  
**Constraints:** Keep columns configurable; keyboard accessible headers.

```jsx
function useSort(data, initial = { key: null, dir: "asc" }) {
  const [sort, setSort] = React.useState(initial);
  const sorted = React.useMemo(() => {
    if (!sort.key) return data;
    const arr = [...data].sort((a, b) => {
      const x = a[sort.key], y = b[sort.key];
      return (x > y ? 1 : x < y ? -1 : 0) * (sort.dir === "asc" ? 1 : -1);
    });
    return arr;
  }, [data, sort]);
  return [sorted, sort, setSort];
}

function usePage(data, size = 5) {
  const [page, setPage] = React.useState(1);
  const pageCount = Math.max(1, Math.ceil(data.length / size));
  const slice = React.useMemo(() => data.slice((page - 1) * size, page * size), [data, page, size]);
  return [slice, page, pageCount, setPage];
}

export function SortablePaginatedTable({ rows, columns }) {
  const [sorted, sort, setSort] = useSort(rows);
  const [pageRows, page, pageCount, setPage] = usePage(sorted, 5);

  const toggleSort = (key) => setSort(s => (s.key === key ? { key, dir: s.dir === "asc" ? "desc" : "asc" } : { key, dir: "asc" }));

  return (
    <div className="table-responsive">
      <table className="table align-middle">
        <thead>
          <tr>
            {columns.map(c => (
              <th key={c.key} scope="col">
                <button className="btn btn-link p-0" onClick={() => toggleSort(c.key)}>
                  {c.header}
                  {sort.key === c.key ? (sort.dir === "asc" ? " ▲" : " ▼") : ""}
                </button>
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {pageRows.map((r, i) => (
            <tr key={r.id ?? i}>
              {columns.map(c => <td key={c.key}>{c.render ? c.render(r) : r[c.key]}</td>)}
            </tr>
          ))}
        </tbody>
      </table>
      <nav>
        <ul className="pagination">
          <li className={"page-item" + (page === 1 ? " disabled" : "")}>
            <button className="page-link" onClick={() => setPage(1)}>« First</button>
          </li>
          <li className={"page-item" + (page === 1 ? " disabled" : "")}>
            <button className="page-link" onClick={() => setPage(p => Math.max(1, p - 1))}>‹ Prev</button>
          </li>
          <li className="page-item disabled"><span className="page-link">{page} / {pageCount}</span></li>
          <li className={"page-item" + (page === pageCount ? " disabled" : "")}>
            <button className="page-link" onClick={() => setPage(p => Math.min(pageCount, p + 1))}>Next ›</button>
          </li>
          <li className={"page-item" + (page === pageCount ? " disabled" : "")}>
            <button className="page-link" onClick={() => setPage(pageCount)}>Last »</button>
          </li>
        </ul>
      </nav>
    </div>
  );
}
```

---

## 3) Modal Wizard (Multi-step Form)

**Goal:** Build a multi-step modal form (Bootstrap modal) with validation per step.  
**Key:** Manage focus and allow keyboard navigation.

```jsx
export function WizardModal() {
  const [step, setStep] = React.useState(1);
  const [form, setForm] = React.useState({ name: "", email: "", plan: "basic" });
  const next = () => setStep(s => Math.min(3, s + 1));
  const prev = () => setStep(s => Math.max(1, s - 1));

  const canNext = step === 1 ? form.name.trim() : step === 2 ? /.+@.+/.test(form.email) : true;

  return (
    <div className="modal show d-block" tabIndex="-1" role="dialog" aria-modal="true">
      <div className="modal-dialog">
        <div className="modal-content">
          <div className="modal-header"><h5 className="modal-title">Sign Up</h5></div>
          <div className="modal-body">
            {step === 1 && (
              <div>
                <label className="form-label">Name</label>
                <input className="form-control" value={form.name} onChange={e => setForm(f => ({...f, name: e.target.value}))} />
              </div>
            )}
            {step === 2 && (
              <div>
                <label className="form-label">Email</label>
                <input className="form-control" value={form.email} onChange={e => setForm(f => ({...f, email: e.target.value}))} />
              </div>
            )}
            {step === 3 && (
              <div>
                <label className="form-label">Plan</label>
                <select className="form-select" value={form.plan} onChange={e => setForm(f => ({...f, plan: e.target.value}))}>
                  <option value="basic">Basic</option>
                  <option value="pro">Pro</option>
                </select>
              </div>
            )}
          </div>
          <div className="modal-footer">
            <button className="btn btn-secondary" onClick={prev} disabled={step === 1}>Back</button>
            {step < 3 ? (
              <button className="btn btn-primary" onClick={next} disabled={!canNext}>Next</button>
            ) : (
              <button className="btn btn-success" onClick={() => alert(JSON.stringify(form))}>Finish</button>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 4) Accordion FAQ with Deep-Linking

**Goal:** Accordion where each item is linkable by hash `#q-id`.  
**Hint:** Manage open state from `location.hash`.

```jsx
export function FAQ({ items }) {
  const [open, setOpen] = React.useState(null);
  React.useEffect(() => {
    const sync = () => setOpen(window.location.hash.slice(1) || null);
    sync();
    window.addEventListener("hashchange", sync);
    return () => window.removeEventListener("hashchange", sync);
  }, []);

  return (
    <div className="accordion" id="faq">
      {items.map((it, idx) => (
        <div className="accordion-item" key={it.id} id={it.id}>
          <h2 className="accordion-header">
            <button className={"accordion-button" + (open === it.id ? "" : " collapsed")}
              onClick={() => setOpen(o => (o === it.id ? null : it.id))}>
              {it.q}
            </button>
          </h2>
          <div className={"accordion-collapse collapse" + (open === it.id ? " show" : "")}}>
            <div className="accordion-body">{it.a}</div>
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

## 5) Debounced Typeahead (Remote Data)

**Goal:** Input that fetches suggestions after debounce and shows a list group.  
**Hint:** Cancel in-flight requests on new input.

```jsx
export function Typeahead() {
  const [q, setQ] = React.useState("");
  const [items, setItems] = React.useState([]);
  const [loading, setLoading] = React.useState(false);

  React.useEffect(() => {
    if (!q) { setItems([]); return; }
    const ctrl = new AbortController();
    const id = setTimeout(async () => {
      setLoading(true);
      try {
        // fake endpoint
        const data = ["alfa","beta","charlie","delta"].filter(x => x.includes(q.toLowerCase()));
        await new Promise(r => setTimeout(r, 300));
        if (!ctrl.signal.aborted) setItems(data);
      } finally { setLoading(false); }
    }, 300);
    return () => { ctrl.abort(); clearTimeout(id); };
  }, [q]);

  return (
    <div className="position-relative">
      <input className="form-control" placeholder="Search..." value={q} onChange={e => setQ(e.target.value)} />
      {loading && <div className="spinner-border spinner-border-sm position-absolute end-0 top-50 me-2" role="status" />}
      {items.length > 0 && (
        <ul className="list-group position-absolute w-100" style={{ zIndex: 10 }}>
          {items.map(x => <li key={x} className="list-group-item">{x}</li>)}
        </ul>
      )}
    </div>
  );
}
```

---

## 6) Toast Notifications Manager

```jsx
function useToasts() {
  const [toasts, setToasts] = React.useState([]);
  const add = (msg) => setToasts(ts => [...ts, { id: Date.now(), msg }]);
  const remove = (id) => setToasts(ts => ts.filter(t => t.id !== id));
  return { toasts, add, remove };
}

export function ToastArea() {
  const { toasts, add, remove } = useToasts();
  return (
    <div>
      <button className="btn btn-primary" onClick={() => add("Saved!")}>Notify</button>
      <div className="toast-container position-fixed bottom-0 end-0 p-3">
        {toasts.map(t => (
          <div key={t.id} className="toast show">
            <div className="toast-header">
              <strong className="me-auto">Notice</strong>
              <button className="btn-close" onClick={() => remove(t.id)}></button>
            </div>
            <div className="toast-body">{t.msg}</div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 7) Form with Bootstrap Validation Styles

```jsx
export function SignupForm() {
  const [form, setForm] = React.useState({ email: "", password: "", agree: false });
  const [submitted, setSubmitted] = React.useState(false);

  const emailValid = /.+@.+/.test(form.email);
  const passValid = form.password.length >= 8;
  const formValid = emailValid && passValid && form.agree;

  const onSubmit = (e) => {
    e.preventDefault();
    setSubmitted(true);
    if (formValid) alert("OK");
  };

  const cn = (ok) => "form-control " + (submitted ? (ok ? "is-valid" : "is-invalid") : "");

  return (
    <form className="needs-validation" noValidate onSubmit={onSubmit}>
      <div className="mb-3">
        <label className="form-label">Email</label>
        <input className={cn(emailValid)} value={form.email} onChange={e => setForm(f => ({...f, email: e.target.value}))} />
        <div className="invalid-feedback">Please provide a valid email.</div>
      </div>
      <div className="mb-3">
        <label className="form-label">Password</label>
        <input type="password" className={cn(passValid)} value={form.password} onChange={e => setForm(f => ({...f, password: e.target.value}))} />
        <div className="invalid-feedback">Minimum 8 characters.</div>
      </div>
      <div className="form-check mb-3">
        <input className="form-check-input" type="checkbox" checked={form.agree} onChange={e => setForm(f => ({...f, agree: e.target.checked}))} id="agree" />
        <label htmlFor="agree" className="form-check-label">I agree to terms</label>
      </div>
      <button className="btn btn-success" disabled={submitted && !formValid}>Create account</button>
    </form>
  );
}
```

---

## 8) Tabbed Dashboard with Cards

```jsx
export function Dashboard() {
  const tabs = ["Overview", "Usage", "Billing"];
  const [tab, setTab] = React.useState(tabs[0]);
  return (
    <div>
      <ul className="nav nav-tabs">
        {tabs.map(t => (
          <li className="nav-item" key={t}>
            <button className={"nav-link" + (tab === t ? " active" : "")}} onClick={() => setTab(t)}>{t}</button>
          </li>
        ))}
      </ul>
      <div className="row g-3 mt-1">
        {[1,2,3].map(n => (
          <div className="col-md-4" key={n}>
            <div className="card h-100">
              <div className="card-body">
                <h5 className="card-title">Card {n}</h5>
                <p className="card-text">Some quick example text.</p>
                <a className="btn btn-outline-primary">Action</a>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 9) Infinite Scroll List

```jsx
export function InfiniteList() {
  const [items, setItems] = React.useState(Array.from({ length: 20 }, (_, i) => i));
  const loadMore = () => setItems(arr => [...arr, ...Array.from({ length: 20 }, (_, i) => arr.length + i)]);

  React.useEffect(() => {
    const onScroll = () => {
      if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 100) loadMore();
    };
    window.addEventListener("scroll", onScroll, { passive: true });
    return () => window.removeEventListener("scroll", onScroll);
  }, []);

  return (
    <ul className="list-group">
      {items.map(i => <li key={i} className="list-group-item">Item {i}</li>)}
    </ul>
  );
}
```

---

## 10) Image Gallery with Modal Viewer

```jsx
export function Gallery({ images }) {
  const [open, setOpen] = React.useState(null);
  return (
    <div className="row g-2">
      {images.map((src, i) => (
        <div className="col-6 col-md-3" key={i}>
          <img role="button" className="img-fluid rounded" src={src} alt="" onClick={() => setOpen(src)} />
        </div>
      ))}
      {open && (
        <div className="modal show d-block" tabIndex="-1">
          <div className="modal-dialog modal-lg">
            <div className="modal-content">
              <img src={open} alt="" />
              <div className="modal-footer">
                <button className="btn btn-secondary" onClick={() => setOpen(null)}>Close</button>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## 11) Shopping Cart (Lifted State + Memo)

```jsx
export function Shop() {
  const products = React.useMemo(() => [
    { id: 1, name: "Keyboard", price: 49.99 },
    { id: 2, name: "Mouse", price: 29.99 }
  ], []);
  const [cart, setCart] = React.useState([]);
  const add = (p) => setCart(c => { 
    const i = c.findIndex(x => x.id === p.id);
    if (i >= 0) return c.map((x, idx) => idx === i ? { ...x, qty: x.qty + 1 } : x);
    return [...c, { ...p, qty: 1 }];
  });
  const total = React.useMemo(() => cart.reduce((s, x) => s + x.price * x.qty, 0), [cart]);

  return (
    <div className="row">
      <div className="col-md-8">
        <div className="row g-3">
          {products.map(p => (
            <div className="col-md-6" key={p.id}>
              <div className="card h-100">
                <div className="card-body">
                  <h5 className="card-title">{p.name}</h5>
                  <p className="card-text">${p.price.toFixed(2)}</p>
                  <button className="btn btn-primary" onClick={() => add(p)}>Add</button>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
      <div className="col-md-4">
        <h5>Cart</h5>
        <ul className="list-group mb-2">
          {cart.map(x => <li key={x.id} className="list-group-item d-flex justify-content-between">{x.name} × {x.qty} <span>${(x.qty * x.price).toFixed(2)}</span></li>)}
        </ul>
        <div className="alert alert-info">Total: ${total.toFixed(2)}</div>
      </div>
    </div>
  );
}
```

---

## 12) Dark Mode Toggle (Bootstrap 5.3)

```jsx
export function ThemeToggle() {
  const [mode, setMode] = React.useState(() => localStorage.getItem("mode") || "light");
  React.useEffect(() => {
    document.documentElement.setAttribute("data-bs-theme", mode);
    localStorage.setItem("mode", mode);
  }, [mode]);
  return (
    <div className="form-check form-switch">
      <input id="theme" className="form-check-input" type="checkbox" checked={mode === "dark"}
             onChange={e => setMode(e.target.checked ? "dark" : "light")} />
      <label htmlFor="theme" className="form-check-label">Dark mode</label>
    </div>
  );
}
```

---

## 13) Accessible Toast Live Region (ARIA)

```jsx
export function LiveRegionToast({ message }) {
  return (
    <div className="visually-hidden" aria-live="polite" aria-atomic="true">
      {message}
    </div>
  );
}
```

---

## 14) Responsive Navbar with Active Highlight

```jsx
export function Navbar({ items, active, onNavigate }) {
  return (
    <nav className="navbar navbar-expand-lg bg-body-tertiary">
      <div className="container-fluid">
        <a className="navbar-brand" href="#">Brand</a>
        <button className="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#nav">
          <span className="navbar-toggler-icon"></span>
        </button>
        <div className="collapse navbar-collapse" id="nav">
          <ul className="navbar-nav me-auto mb-2 mb-lg-0">
            {items.map(i => (
              <li className="nav-item" key={i.path}}>
                <a className={"nav-link" + (active === i.path ? " active" : "")} href={i.path} onClick={e => { e.preventDefault(); onNavigate(i.path); }}>
                  {i.label}
                </a>
              </li>
            ))}
          </ul>
        </div>
      </div>
    </nav>
  );
}
```

---

## 15) Drag-and-Drop Reorder (Keyboard Friendly)

*(Using simple up/down buttons for accessibility rather than HTML5 DnD)*

```jsx
export function ReorderableList() {
  const [items, setItems] = React.useState(["Alpha","Bravo","Charlie"]);

  const move = (i, dir) => setItems(arr => {
    const j = i + dir;
    if (j < 0 || j >= arr.length) return arr;
    const copy = [...arr];
    [copy[i], copy[j]] = [copy[j], copy[i]];
    return copy;
  });

  return (
    <ul className="list-group">
      {items.map((x, i) => (
        <li key={x} className="list-group-item d-flex justify-content-between align-items-center">
          <span>{x}</span>
          <div className="btn-group">
            <button className="btn btn-outline-secondary btn-sm" onClick={() => move(i, -1)} disabled={i === 0}>↑</button>
            <button className="btn btn-outline-secondary btn-sm" onClick={() => move(i, +1)} disabled={i === items.length - 1}>↓</button>
          </div>
        </li>
      ))}
    </ul>
  );
}
```

---

## Tips for Interview Delivery

- State constraints first; outline approach; mention trade-offs.  
- Narrate **why** you choose a hook/pattern, and how you’d **test** and **measure** it.  
- Make accessibility and performance part of the solution, not an afterthought.

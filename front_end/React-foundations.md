# React Foundations — Senior Frontend Interview Prep

> Use this as a *practical handbook* to refresh fundamentals, align on modern best practices, and prepare solid explanations for interviews. Examples default to modern React (v18+) with functional components.

## Table of Contents

- [Mental Model](#mental-model)
- [JSX & Components](#jsx--components)
- [Props vs State](#props-vs-state)
- [Rendering & Reconciliation](#rendering--reconciliation)
- [Hooks](#hooks)
  - [useState](#usestate)
  - [useEffect](#useeffect)
  - [useMemo & useCallback](#usememo--usecallback)
  - [useRef](#useref)
  - [useReducer](#usereducer)
  - [useContext & Context API](#usecontext--context-api)
  - [Custom Hooks](#custom-hooks)
- [Forms: Controlled vs Uncontrolled](#forms-controlled-vs-uncontrolled)
- [Refs, Portals, Imperative Handles](#refs-portals-imperative-handles)
- [Error Boundaries](#error-boundaries)
- [Suspense & Concurrent Features](#suspense--concurrent-features)
- [Performance](#performance)
- [State Management (when to lift vs external)](#state-management-when-to-lift-vs-external)
- [Routing](#routing)
- [Data Fetching Patterns](#data-fetching-patterns)
- [Server-Side Rendering & Next.js](#server-side-rendering--nextjs)
- [Accessibility](#accessibility)
- [Testing Strategy](#testing-strategy)
- [TypeScript with React](#typescript-with-react)
- [Styling Options](#styling-options)
- [Security Considerations](#security-considerations)
- [Common Pitfalls](#common-pitfalls)
- [Cheat Sheet](#cheat-sheet)

---

## Mental Model

React renders **pure UI from state**: `UI = f(state)`. State changes trigger re-renders which **reconcile** with the DOM (diff/patch). Components should be **pure** with side effects handled explicitly (effects, event handlers).

**Key idea:** Renders are cheap; DOM mutations are not. Keep side effects out of render and embrace **idempotent** render functions.

---

## JSX & Components

JSX is syntax sugar for `React.createElement(type, props, ...children)`.

```jsx
export function Hello({ name }) {
  return <h1>Hello, {name ?? "World"}!</h1>;
}
```

Rules:
- Components must be **pure**: same inputs → same output.
- **Top-level** hooks only; never in loops/conditions.
- Keys on lists must be **stable and unique** among siblings:

```jsx
{items.map(item => (
  <li key={item.id}>{item.label}</li>
))}
```

---

## Props vs State

- **Props**: read-only inputs from parent; change triggers child re-render.
- **State**: owned by component; changing it triggers re-render of that component subtree.

Lifting state up:
- When multiple children need the same data or need to coordinate behavior—**lift** to the closest common ancestor.

---

## Rendering & Reconciliation

React compares previous and next virtual tree to **reconcile** minimal DOM updates.
- Changes in type replace node subtree.
- Changes in props patch existing node.
- Keys guide list diffing; wrong keys → re-mounts (lost state).

**Batching**: state updates in React events are batched; after async/await boundaries they also batch in React 18+.

---

## Hooks

### useState
```jsx
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount(c => c + 1); // functional update prevents stale closures
  return <button onClick={increment}>{count}</button>;
}
```

### useEffect
Use for **side effects** (subscriptions, timers, network calls that depend on render result).

```jsx
function Clock() {
  const [now, setNow] = React.useState(() => new Date());
  React.useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(id); // cleanup on unmount or dependency change
  }, []);
  return <time dateTime={now.toISOString()}>{now.toLocaleTimeString()}</time>;
}
```

**Dependency array** lists *inputs* used by the effect body. Include everything you read from scope (except stable values like `setState`, refs).

### useMemo & useCallback

- `useMemo(factory, deps)` memoizes a **value** to avoid expensive recomputation.
- `useCallback(fn, deps)` memoizes a **function** to provide stable identity to children (e.g., when a child is memoized).

```jsx
const filtered = React.useMemo(
  () => items.filter(i => i.visible),
  [items]
);
const onSelect = React.useCallback((id) => setSelected(id), [setSelected]);
```

### useRef

- Mutable container that **does not re-render** on `.current` change.
- Access DOM nodes, store instance values.

```jsx
function FocusInput() {
  const inputRef = React.useRef(null);
  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

### useReducer

Good for **complex state transitions** or when updates depend on previous state many times.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "add": return { ...state, items: [...state.items, action.payload] };
    case "remove": return { ...state, items: state.items.filter(i => i.id !== action.id) };
    default: return state;
  }
}

function Cart() {
  const [state, dispatch] = React.useReducer(reducer, { items: [] });
  return (
    <div>
      <button onClick={() => dispatch({ type: "add", payload: { id: Date.now() } })}>Add</button>
      <pre>{JSON.stringify(state, null, 2)}</pre>
    </div>
  );
}
```

### useContext & Context API

Provide values through a tree without prop-drilling. Use for **stable** values (theme, auth, i18n). For **frequently changing** values, consider splitting context or external state.

```jsx
const ThemeContext = React.createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = React.useContext(ThemeContext);
  return <div data-theme={theme}>Toolbar</div>;
}
```

### Custom Hooks

Encapsulate reusable stateful logic.

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = React.useState(() => {
    const raw = localStorage.getItem(key);
    return raw != null ? JSON.parse(raw) : initialValue;
  });
  React.useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  return [value, setValue];
}
```

---

## Forms: Controlled vs Uncontrolled

- **Controlled**: value stored in React state; reliable, easy validation.
- **Uncontrolled**: value in DOM; access via refs; less overhead for large forms.

```jsx
// Controlled
<input value={email} onChange={e => setEmail(e.target.value)} />

// Uncontrolled
<input ref={emailRef} defaultValue="test@example.com" />
```

---

## Refs, Portals, Imperative Handles

- **Portals** render children into a different DOM subtree (e.g., modals).
- **forwardRef** and **useImperativeHandle** expose imperative APIs.

```jsx
const Modal = React.forwardRef(function Modal(_, ref) {
  const [open, setOpen] = React.useState(false);
  React.useImperativeHandle(ref, () => ({ open: () => setOpen(true) }));
  return open ? ReactDOM.createPortal(<div className="modal">Hello</div>, document.body) : null;
});
```

---

## Error Boundaries

Catch errors in **render**, lifecycle, and constructors *below* them; **not** async events.

```jsx
class ErrorBoundary extends React.Component { 
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) { console.error(error, info); }
  render() { return this.state.hasError ? <h1>Something went wrong.</h1> : this.props.children; }
}
```

---

## Suspense & Concurrent Features

- **Suspense** lets components wait for something (e.g., lazy-loaded code or data with a library that supports Suspense).  
- **Concurrent rendering** improves responsiveness; React may pause, abort, or resume renders.

```jsx
const UserCard = React.lazy(() => import("./UserCard"));
function App() {
  return (
    <React.Suspense fallback={<div>Loading…</div>}>
      <UserCard />
    </React.Suspense>
  );
}
```

---

## Performance

- Avoid unnecessary re-renders: `React.memo`, `useMemo`, `useCallback`.
- Split large lists: windowing (e.g., `react-window`).
- Defer non-critical work: `useTransition` (18+).

```jsx
const [isPending, startTransition] = React.useTransition();
const onSearch = (q) => startTransition(() => setQuery(q));
```

Measure first:
- **React DevTools Profiler**
- **Performance** tab in browser
- Use **keyed list** wisely; stable keys preserve state across reorders.

---

## State Management (when to lift vs external)

- **Lift state** when few components share it and update frequency is low to moderate.
- Use **context** for stable, global settings.
- For complex graphs/frequent updates: consider external stores (Redux Toolkit, Zustand, Jotai, Recoil). Prefer **colocation** first; introduce stores only when needed.

---

## Routing

- React Router (SPA) vs Next.js (file-based, SSR/SSG). Keep routes lazy-loaded.

```bash
npm i react-router-dom
```

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/settings" element={<Settings />} />
  </Routes>
</BrowserRouter>
```

---

## Data Fetching Patterns

- **fetch in effect** with cancellation.
- Use **SWR/React Query** for caching, dedupe, mutation states.
- With Suspense-enabled libraries, boundary falls back while data loads.

```jsx
React.useEffect(() => {
  const ctrl = new AbortController();
  fetch(url, { signal: ctrl.signal }).then(r => r.json()).then(setData);
  return () => ctrl.abort();
}, [url]);
```

---

## Server-Side Rendering & Next.js

SSR improves TTFB/SEO; hydrate on client. Next.js offers **SSR/SSG/ISR**, server actions, and file-based routing.

- Prefer **streaming** for better TTFB with Suspense.
- Handle **data consistency** across server and client.

---

## Accessibility

- Semantic HTML first (use `button`, `nav`, `main`, etc.).
- Manage focus, keyboard interactions.
- ARIA only to *add* missing semantics.
- Color contrast, skip links, visible focus rings.

---

## Testing Strategy

- **Unit** (pure functions, hooks).
- **Component** (React Testing Library): test behavior, not implementation details.
- **E2E** (Playwright/Cypress) for flows.

```jsx
// React Testing Library example
import { render, screen, fireEvent } from "@testing-library/react";
import Counter from "./Counter";

test("increments", () => {
  render(<Counter />);
  fireEvent.click(screen.getByRole("button"));
  expect(screen.getByRole("button")).toHaveTextContent("1");
});
```

---

## TypeScript with React

- Use `FC` sparingly; prefer explicit props.
- Derive types from data when possible.

```tsx
type ButtonProps = {
  onClick?: () => void;
  children: React.ReactNode;
};

export function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

---

## Styling Options

- CSS Modules / SCSS, Tailwind, CSS-in-JS, vanilla-extract. Choose based on team scale and runtime constraints.
- Co-locate styles with components. Keep **design tokens** centralized.

---

## Security Considerations

- Never dangerously set HTML from untrusted sources.
- Escape user content, validate forms, guard secrets in env server-side.
- Beware of third-party scripts and XSS vectors in event handlers/URLs.

---

## Common Pitfalls

- Missing effect deps → stale data or extra network calls.
- Unstable keys in lists → broken state.
- Heavy state at the top → unnecessary re-renders deep down.
- Overusing context for frequently changing values.

---

## Cheat Sheet

- Prefer **pure components**.
- Use **functional updates** when state depends on previous state.
- **Memoize** expensive calcs and stable callbacks for memoized children.
- **Measure then optimize**.
- Introduce **state libraries** only when necessary.


## useEffect Dependency Array
The dependency array controls when `useEffect` runs.

| Dependency Array | Meaning |
|------------------|----------|
| `[]` | Runs only once (on mount). |
| _(no array)_ | Runs every render. |
| `[deps]` | Runs when one or more dependencies change. |

Example:
```tsx
useEffect(() => {
  console.log('Effect triggered');
  return () => console.log('Cleanup');
}, [count]);
```

---

## useMemo and useCallback
Optimization hooks to prevent unnecessary re-renders.

- `useMemo(fn, deps)` caches the **result** of a computation.
- `useCallback(fn, deps)` caches the **function reference**.

Example:
```tsx
const expensiveValue = useMemo(() => compute(data), [data]);
const handleClick = useCallback(() => doSomething(value), [value]);
```

Avoid overusing them — memoization itself has a cost.

---

## Preventing Child Re-render
When a parent re-renders, children also re-render by default.  
To avoid it, memoize props and use `React.memo`:

```tsx
const Child = React.memo(({ onClick }) => {
  console.log('child render');
  return <button onClick={onClick}>Click</button>;
});

export default function Parent() {
  const [count, setCount] = useState(0);
  const handleClick = useCallback(() => setCount(c => c + 1), []);
  return <Child onClick={handleClick} />;
}
```

---

## Shallow Comparison
React uses shallow equality for performance — it compares **references**, not deep values.

```js
{ a: 1 } === { a: 1 } // false
const obj = { a: 1 }; obj === obj // true
```

This is why `{}` or `[]` created inline will trigger re-renders unless memoized.

---

## useRef
`useRef` stores a mutable value that persists between renders without causing re-render.

Use cases:
- Accessing DOM nodes
- Storing timers or previous values
- Avoiding stale closures

```tsx
const inputRef = useRef(null);
useEffect(() => inputRef.current?.focus(), []);
```

---

## Re-render vs Re-mount
| Concept | Description |
|----------|-------------|
| **Re-render** | Function executes again; state preserved. |
| **Re-mount** | Component unmounts and remounts; state reset. |

Re-mount triggers: changing `key`, conditional unmounting.  
Re-render happens on state/prop change and keeps existing state.

---
title: "React Interview Questions — Senior Frontend"
description: "Large set of React interview questions with concise, senior-level answers and code samples."
date: 2025-10-03
keywords: [React interview, hooks, Suspense, reconciliation, performance, testing, SSR, patterns, TypeScript]
---

# React Interview Questions — Senior Frontend

> Use this bank to practice *explanations first*, then code. Prioritize clarity, trade-offs, and measurable impact.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Hooks](#hooks)
- [Rendering & Reconciliation](#rendering--reconciliation)
- [State Management](#state-management)
- [Performance](#performance)
- [Forms](#forms)
- [Async & Data Fetching](#async--data-fetching)
- [Suspense & Concurrent](#suspense--concurrent)
- [SSR/SSG/Next.js](#ssrssgnextjs)
- [Testing](#testing)
- [Accessibility](#accessibility)
- [TypeScript](#typescript)
- [Security](#security)
- [Architecture & Patterns](#architecture--patterns)
- [Rapid Fire](#rapid-fire)

---

## Core Concepts

**Q: What is the difference between props and state?**  
A: Props are *inputs* passed from parent; state is owned by a component. Changing either triggers re-render, but props are immutable by the child.

**Q: Why keys in lists?**  
A: To help React match elements across renders and preserve instance state. Use stable, unique keys; avoid indices when list can reorder.

**Q: Controlled vs uncontrolled components?**  
A: Controlled stores value in React state; uncontrolled relies on DOM (refs). Controlled = validation, dynamic rules; uncontrolled = less render overhead for large forms.

**Q: How does React handle events?**  
A: Synthetic event system normalizes browser differences; handlers run during render commit. In React 18, event batching is automatic across async boundaries.

**Q: What is lifting state up?**  
A: Move state to the nearest common ancestor to synchronize child components.

---

## Hooks

**Q: Rules of Hooks? Why?**  
A: Only call at top-level and from React functions. Ensures React can maintain a consistent hook order between renders.

**Q: useEffect vs useLayoutEffect?**  
A: `useEffect` runs after paint (async), `useLayoutEffect` runs synchronously after DOM mutations but before paint; use for layout reads/writes to avoid flicker.

```jsx
React.useLayoutEffect(() => {
  const rect = el.current.getBoundingClientRect();
  // write layout synchronously
}, []);
```

**Q: What problems do useMemo/useCallback solve?**  
A: Avoid expensive recalculations and avoid re-creating callback identities for memoized children. Overuse is an anti-pattern—profile first.

**Q: How to write a robust custom hook?**  
A: Encapsulate state + effects; accept inputs as parameters; handle edge-cases (cleanup, race conditions), return a stable API.

```jsx
function useDebounce(value, delay = 300) {
  const [v, setV] = React.useState(value);
  React.useEffect(() => {
    const id = setTimeout(() => setV(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return v;
}
```

---

## Rendering & Reconciliation

**Q: How does React decide to re-render?**  
A: Any state/prop/context change in a component triggers a render for that component and its children by default. Memoized children may bail out if props are shallow-equal.

**Q: What does React.memo do?**  
A: Wraps a component to memoize its render output based on shallow props comparison.

```jsx
const Row = React.memo(function Row({ item, onSelect }) {
  return <li onClick={() => onSelect(item.id)}>{item.name}</li>;
});
```

**Q: Why can using array index as key be harmful?**  
A: Reordering causes mismatched instances (focus, input caret, animations, local state).

---

## State Management

**Q: When to choose Redux (Toolkit) vs Context vs local state?**  
A: Prefer local state/colocation until you feel cross-cutting concerns, derived state across distant nodes, or debugging/time-travel needs. Context for stable globals; Redux Toolkit or lightweight stores (Zustand) for frequent updates or complex graphs.

**Q: How to avoid “prop drilling” without global re-renders?**  
A: Split contexts, memoize providers’ value, or use external stores with selectors.

---

## Performance

**Q: How to diagnose and fix a slow list?**  
A: Profile; apply windowing (`react-window`), memoize rows, move expensive calculations outside render, avoid re-creating handlers, and ensure stable keys.

**Q: useTransition use case?**  
A: Mark updates as non-urgent (e.g., filtering huge lists) so React can keep the UI responsive.

```jsx
const [isPending, startTransition] = React.useTransition();
const onFilter = (q) => startTransition(() => setQuery(q));
```

**Q: What is tearing and how to avoid it?**  
A: UI shows inconsistent snapshots from different states. External stores or strict mode can reveal it; libraries with concurrent-safe APIs (e.g., React Redux v8+, Zustand with selectors) mitigate it.

---

## Forms

**Q: Validation strategies?**  
A: Synchronous rules, async server validation, schema-based (Zod/Yup), show errors on blur/submit, prevent layout shift, announce errors for screen readers.

**Q: How to handle file inputs or uncontrolled parts?**  
A: Use refs, FormData, and progressive enhancement.

---

## Async & Data Fetching

**Q: Why React Query/SWR over hand-rolled fetch?**  
A: Cache, dedupe, background revalidation, mutation states, retry, pagination, query keys, and optimistic updates—less glue code, fewer bugs.

**Q: Abort fetch in effects?**  
A: Use `AbortController` and cleanup to avoid setting state on unmounted component.

---

## Suspense & Concurrent

**Q: How does Suspense for data work conceptually?**  
A: A read throws a promise; boundary catches and shows fallback until it resolves. Libraries provide resource APIs to integrate cleanly.

**Q: Why is Suspense powerful?**  
A: Declarative loading states, streaming, and higher-quality UX with fewer “isLoading” branches.

---

## SSR/SSG/Next.js

**Q: Differences: CSR vs SSR vs SSG vs ISR?**  
A: CSR renders on client; SSR renders per request; SSG builds at compile time; ISR revalidates periodically. Trade-offs: TTFB, freshness, infra cost.

**Q: Hydration issues?**  
A: Mismatch between server and client render; ensure deterministic output and avoid `window` during SSR.

---

## Testing

**Q: RTL philosophy?**  
A: Test behavior via user-facing queries (`getByRole`, `getByText`) rather than implementation details (`className`, internal state).

**Q: Mocking fetch vs MSW?**  
A: MSW simulates real network layer, making tests closer to production.

---

## Accessibility

**Q: How to make a custom component accessible?**  
A: Use semantic roles/ARIA, keyboard navigation, focus management, and announce dynamic updates with `aria-live` when appropriate.

---

## TypeScript

**Q: Typing children and events?**  
A: `children: React.ReactNode`. For events: `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent<HTMLButtonElement>` etc.

```tsx
type InputProps = {
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
};
```

---

## Security

**Q: When to use dangerouslySetInnerHTML?**  
A: Only for sanitized/trusted content; otherwise XSS risk. Prefer server-side sanitization and CSP.

---

## Architecture & Patterns

**Q: Container/Presentational split still useful?**  
A: As a mental model: co-locate stateful containers with dumb presentational components. With hooks, custom hooks often replace heavy containers.

**Q: Render props vs HOC vs hooks?**  
A: Hooks provide the cleanest composition now; render props and HOCs still valid for specific libray APIs or cross-cutting concerns.

**Q: Feature-based foldering?**  
A: Group by feature/module; co-locate tests, styles, and subcomponents. Keep shared layers small and intentional.

---

## Rapid Fire

- What causes an infinite re-render loop with useEffect? **Setting state unconditionally without a guard or missing deps.**
- How to keep a callback stable without stale data? **useCallback + include deps; or ref pattern to latest value.**
- Why might `React.memo` not help? **Props always new or expensive comparison > render cost.**
- How to make a component controlled and uncontrolled? **Provide both `value/onChange` and `defaultValue`; warn when switching.**
- What is the cleanup timing for effects? **Before next effect run and on unmount.**
- How to debounce user input? **custom hook or libraries; avoid debouncing in render.**
- Prevent expensive renders on scroll? **`requestAnimationFrame`, passive listeners, or virtualization.**


## Controlled vs Uncontrolled Components

| Type | Controlled | Uncontrolled |
|------|-------------|--------------|
| Data Source | React state | DOM element |
| Update method | onChange + setState | direct DOM mutation / ref |
| Triggers re-render | ✅ Yes | ❌ No |
| Use case | Validation, live updates | Simple forms, file inputs |

Example Controlled:
```tsx
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />
```

Example Uncontrolled:
```tsx
const inputRef = useRef();
<input ref={inputRef} defaultValue="Apple" />
<button onClick={() => alert(inputRef.current.value)}>Submit</button>
```

**Why Uncontrolled?**
- No re-render per keystroke → better performance for large forms.
- Required for file inputs (cannot be controlled).
- Works with legacy DOM libraries (e.g. jQuery, Bootstrap).

---

## Shallow Compare in React
React performs shallow equality to detect changes efficiently.  
Objects/arrays compared by reference, not structure.

```js
const obj1 = { a: 1 }; const obj2 = { a: 1 };
obj1 === obj2 // false
```

Use `useMemo` to keep reference stable:
```js
const memoizedObj = useMemo(() => ({ a: 1 }), []);
```

---

## Why Use Uncontrolled Components
- **Performance**: DOM handles input updates.  
- **Simplicity**: Only read values when submitting.  
- **Compatibility**: Needed when integrating DOM-heavy third-party UI libs.

Hybrid forms (some controlled, some uncontrolled) often work best.

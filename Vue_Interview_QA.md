# Vue Interview Q&A — Ultra‑Detailed (Vue 3 + Vue 2, with React Comparisons)

---

## Core & Ecosystem

**Q1. What are the biggest changes in Vue 3 vs 2, and why do they matter?**  
**Answer:** Vue 3 replaced getter/setter reactivity with **Proxy‑based** reactivity, improving performance and enabling true detection of property additions/deletions. It introduced the **Composition API**, which organizes logic by **feature** rather than option blocks, enabling clean **composables** and better TypeScript support. New features include **Fragments** (multiple roots), **Teleport** (portals), and **Suspense** (async trees). Result: more scalable codebases and better DX while keeping backward compatibility with the Options API.

**Q2. Compare Vue’s reactivity to React’s rendering model.**  
**Answer:** Vue tracks dependencies at the **property level** and only re‑renders parts of the template that depend on changed values. React generally re‑renders the component and reconciles the virtual DOM, requiring techniques like `memo`, `useMemo`, and `useCallback` to prevent unnecessary renders. Vue uses **computed** for cached derivations and **watch/watchEffect** for side‑effects, reducing manual memoization needs.

---

## Templates & Directives

**Q3. `v-if` vs `v-show` — when to use each?**  
**Answer:** `v-if` adds/removes elements from the DOM; use it for rarely shown content or expensive trees. `v-show` toggles `display: none` while keeping elements in the DOM; use it for frequent toggles (tab content).

**Q4. Why are `:key`s important in `v-for`? What makes a good key?**  
**Answer:** Keys uniquely identify items, allowing correct DOM diffing and preventing incorrect state reuse (e.g., wrong input values). Use stable IDs from data, not array indices, to avoid reorder bugs.

**Q5. How do class/style bindings work?**  
**Answer:** You can bind arrays/objects: `:class="{ active:isActive }"`, `:class="['a', cond && 'b']"`, `:style="{ color, fontSize:size+'px' }"`. These bindings recompute as reactive dependencies change.

---

## Reactivity & Composition

**Q6. When to choose `ref` vs `reactive` vs `computed`?**  
**Answer:** `ref` for standalone values (numbers/strings/booleans/any), `reactive` for objects/arrays where you want deep reactivity, and `computed` for read‑only derived values (cached). Prefer `computed` over `watch` when deriving values, and `watch`/`watchEffect` for side‑effects.

**Q7. `watch` vs `watchEffect` and debugging guidance?**  
**Answer:** Use `watch` for **explicit sources** and need old/new values. Use `watchEffect` to auto‑track anything used inside the callback (quick side‑effects), but it can be harder to debug because dependencies are implicit. For production‑critical logic, prefer explicit `watch` with clear sources.

**Q8. How do composables compare to React hooks?**  
**Answer:** They’re nearly analogous. Composables are plain functions that return refs/computed/actions. They must be called during setup (or another composable). Unlike React’s rules of hooks, Vue relies on the setup call order without linter rules, but the pattern is similar in practice.

**Q9. Common reactivity pitfalls and fixes?**  
**Answer:** Forgetting `.value` in script; destructuring a `reactive` object (loses reactivity) — use `toRefs`; Vue 2 array index replacement not reactive — use `splice` or `$set`; watchers without `flush` mode causing timing issues — consider `flush:'post'` for DOM‑after effects.

---

## Lifecycle & Patterns

**Q10. Where do you fetch data?**  
**Answer:** Typically in `onMounted` or as part of a composable (e.g., `useFetch`). For route transitions, use router guards or lazy load the route component and fetch on mount. For SSR with Nuxt, use `useFetch/asyncData` and return server‑rendered content for hydration.

**Q11. How do you handle cleanup?**  
**Answer:** `onUnmounted` for listeners/timers; watchers can return a cleanup function. Always clean up to prevent memory leaks across route changes.

---

## Props, Emits, and `v-model`

**Q12. Implementing `v-model` on a custom component?**  
**Answer:** Accept a `modelValue` prop and emit `update:modelValue` with the new value. For multiple models use `v-model:name` and emit `update:name`.

**Q13. Provide/Inject — when and why?**  
**Answer:** For hierarchical dependencies (theme, form context), avoiding prop drilling. The provided value should be reactive to update injectors. Not a replacement for global state — use Pinia for app‑wide data.

---

## Routing & Guards

**Q14. Route params vs query strings?**  
**Answer:** Use params for resource identity (`/users/:id`) and query strings for filters/sorting (`?q=vue&page=2`). Prefer `props:true` in route definitions so components receive typed props rather than using `$route` directly.

**Q15. How to protect routes?**  
**Answer:** Use `beforeEach` or per‑route `beforeEnter` guards to check auth and redirect. Ensure guards are fast or do early skeleton UI + fetch on mount to avoid jank.

---

## State Management

**Q16. Pinia vs Vuex?**  
**Answer:** Pinia is the recommended store for Vue 3: simple API, full TypeScript, no required mutations. Vuex remains in legacy apps. Prefer Pinia unless team standards dictate otherwise.

**Q17. State persistence and security?**  
**Answer:** Persist with plugins or manual `watch` to localStorage/sessionStorage. Avoid storing sensitive tokens in localStorage; prefer http‑only cookies or secure storage patterns.

---

## Forms & Validation

**Q18. How do you validate forms?**  
**Answer:** Libraries like **vee‑validate** + **yup/zod** are common. Or roll your own with computed errors and `v-model` modifiers. Debounce async validators to avoid overfetching.

**Q19. What do `.lazy`, `.trim`, `.number` do?**  
**Answer:** `.lazy` updates on change rather than input, `.trim` trims whitespace, `.number` casts to number (if parseable).

---

## Performance & Optimization

**Q20. How to optimize a large data table with filters and sorting?**  
**Answer:** Use computed chains to derive a sorted/filtered view, avoid mutating the source in place, paginate, virtualize long lists, and use stable keys. When embedding heavy objects, store them in `markRaw` or `shallowRef` to avoid deep tracking.

**Q21. When to use `watchEffect` vs `computed`?**  
**Answer:** `computed` for pure derived values; `watchEffect` for side‑effects. Don’t mutate state inside computed — it should be pure; otherwise you risk infinite loops.

---

## TypeScript & Testing

**Q22. Typing `defineProps/defineEmits`?**  
**Answer:** Supply generic/inline type: `defineProps<{ id:number }>()`. For emits: `defineEmits<{ (e:'save', id:number): void }>()`. This ensures component contracts are checked by TS.

**Q23. Testing with Pinia/Router dependencies?**  
**Answer:** Mount with global plugins: `global: { plugins: [createPinia(), router] }`. Use `createTestingPinia` to stub stores. Stub network with `fetch-mock` or MSW. Assert emitted events and text content.

---

## SSR/SSG & Nuxt

**Q24. What benefits does Nuxt 3 bring?**  
**Answer:** File‑based routing, `useFetch/asyncData` for server/client data, Nitro server, auto imports for composables, first‑class DX for SSR/SSG, and strong TS support — comparable to Next.js in the React world.

---

## Security & Accessibility

**Q25. How does Vue help prevent XSS?**  
**Answer:** Interpolations escape by default. `v-html` renders raw HTML and must be sanitized. Avoid binding untrusted values directly into attributes that can execute code; use CSP where possible.

**Q26. Accessible modals with Teleport?**  
**Answer:** Render modal to `body`, add `role="dialog"` + `aria-modal="true"`, manage focus trap, restore focus on close, hide background from screen readers (aria‑hidden or inert), and support Esc to close.

---

## React Comparisons

**Q27. Key developer‑experience differences switching from React?**  
**Answer:** Vue’s templates and directives reduce boilerplate; **computed** replaces many memo needs; `v-model` standardizes two‑way binding; **Pinia** is simpler than Redux; fewer explicit memoizations. JSX is possible in Vue but templates are idiomatic and better supported by tooling.

**Q28. Using JSX with Vue — when and how?**  
**Answer:** Add `@vitejs/plugin-vue-jsx`. JSX is useful for render‑function heavy components or when porting React code. Most teams still prefer templates for readability and designer‑friendliness.

---

## Troubleshooting & Edge Cases

**Q29. Why did destructuring my `reactive` object break updates?**  
**Answer:** Destructuring returns plain values. Use `toRefs`/`toRef` to keep reactivity: `const { a, b } = toRefs(state)`.

**Q30. Why doesn’t `arr[index] = x` update in Vue 2?**  
**Answer:** Vue 2 can’t observe index assignments; use `this.$set(arr, index, x)` or `arr.splice(index, 1, x)` instead. Vue 3 fixes this via proxies.

---

(If an interviewer goes deep on any topic above, ask for clarification and tailor examples to their stack: Pinia vs Vuex, Router guards, or Nuxt.)


# Vue Knowledge Points — Ultra‑Detailed (Vue 3 + Vue 2, with React Comparisons)

Audience: React‑heavy engineer learning Vue quickly for interviews.  
Scope: **Vue 3 (Composition API)** first; **Vue 2 (Options API)** notes; **React comparisons** per topic; code you can paste into a Vite project.

---

## 1) Mental Model & Ecosystem

### What & How
- **Progressive framework**: adopt via `<script>` tag, then scale to SPA with router + store.
- **Core**: Vue runtime (reactivity + renderer), template compiler, devtools.
- **Official libs**: `vue-router`, **Pinia** (state, Vue 3), **Vuex** (legacy), **vueuse** (community composables), **Nuxt** (SSR/SSG).
- **Vue 3 improvements**: proxy‑based reactivity, Composition API, Fragments/Teleport/Suspense, better TS support.

### Why it matters (interview)
- You’ll be asked “Vue vs React” and “Composition vs Options”. Knowing the **reactivity model** and **SFC ergonomics** shows seniority.

### React comparison
- Templates & directives replace much JSX ceremony; **computed** removes many `useMemo` cases; **watch/watchEffect** vs `useEffect`.

---

## 2) Project Setup & Tooling

### Vue 3 + Vite (recommended)
```bash
npm create vite@latest vue3-demo -- --template vue-ts
cd vue3-demo && npm i && npm run dev
```
`main.ts`
```ts
import { createApp } from 'vue'
import App from './App.vue'
createApp(App).mount('#app')
```

### Vue 2 (when needed)
```bash
npm i vue@^2.7 @vitejs/plugin-vue2 vue-template-compiler -D
# Vite config uses plugin-vue2
```

### Pitfalls
- Mismatched versions (router 4 is for Vue 3; router 3 for Vue 2).  
- TypeScript: use Volar extension (replaces Vetur).

---

## 3) Single‑File Components (SFC)

### Anatomy
- `<template>`: declarative HTML‑like DSL.
- `<script setup>`: compile‑time sugar that auto‑exposes bindings.
- `<style scoped>`: CSS scoped to component (attribute selectors).

### Vue 3 Example (`<script setup>`)
```vue
<template>
  <h1>{{ title }}</h1>
  <button @click="inc">Count: {{ count }}</button>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const title = 'Hello Vue 3'
const count = ref(0)
function inc(){ count.value++ }
</script>

<style scoped>
button { padding:.5rem 1rem; }
</style>
```

### Vue 2 Equivalent (Options API)
```vue
<script>
export default {
  data(){ return { title: 'Hello Vue 2', count: 0 } },
  methods:{ inc(){ this.count++ } }
}
</script>
```

### Why it matters
- SFCs co‑locate template/logic/styles → readable code; `<script setup>` reduces boilerplate dramatically.

### React comparison
- React collocates JSX+logic+CSS Modules; Vue’s **scoped CSS** and **template** are first‑class without extra libs.

---

## 4) Template Syntax & Directives

### Basics
- Interpolation: `{{ expr }}`.
- Bindings: `:prop="expr"` / `v-bind:prop="expr"`.
- Events: `@click="fn"` / `v-on:click="fn"` with **modifiers** (`@submit.prevent`).

### Conditionals & Loops
```vue
<p v-if="ok">Shown</p><p v-else>Hidden</p>
<li v-for="(item,i) in items" :key="item.id">{{ i }}: {{ item.name }}</li>
```

### Classes & Styles
```vue
<div :class="['btn', { active: isActive }]" :style="{ color: colorVar }"></div>
```

### `v-model`
```vue
<input v-model.trim="name">
<select v-model="country"><option v-for="c in countries" :key="c" :value="c">{{ c }}</option></select>
```

### Pitfalls
- Forgetting stable `:key` in lists → DOM reuse bugs.  
- Using index as key → animation/control issues.

### React comparison
- Vue directives map to JSX idioms (`{cond && <X/>}`, `.map`, controlled inputs). `v-model` reduces boilerplate relative to React’s `value/onChange` pairing.

---

## 5) Reactivity Deep Dive

### Core APIs
- **`ref(value)`** → `{ value }` (for primitives or any value).
- **`reactive(obj)`** → proxy for objects/arrays (deeply reactive).
- **`computed(getter)`** → cached derivation.
- **`watch(source, cb)`** → observe explicit sources; **`watchEffect`** → auto‑track deps used inside.

```ts
import { ref, reactive, computed, watch, watchEffect } from 'vue'
const count = ref(0)
const state = reactive({ filter:'', items:[1,2,3] })
const doubled = computed(() => count.value * 2)
watch(() => state.filter, (nv) => { /* fetch */ })
watchEffect(() => console.log('count is', count.value))
```

### Advanced
- `toRef/toRefs` to preserve reactivity on destructuring.
- `shallowRef/shallowReactive` for large structures (track top‑level only).
- `markRaw` to exclude from reactivity (e.g., 3rd‑party instances).

### Vue 2 caveats
- Property addition/deletion not reactive (use `Vue.set` or `this.$set`).  
- Array index assignment not reactive: use `splice(index,1,newVal)`.

### Pitfalls
- Forgetting `.value` in `<script>` (templates unwrap automatically).  
- Deep watchers firing too often; prefer computed + watch precise sources.

### React comparison
- React re‑renders components on setState; Vue patches only where a dependency changed → fewer explicit memoizations.

---

## 6) Composition API vs Options API

### Composition API (`setup` / `<script setup>`)
- Organize by **feature** using **composables** (functions that return reactive state/actions).

```ts
// useFetch.ts
import { ref, onMounted } from 'vue'
export function useFetch<T>(url: string){
  const data = ref<T|null>(null), error = ref<any>(null), loading = ref(false)
  onMounted(async () => {
    loading.value=true
    try{ const r = await fetch(url); data.value = await r.json() } catch(e){ error.value=e }
    finally{ loading.value=false }
  })
  return { data, error, loading }
}
```

### Options API
- Organize by `data/methods/computed/watch`. Easier for beginners, verbose for cross‑cutting concerns.

### Why it matters
- Composition scales in large codebases and interviews often ask to “extract logic into a composable”.

### React comparison
- Composables ≈ custom hooks; no rules‑of‑hooks name/enforcement, but must run during setup.

---

## 7) Lifecycle

### Vue 3 hooks
`onBeforeMount`, `onMounted`, `onBeforeUpdate`, `onUpdated`, `onBeforeUnmount`, `onUnmounted`, `onErrorCaptured`.

```ts
import { onMounted, onUnmounted } from 'vue'
function onResize(){ /* ... */ }
onMounted(() => window.addEventListener('resize', onResize))
onUnmounted(() => window.removeEventListener('resize', onResize))
```

### Vue 2
`created`, `mounted`, `updated`, `destroyed`, …

### React comparison
- Most `useEffect` use‑cases map to `onMounted` + `onUnmounted` or to `watch/watchEffect` for reactive sources.

---

## 8) Component Communication

### Parent → Child: **props**  
### Child → Parent: **emits/events**  
### Two‑way: **`v-model` on custom components**  
### Deep tree: **provide/inject**  
### App‑wide: **Pinia/Vuex**

```vue
<!-- Child.vue -->
<script setup lang="ts">
const props = defineProps<{ modelValue: string }>()
const emit = defineEmits<{ (e:'update:modelValue', v:string):void }>()
const onI = (e:Event) => emit('update:modelValue', (e.target as HTMLInputElement).value)
</script>
<template><input :value="props.modelValue" @input="onI" /></template>
```

### Why it matters
- Implementing `v-model` correctly is a frequent interview task.

### React comparison
- React uses callbacks (`onChange`) + controlled props; Vue standardizes two‑way contract via `modelValue` + `update:modelValue`.

---

## 9) Slots, Scoped Slots, Teleport, Suspense

### Slots
- Content projection: default and named slots.

### Scoped slots
- Child exposes data to slot content.

```vue
<!-- List.vue -->
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      <slot name="row" :item="item">{{ item.text }}</slot>
    </li>
  </ul>
</template>
<script setup>defineProps<{ items: Array<{id:number,text:string}> }>()</script>

<!-- Parent.vue -->
<List :items="todos">
  <template #row="{ item }"><strong>{{ item.text }}</strong></template>
</List>
```

### Teleport & Suspense (Vue 3)
- **Teleport**: render to another DOM container (modals).  
- **Suspense**: async component trees with fallback while waiting.

### React comparison
- Slots ≈ children/render props. Teleport ≈ `createPortal`. Suspense exists in both ecosystems (data patterns differ).

---

## 10) Router (vue‑router)

### Vue 3 (router v4)
```ts
import { createRouter, createWebHistory } from 'vue-router'
const routes = [
  { path:'/', component:()=>import('./pages/Home.vue') },
  { path:'/users/:id', name:'user', component:()=>import('./pages/User.vue'), props:true }
]
export const router = createRouter({ history:createWebHistory(), routes })
```

### Guards
```ts
router.beforeEach((to) => {
  const authed = !!localStorage.getItem('auth')
  if(to.meta.requiresAuth && !authed) return { name:'login', query:{ redirect: to.fullPath } }
})
```

### React comparison
- Similar to React Router. `<router-link>` and `<router-view>` provide declarative integration in templates.

---

## 11) State Management (Pinia, Vuex)

### Pinia (Vue 3)
```ts
// stores/cart.ts
import { defineStore } from 'pinia'
export const useCart = defineStore('cart', {
  state: () => ({ items: [] as {id:number, price:number, qty:number}[] }),
  getters: { total: (s) => s.items.reduce((a,i)=>a+i.price*i.qty,0) },
  actions: {
    add(it:{id:number,price:number}){ const x=this.items.find(i=>i.id===it.id); x? x.qty++: this.items.push({...it,qty:1}) }
  }
})
```
`main.ts`
```ts
import { createPinia } from 'pinia'
app.use(createPinia())
```

### Vuex
- Flux‑like with mutations/actions; still fine for legacy apps.

### React comparison
- Pinia ≈ Zustand/Easy‑Peasy; Vuex ≈ Redux with less ceremony.

---

## 12) Forms & Validation

### `v-model` modifiers
`.trim`, `.number`, `.lazy`.

```vue
<input v-model.number="age">
```

### Validation libs
- **vee‑validate** + **yup/zod**; or manual computed error objects.

### Example (manual)
```vue
<script setup>
import { ref, computed } from 'vue'
const name = ref(''), age = ref<number|null>(null)
const errors = computed(() => ({
  name: !name.value ? 'Required' : null,
  age: age.value==null || age.value<0 ? 'Invalid age' : null
}))
const valid = computed(() => !errors.value.name && !errors.value.age)
</script>
```

---

## 13) Async Data Patterns

- Fetch in `onMounted` or in router guards or via composables.  
- Handle cancellation with `AbortController`; debounce watchers for search.

```ts
const q = ref(''), data = ref<any>(null), loading = ref(false), error = ref<any>(null)
let ctrl:AbortController|null = null
watch(q, async () => {
  ctrl?.abort(); ctrl = new AbortController()
  try{ loading.value=true; const r=await fetch('/api?q='+q.value,{signal:ctrl.signal}); data.value=await r.json() }
  catch(e){ if((e as any).name!=='AbortError') error.value=e }
  finally{ loading.value=false }
})
```

---

## 14) Performance

- Stable `:key`, avoid giant reactive objects (use `markRaw`/`shallowRef`).  
- Use **computed** for derivations; lazy routes; `<keep-alive>` for tab caches.  
- Virtualize long lists.  
- In Vue 2 watch object/array caveats.

---

## 15) Transitions

```vue
<transition name="fade">
  <p v-if="show">Hello</p>
</transition>
<style scoped>
.fade-enter-from,.fade-leave-to{ opacity:0 }
.fade-enter-active,.fade-leave-active{ transition: opacity .2s ease }
</style>
```

---

## 16) TypeScript

- `<script setup lang="ts">`, typed `defineProps/defineEmits`, generics in composables.

```vue
<script setup lang="ts">
const props = defineProps<{ id:number, name?:string }>()
const emit = defineEmits<{ (e:'select', id:number): void }>()
</script>
```

---

## 17) Testing (Vue Test Utils + Vitest/Jest)

```ts
import { mount } from '@vue/test-utils'
import Counter from './Counter.vue'
test('increments', async () => {
  const w = mount(Counter)
  await w.get('button').trigger('click')
  expect(w.text()).toContain('Count: 1')
})
```

---

## 18) SSR/SSG & Nuxt 3
- File‑based routing, `useFetch`, server routes, hybrid rendering.

---

## 19) Accessibility & i18n
- ARIA labels, focus management for modals (with Teleport), keyboard handlers.  
- `vue-i18n` for translations; watch RTL/layout issues.

---

## 20) Error Handling
- `app.config.errorHandler`, `onErrorCaptured`, router error hooks; guard against unhandled promise rejections.

---

## 21) Migration Vue 2 → 3 & React‑to‑Vue Map
- Many APIs compatible; replace filters with computed/methods; global API changes (`app.use`).  
- Map: `useState`→`ref/reactive`, `useMemo`→`computed`, `useEffect`→`watch/watchEffect`/lifecycle, Context→provide/inject/Pinia, Portal→Teleport.

---

**Practice:** Convert a React search component to Vue with a composable and Suspense.  

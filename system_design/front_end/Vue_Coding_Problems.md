# Vue Coding Problems — Ultra‑Detailed (Vue 3 focus + Vue 2 notes)

Each task includes **goal**, **why**, **solution**, **complexity**, and **pitfalls**. Implement with Vite + Vue 3; Vue 2 variants noted where useful.

---

## 1) Counter with `v-model` + Step + Reset
**Goal:** Component supports `v-model`, `step` prop, and reset.  
**Why:** Tests custom `v-model` and emits.

```vue
<!-- Counter.vue -->
<template>
  <div class="counter">
    <button @click="dec">-{{ step }}</button>
    <span>Count: {{ modelValue }}</span>
    <button @click="inc">+{{ step }}</button>
    <button @click="$emit('update:modelValue', 0)">Reset</button>
  </div>
</template>

<script setup lang="ts">
const props = defineProps<{ modelValue:number, step?:number }>()
const emit = defineEmits<{ (e:'update:modelValue', v:number):void }>()
const step = props.step ?? 1
function inc(){ emit('update:modelValue', props.modelValue + step) }
function dec(){ emit('update:modelValue', props.modelValue - step) }
</script>

<style scoped>.counter{ display:flex; gap:.5rem; align-items:center; }</style>
```

**Complexity:** O(1).  
**Pitfalls:** Forgetting to emit `update:modelValue`; mixing controlled state internally.

---

## 2) Todo List with Filters + LocalStorage
**Goal:** Add/toggle/delete; computed filters; persist.  
**Why:** Demonstrates lists, computed, watchers.

```vue
<script setup lang="ts">
import { ref, computed, watch } from 'vue'
type Todo = { id:number, text:string, done:boolean }
const todos = ref<Todo[]>(JSON.parse(localStorage.getItem('todos')||'[]'))
const filter = ref<'all'|'active'|'done'>('all')
const visible = computed(() => todos.value.filter(t =>
  filter.value==='all' ? true : filter.value==='active' ? !t.done : t.done))
watch(todos, v => localStorage.setItem('todos', JSON.stringify(v)), { deep:true })
function add(text:string){ todos.value.push({ id:Date.now(), text, done:false }) }
function toggle(id:number){ const t=todos.value.find(t=>t.id===id)!; t.done=!t.done }
function remove(id:number){ todos.value = todos.value.filter(t=>t.id!==id) }
</script>
```

**Complexity:** O(n).  
**Pitfalls:** Missing keys; forgetting deep watcher.

---

## 3) Debounced Search Composable with Cancellation
**Goal:** Debounce input, cancel outdated requests.  
**Why:** Realistic search UX.

```ts
// useDebouncedFetch.ts
import { ref, watch } from 'vue'
export function useDebouncedFetch(urlOf:(q:string)=>string, delay=300){
  const q=ref(''), data=ref<any>(null), error=ref<any>(null), loading=ref(false)
  let t:any, ctrl:AbortController|null=null
  watch(q, () => {
    clearTimeout(t)
    t=setTimeout(async () => {
      ctrl?.abort(); ctrl = new AbortController()
      loading.value=true; error.value=null
      try{ const r=await fetch(urlOf(q.value), {signal:ctrl.signal}); data.value=await r.json() }
      catch(e){ if((e as any).name!=='AbortError') error.value=e }
      finally{ loading.value=false }
    }, delay)
  })
  return { q, data, error, loading }
}
```

**Pitfalls:** Not handling AbortError; stale results overwriting fresh ones.

---

## 4) Modal with Teleport, Esc, and Focus Restore
**Goal:** Accessible modal.  
**Why:** Portals + a11y.

```vue
<!-- Modal.vue -->
<template>
  <teleport to="body">
    <div v-if="open" class="backdrop" @click.self="close">
      <div class="dialog" role="dialog" aria-modal="true" tabindex="-1" ref="box" @keydown.esc="close">
        <slot/><button @click="close">Close</button>
      </div>
    </div>
  </teleport>
</template>
<script setup>
import { ref, watch } from 'vue'
const props = defineProps<{ open:boolean }>()
const emit = defineEmits<{ (e:'close'):void }>()
const box = ref<HTMLElement|null>(null)
let last:HTMLElement|null=null
function close(){ emit('close') }
watch(() => props.open, (o) => {
  if(o){ last=document.activeElement as HTMLElement; queueMicrotask(()=>box.value?.focus()) }
  else{ last?.focus() }
})
</script>
<style scoped>
.backdrop{ position:fixed; inset:0; background:rgba(0,0,0,.45); display:flex; align-items:center; justify-content:center }
.dialog{ background:#fff; padding:1rem; min-width:300px }
</style>
```

---

## 5) Protected Nested Routes
```ts
// router.ts
router.beforeEach(async (to) => {
  const authed = !!localStorage.getItem('auth')
  if(to.meta.requiresAuth && !authed) return { name:'login', query:{ redirect: to.fullPath } }
})
```

**Pitfalls:** Infinite loops; not returning a navigation object.

---

## 6) Pinia Cart Store with Derived Total
```ts
export const useCart = defineStore('cart', {
  state: () => ({ items: [] as {id:number, price:number, qty:number}[] }),
  getters: { total: (s) => s.items.reduce((a,i)=>a+i.price*i.qty, 0) },
  actions: {
    add(p:{id:number,price:number}){ const x=this.items.find(i=>i.id===p.id); x? x.qty++ : this.items.push({...p,qty:1}) },
    remove(id:number){ this.items = this.items.filter(i=>i.id!==id) }
  }
})
```

**Pitfalls:** Mutating state outside actions in strict mode.

---

## 7) Custom Input with Validation & `valid` Event
```vue
<script setup lang="ts">
const props = defineProps<{ modelValue:string, required?:boolean }>()
const emit = defineEmits<{ (e:'update:modelValue', v:string):void, (e:'valid', ok:boolean):void }>()
function onI(e:Event){ const v=(e.target as HTMLInputElement).value; emit('update:modelValue', v); emit('valid', !!v || !props.required) }
</script>
<template>
  <div><input :value="modelValue" @input="onI"><small v-if="required && !modelValue">Required</small></div>
</template>
```

---

## 8) Infinite Scroll with IntersectionObserver
(See earlier pattern; remember to `disconnect()` on unmount).

---

## 9) Error Boundary Component
```vue
<script setup>
import { ref, onErrorCaptured } from 'vue'
const err = ref<any>(null)
onErrorCaptured(e => { err.value = e; return false })
</script>
<template><div v-if="err">Something went wrong</div><slot v-else/></template>
```

---

## 10) Reactive Table (Sort + Filter)
```vue
<script setup>
import { ref, computed } from 'vue'
const rows = ref([{id:1,name:'A'},{id:2,name:'C'},{id:3,name:'B'}])
const q = ref(''); const sortAsc = ref(true)
const view = computed(() => rows.value
  .filter(r => r.name.toLowerCase().includes(q.value.toLowerCase()))
  .slice().sort((a,b)=> sortAsc.value? a.name.localeCompare(b.name): b.name.localeCompare(a.name)))
</script>
```

---

## 11) Vue 2 Variants (Notes)
- Custom `v-model`: use `value` prop + `input` event.  
- Array updates: `this.$set(arr, index, val)` or `arr.splice(index, 1, val)`.  
- Provide/inject available (since 2.2); Composition API via plugin in 2.7 if needed.

---

**Study plan:** Implement each problem twice (Composition & Options) + add tests with VTU; wire router/Pinia in a small demo app.

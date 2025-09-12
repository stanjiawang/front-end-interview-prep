# Redux Front-End Interview Q&A

## 1. What is Redux and why use it?
Redux is a predictable state container for JavaScript apps.  
You use it when you have **shared state** across components, **complex state transitions**, or need **debuggable, testable state management**.

---

## 2. Core Principles of Redux
1. **Single Source of Truth** – one store holds the entire app state  
2. **State is Read-Only** – state updates via dispatched actions  
3. **Changes via Pure Functions** – reducers must be pure

---

## 3. Redux Data Flow
Redux follows a strict **unidirectional data flow**:

```
View -> dispatch(action) -> Reducer -> Store -> View re-renders
```

---

## 4. What is a Reducer?
A reducer is a pure function that takes current state + action → returns new state.

```ts
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { value: state.value + 1 }
    default:
      return state
  }
}
```

---

## 5. What is an Action?
An action is a plain JS object with a `type` field (and optional `payload`).

```ts
const increment = { type: 'INCREMENT' }
const addAmount = { type: 'ADD', payload: 5 }
```

---

## 6. Redux Toolkit Example (Modern Redux)
```ts
import { configureStore, createSlice } from '@reduxjs/toolkit'

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: state => { state.value++ },
    decrement: state => { state.value-- },
    addByAmount: (state, action) => { state.value += action.payload }
  }
})

export const { increment, decrement, addByAmount } = counterSlice.actions

const store = configureStore({ reducer: counterSlice.reducer })

store.dispatch(increment())  // value = 1
```

---

## 7. What is Middleware?
Middleware sits between dispatching an action and the moment it reaches the reducer.

```ts
const logger = store => next => action => {
  console.log('dispatching', action)
  return next(action)
}
```

---

## 8. How to Handle Async Operations?
Use **Redux Thunk** or **RTK createAsyncThunk**.

```ts
import { createAsyncThunk } from '@reduxjs/toolkit'

export const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  const res = await fetch(`/api/users/${id}`)
  return res.json()
})
```

---

## 9. What is `combineReducers`?
It merges multiple slice reducers into one root reducer.

```ts
import { combineReducers } from '@reduxjs/toolkit'
const rootReducer = combineReducers({ auth: authReducer, posts: postsReducer })
```

---

## 10. What is `useSelector` and `useDispatch`?
- `useSelector`: Selects part of the state tree  
- `useDispatch`: Returns store’s dispatch function

```tsx
import { useSelector, useDispatch } from 'react-redux'
const count = useSelector(state => state.counter.value)
const dispatch = useDispatch()
dispatch(increment())
```

---

## 11. How to Optimize Performance?
- Use **multiple slices** for state separation  
- Use **memoized selectors** (e.g., Reselect)  
- Avoid selecting large objects → select primitive values/arrays  
- Keep reducers pure so references don’t change unnecessarily

---

## 12. What is RTK Query?
A data-fetching & caching solution built into Redux Toolkit.

```ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: build => ({
    getPosts: build.query({ query: () => '/posts' })
  })
})

const { data, isLoading } = api.useGetPostsQuery()
```

---

## 13. Redux vs Context API

| Redux Toolkit              | React Context |
|---------------------------|--------------|
| Predictable, single store | Simple global state |
| DevTools + Middleware     | No built-in debugging |
| Great for complex state   | Good for small apps |

---

## 14. Common Pitfalls
- ❌ **Mutating State Directly:** Reducers must not mutate state manually unless using Immer (RTK does).  
- ❌ **Async in Reducers:** Reducers must be pure — handle async in middleware or thunks.

---

## 15. Example End-to-End
```tsx
// Counter.tsx
import React from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { increment } from './counterSlice'

export function Counter() {
  const value = useSelector((s) => s.counter.value)
  const dispatch = useDispatch()

  return (
    <button onClick={() => dispatch(increment())}>
      Count: {value}
    </button>
  )
}
```

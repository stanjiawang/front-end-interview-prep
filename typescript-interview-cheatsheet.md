# 🚀 TypeScript Interview Cheatsheet

A concise guide to **common TypeScript interview topics** with the most used code snippets.

---

## 1. `type` vs `interface`

- **Interface**: best for object/class contracts, supports declaration merging.  
- **Type**: more flexible, can describe unions, tuples, primitives.

```ts
// interface
interface User {
  id: number;
  name: string;
}
interface Admin extends User {
  role: "admin";
}

// type
type ID = string | number;
type Point = [number, number];
type Status = "idle" | "loading" | "success";

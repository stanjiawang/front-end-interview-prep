# 🚀 TypeScript Interview Cheatsheet

A complete reference of **TypeScript interview concepts**, including fundamentals, advanced types, configuration, and comparisons with JavaScript.  
Includes **examples** for each concept.

---

## 📑 Table of Contents
- [Core Concepts & Fundamentals](#core-concepts--fundamentals)
  - [1. Basic Types & Type Inference](#1-basic-types--type-inference)
  - [2. Interface vs Type Alias](#2-interface-vs-type-alias)
  - [3. Functions & Generics](#3-functions--generics)
- [Advanced Types & Utilities](#advanced-types--utilities)
  - [4. Advanced Types](#4-advanced-types)
  - [5. void vs never](#5-void-vs-never)
  - [6. Type Guards & Type Checking](#6-type-guards--type-checking)
- [Modules & Configuration](#modules--configuration)
  - [7. Modules](#7-modules)
  - [8. tsconfig.json Configuration](#8-tsconfigjson-configuration)
- [TypeScript vs JavaScript](#typescript-vs-javascript)
  - [9. Key Differences](#9-key-differences)
  - [10. TypeScript in Popular Frameworks](#10-typescript-in-popular-frameworks)
- [✅ Quick Interview One-Liners](#-quick-interview-one-liners)

---

## Core Concepts & Fundamentals

### 1. Basic Types & Type Inference

- **Primitive Types**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.  
- **Arrays**: `number[]` or `Array<number>`.  
- **Tuples**: `[string, number]`.  
- **Enums**: use sparingly, prefer union literals.  
- **Any vs Unknown**:  
  - `any` disables type checking.  
  - `unknown` is safer (requires narrowing).  

```ts
// Tuple
let userProfile: [string, number, boolean];
userProfile = ['Alice', 30, true];

// Any vs Unknown
let val: unknown = 'hello';
let str1: string = val as string; // assertion required

let val2: any = 'world';
let str2: string = val2; // bypassed checks

// Enum vs Union
enum Status {
  Loading = 'LOADING',
  Success = 'SUCCESS',
  Failure = 'FAILURE',
}
type StatusUnion = 'LOADING' | 'SUCCESS' | 'FAILURE';
```

---

### 2. Interface vs Type Alias

- **Interface**: describes object/class contracts, supports merging.  
- **Type alias**: more flexible, supports unions, intersections, primitives, tuples.  

```ts
// Interface
interface User {
  name: string;
  age: number;
}
interface User { id: number; } // merges

const john: User = { name: 'John', age: 45, id: 101 };

// Type alias
type ID = string | number;
type Status = 'active' | 'inactive';
type Admin = User & { accessLevel: 'admin' | 'superadmin' };
```

---

### 3. Functions & Generics

- Function type: `(name: string) => number`.  
- Overloading: multiple call signatures.  
- **Generics**: reusable + type-safe.

```ts
// Generic function
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr.length > 0 ? arr[0] : undefined;
}

const firstNum = getFirstElement([1, 2, 3]); // number | undefined
const firstStr = getFirstElement(['a', 'b']); // string | undefined

// Generic class
class GenericList<T> {
  private items: T[] = [];
  addItem(item: T) { this.items.push(item); }
  getItems(): T[] { return this.items; }
}
```

---

## Advanced Types & Utilities

### 4. Advanced Types

- **Union**: `string | number`  
- **Intersection**: `A & B`  
- **Literal**: `'success' | 123`  

```ts
type Shape = { x: number } | { y: number };
type WithId = { id: number } & { createdAt: Date };
type Status = 'loading' | 'success' | 'error';
```

---

### 5. void vs never

- `void`: returns nothing (implicitly `undefined`).  
- `never`: function never completes (error/infinite loop).  

```ts
function logMessage(msg: string): void { console.log(msg); }

function throwError(msg: string): never { throw new Error(msg); }
function infiniteLoop(): never { while (true) {} }
```

---

### 6. Type Guards & Type Checking

- `typeof`: primitives.  
- `in`: property existence.  
- `instanceof`: prototype chain.  
- `as`: type assertion (compiler-only).  
- `is`: type predicate (runtime + narrowing).  

```ts
// typeof / in
function isVehicle(arg: any) {
  if (typeof arg === 'object' && arg !== null) {
    if ('startEngine' in arg) console.log('Has engine');
  }
}

// instanceof
class Car { drive() {} }
const myCar = new Car();
if (myCar instanceof Car) console.log('Car instance');

// is predicate
interface Cat { meow(): void }
interface Dog { bark(): void }
function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}
```

---

## Modules & Configuration

### 7. Modules

- Use `export` and `import`.  
- Resolution strategies: `node` vs `classic`.  

```ts
// utils.ts
export function sum(a: number, b: number) { return a + b; }

// app.ts
import { sum } from './utils';
```

---

### 8. tsconfig.json Configuration

- **Important options**: `target`, `module`, `strict`, `lib`, `jsx`.  
- **strict mode** recommended.  
- **paths** for module aliases.  

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "baseUrl": "./",
    "paths": {
      "@utils/*": ["src/utils/*"],
      "@models/*": ["src/models/*"]
    }
  }
}
```

---

## TypeScript vs JavaScript

### 9. Key Differences

- **Type system**: static vs dynamic.  
- **Compilation**: TS → JS.  
- **OOP**: TS has access modifiers, abstract classes, interfaces.  

---

### 10. TypeScript in Popular Frameworks

- React → prefer **functional components with hooks**.  
- Class components = legacy.  

```tsx
// Functional React component
type Props = { initial: number };
function Counter({ initial }: Props) {
  const [count, setCount] = React.useState(initial);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

---

# ✅ Quick Interview One-Liners

- **Interface vs Type**: Interface = contracts & merging; Type = unions/primitives.  
- **Generics**: Reuse logic with type safety.  
- **`as` vs `is`**: `as` = assertion, `is` = predicate + narrowing.  
- **`void` vs `never`**: void = returns nothing; never = never returns.  
- **Enums**: prefer union literals when possible.  
- **`typeof` / `in` / `instanceof`**: type checks.

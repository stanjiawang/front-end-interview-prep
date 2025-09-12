# 🚀 TypeScript Interview Core Concepts

Comprehensive summary of **TypeScript fundamentals, advanced types, configuration, and comparisons with JavaScript**.

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
  - [8. tsconfigjson Configuration](#8-tsconfigjson-configuration)
- [TypeScript vs JavaScript](#typescript-vs-javascript)
  - [9. Key Differences](#9-key-differences)
  - [10. TypeScript in Popular Frameworks](#10-typescript-in-popular-frameworks-eg-react)

---

## Core Concepts & Fundamentals

### 1. Basic Types & Type Inference

- **Primitive Types**: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
- **Arrays**: `number[]` or `Array<number>`.
- **Tuples**: `[string, number]` → fixed number of elements with known types.
- **Enums**: 
  ```ts
  enum Direction { Up, Down }
````

For simpler cases, union literal types are preferred.

* **Any vs Unknown**:

  * `any` bypasses type checking.
  * `unknown` is safer, requires narrowing.

```ts
// Tuple
let userProfile: [string, number, boolean];
userProfile = ['Alice', 30, true];

// Any vs Unknown
let val: unknown = 'hello';
let str1: string = val as string; // type assertion needed

let val2: any = 'world';
let str3: string = val2; // OK - bypasses checks

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

* **Interfaces** define object shapes, act as contracts, and support declaration merging.
* **Type aliases** are more versatile: unions, intersections, primitives, tuples.
* **Use cases**:

  * Interfaces → public APIs and class contracts.
  * Type aliases → unions, intersections, and non-object types.

```ts
// Interface
interface User {
  name: string;
  age: number;
}

interface User { id: number; } // merges automatically

const john: User = { name: 'John', age: 45, id: 101 };

// Type alias
type ID = string | number;
type Status = 'active' | 'inactive';

// Intersection
type Admin = User & { accessLevel: 'admin' | 'superadmin' };
```

---

### 3. Functions & Generics

* **Function type**: `(name: string) => number`.
* **Overloading**: multiple call signatures.
* **Generics**: reusable components with type safety.

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

const stringList = new GenericList<string>();
stringList.addItem('hello');
```

---

## Advanced Types & Utilities

### 4. Advanced Types

* **Union**: `string | number`
* **Intersection**: `A & B`
* **Literal**: `'success' | 123`

---

### 5. void vs never

* **`void`**: function returns nothing (`undefined`).
* **`never`**: function never returns (error, infinite loop).

```ts
function logMessage(message: string): void {
  console.log(message);
}

function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}
```

---

### 6. Type Guards & Type Checking

* `typeof`: primitive type check.
* `in`: property existence check.
* `instanceof`: class instance check.
* `as`: type assertion.
* `is`: type predicate for custom guards.

```ts
// typeof, in, instanceof
function isVehicle(arg: any) {
  if (typeof arg === 'object' && arg !== null) {
    if ('startEngine' in arg) {
      console.log('Has startEngine method.');
    }
  }
}

class Car { drive() {} }
const myCar = new Car();
if (myCar instanceof Car) { console.log('Instance of Car.'); }

// as vs is
interface Cat { name: string; meow(): void; }
interface Dog { name: string; bark(): void; }

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) animal.meow();
  else animal.bark();
}
```

---

## Modules & Configuration

### 7. Modules

* Use `export` and `import`.
* Module resolution strategies: `node` vs `classic`.

---

### 8. tsconfig.json Configuration

* **Important options**: `target`, `module`, `strict`, `lib`, `jsx`.
* **strict mode**: recommended for better type safety.
* **paths**: configure module aliases.

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
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

* **Type System**: TS is statically typed, JS is dynamic.
* **Compilation**: TS must be compiled into JS.
* **OOP**: TS supports modifiers (`public`, `private`, `protected`).

---

### 10. TypeScript in Popular Frameworks (e.g., React)

* Prefer **functional components** with hooks.
* **Class components** are legacy (more verbose, `this` binding issues).

```tsx
// Functional Component
type Props = { initial: number };
function Counter({ initial }: Props) {
  const [count, setCount] = React.useState(initial);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```


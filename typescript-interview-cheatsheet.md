TypeScript Interview Core Concepts
Overview
This guide is designed to help developers systematically review and prepare for technical interviews on TypeScript. It covers a wide range of topics, from fundamental concepts to advanced types, and from common utilities to practical applications.

Core Concepts & Fundamentals
1. Basic Types & Type Inference
Primitive Types: string, number, boolean, null, undefined, symbol, bigint.

Arrays: Defined using number[] or Array<number>.

Tuples: [string, number], an array with a fixed number of elements whose types are known.

Enums: enum Direction { Up, Down }. For a list of simple named constants, a union of literal types is often preferred as it's more tree-shakable and simpler.

Any vs Unknown: any bypasses type checking, while unknown is a safer alternative that requires type narrowing before use.

Code Example:
// Tuple: fixed-length array with defined types at each position
let userProfile: [string, number, boolean];
userProfile = ['Alice', 30, true]; // OK
// userProfile = [30, 'Alice', true]; // Error - wrong order of types

// Any vs Unknown
let val: unknown = 'hello';
let str1: string = val as string; // Type assertion needed
// let str2: string = val; // Error: 'val' is of type 'unknown'

let val2: any = 'world';
let str3: string = val2; // OK - type checking is bypassed

// Enum vs Union Type
enum Status {
  Loading = 'LOADING',
  Success = 'SUCCESS',
  Failure = 'FAILURE',
}
type StatusUnion = 'LOADING' | 'SUCCESS' | 'FAILURE';

2. Interface vs. Type Alias
Definition & Difference: Interfaces are used to define the shape of objects. They act as "contracts" and are "open to merging," meaning multiple interface declarations with the same name will be automatically merged. Type aliases give a new name to a type and are more versatile for defining complex type combinations like unions or intersections. Type aliases cannot be merged.

Use Cases: Interfaces are typically used for public APIs of objects and classes, while type aliases are great for non-object types or combining types.

Code Example:
// Interface for an object
interface User {
  name: string;
  age: number;
}

// Interfaces can be extended and merged
interface User {
  id: number; // Merges with the above User interface
}

const john: User = {
  name: 'John',
  age: 45,
  id: 101, // Now required due to interface merging
};

// Type alias for a combination of types
type ID = string | number;
type Status = 'active' | 'inactive';

// Type aliases can be used to create complex unions or intersections
type Admin = User & { accessLevel: 'admin' | 'superadmin' };

3. Functions & Generics
Function Type Definition: (name: string) => number.

Function Overloading: Allows a function to have multiple call signatures.

Generics: Why do we need generics? They enable you to create reusable components that can work with a variety of types, ensuring type safety without sacrificing flexibility. The generic type variable, like T, "ties" the input type to the output type.

Code Example:
// Generic function
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr.length > 0 ? arr[0] : undefined;
}

const numbers = [1, 2, 3];
const firstNum = getFirstElement(numbers); // firstNum is inferred as number | undefined

const strings = ['a', 'b', 'c'];
const firstStr = getFirstElement(strings); // firstStr is inferred as string | undefined

// Generic class
class GenericList<T> {
  private items: T[] = [];
  addItem(item: T) {
    this.items.push(item);
  }
  getItems(): T[] {
    return this.items;
  }
}

const stringList = new GenericList<string>();
stringList.addItem('hello');
const stringItems = stringList.getItems(); // stringItems is string[]

Advanced Types & Utilities
4. Advanced Types
Union Types: string | number, a value can be one of several types.

Intersection Types: A & B, combines multiple types into one.

Literal Types: 'success', 123, a more precise type that represents a specific value.

5. void vs. never
void: Represents the absence of a return value. A function declared to return void will return undefined.

never: Represents a function that will never return. This is used for functions that throw an error or have an infinite loop. The function never completes its execution.

Code Example:
// void example
function logMessage(message: string): void {
  console.log(message);
}
// This function returns undefined, but the return type is void.

// never example
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}

6. Type Guards & Type Checking
typeof: Checks the basic JavaScript type (e.g., 'string', 'number', 'object').

instanceof: Checks if an object is an instance of a specific class.

in: Checks if a property exists in an object.

as vs. is: as is a type assertion. It tells the compiler to treat a value as a specific type without any runtime checks. is is a type predicate used in a function to prove the type at runtime, creating a type guard.

Code Example:
// `typeof`, `in`, `instanceof`
function isVehicle(arg: any): void {
  if (typeof arg === 'object' && arg !== null) { // Typeof check
    if ('startEngine' in arg) { // 'in' check
      console.log('This object has a startEngine method.');
    }
  }
}

class Car {
  drive() {}
}
const myCar = new Car();
if (myCar instanceof Car) { // instanceof check
  console.log('myCar is an instance of Car.');
}

// `as` vs. `is`
interface Cat {
  name: string;
  meow(): void;
}

interface Dog {
  name: string;
  bark(): void;
}

// `is` is a type predicate that performs a runtime check.
function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow(); // TypeScript now knows 'animal' is a Cat
  } else {
    animal.bark(); // 'animal' must be a Dog
  }
}

Modules & Configuration
7. Modules
Use of export and import.

Module Resolution Strategies (node vs classic).

8. tsconfig.json Configuration
Important Options: target, module, strict, lib, jsx, etc.

strict Mode: Why is it recommended? It enables a strict set of type-checking options that improve code quality and catch common errors.

paths: How to configure module path aliases.

Code Example (tsconfig.json):
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

TypeScript vs. JavaScript
9. Key Differences
Type System: TypeScript is statically typed, while JavaScript is dynamically typed.

Compilation: TypeScript code must be compiled into JavaScript to run.

Object-Oriented Programming: TypeScript provides a more complete set of OOP features like access modifiers.

10. TypeScript in Popular Frameworks (e.g., React)
Prefer Functional Components: The modern, idiomatic way to write React with TypeScript is to use functional components with hooks.

Class Components: While they still work, class components are considered legacy and less common in new projects due to their verbosity and the complexities of this binding.

Keep this document updated and feel free to add your own notes and examples!

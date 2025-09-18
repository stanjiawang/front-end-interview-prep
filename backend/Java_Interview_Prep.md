
# Java Preparation Guide

This guide covers **fundamental → intermediate → commonly used advanced Java knowledge**, with **examples** and **interview questions** to help you prepare for backend interviews.

---

## 1. Java Basics

### 1.1 Hello World Program
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Zoom!");
    }
}
```

### 1.2 Variables and Data Types
- **Primitive Types**: `int`, `double`, `boolean`, `char`
- **Reference Types**: Objects, Arrays

```java
int age = 30;
double price = 19.99;
boolean isActive = true;
String name = "Jia";
```

### 1.3 Control Flow
```java
for (int i = 0; i < 5; i++) {
    System.out.println("i = " + i);
}

if (age > 18) {
    System.out.println("Adult");
} else {
    System.out.println("Minor");
}
```

---

## 2. Object-Oriented Programming (OOP)

### 2.1 Classes and Objects
```java
class Car {
    String brand;
    int year;

    public void drive() {
        System.out.println("Driving " + brand);
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.brand = "Tesla";
        car.year = 2025;
        car.drive();
    }
}
```

### 2.2 Inheritance
```java
class Animal {
    void sound() {
        System.out.println("Animal makes sound");
    }
}

class Dog extends Animal {
    void sound() {
        System.out.println("Bark");
    }
}
```

### 2.3 Polymorphism
```java
Animal a = new Dog();
a.sound(); // "Bark"
```

### 2.4 Interfaces
```java
interface Vehicle {
    void move();
}

class Bike implements Vehicle {
    public void move() {
        System.out.println("Bike is moving");
    }
}
```

---

## 3. Exceptions

### 3.1 Try-Catch
```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Error: " + e.getMessage());
}
```

### 3.2 Custom Exception
```java
class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}
```

---

## 4. Collections Framework

### 4.1 List
```java
import java.util.*;

List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
System.out.println(names.get(0));
```

### 4.2 Set
```java
Set<Integer> numbers = new HashSet<>();
numbers.add(1);
numbers.add(1); // Duplicates ignored
```

### 4.3 Map
```java
Map<String, Integer> ages = new HashMap<>();
ages.put("Alice", 25);
ages.put("Bob", 30);
System.out.println(ages.get("Alice"));
```

---

## 5. Advanced Java Topics

### 5.1 Generics
```java
class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}

Box<Integer> intBox = new Box<>();
intBox.set(123);
```

### 5.2 Streams API (Java 8+)
```java
import java.util.*;

List<Integer> nums = Arrays.asList(1,2,3,4,5);
nums.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .forEach(System.out::println);
```

### 5.3 Lambda Expressions
```java
List<String> list = Arrays.asList("a", "b", "c");
list.forEach(item -> System.out.println(item));
```

### 5.4 Multithreading
```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}
MyThread t = new MyThread();
t.start();
```

### 5.5 Executors
```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(2);
executor.submit(() -> System.out.println("Task executed"));
executor.shutdown();
```

---

## 6. Popular Interview Questions

### 6.1 Core Java
1. **What is the difference between `==` and `.equals()`?**  
   - `==` checks reference equality, `.equals()` checks object content.

2. **Difference between `final`, `finally`, and `finalize`?**  
   - `final`: constant or prevent inheritance.  
   - `finally`: block after try-catch.  
   - `finalize`: method called by GC before object removal.

3. **Explain checked vs unchecked exceptions.**  
   - Checked: must be handled (`IOException`).  
   - Unchecked: runtime (`NullPointerException`).

4. **Why use multithreading?**  
   - To perform tasks concurrently and improve performance.

### 6.2 Coding Questions

**Q1. Reverse a String**
```java
public static String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}
```

**Q2. FizzBuzz**
```java
for (int i = 1; i <= 15; i++) {
    if (i % 3 == 0 && i % 5 == 0) System.out.println("FizzBuzz");
    else if (i % 3 == 0) System.out.println("Fizz");
    else if (i % 5 == 0) System.out.println("Buzz");
    else System.out.println(i);
}
```

**Q3. Find Max in Array**
```java
int[] nums = {3, 7, 2, 9};
int max = nums[0];
for (int n : nums) {
    if (n > max) max = n;
}
System.out.println(max);
```

**Q4. Two Sum**
```java
import java.util.*;

public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

---

# ✅ End Goal

By mastering these fundamentals → intermediate → advanced topics, and practicing the coding questions, you’ll be ready to confidently handle Java interview rounds at Zoom or any backend-focused company.

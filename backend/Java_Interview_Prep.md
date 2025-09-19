
# Java Ultra‑Detailed Interview Preparation Guide
**Scope:** Fundamentals → Intermediate → Common Advanced Topics, with **clear explanations**, **pitfalls**, and **runnable examples**. Ends with **expanded interview Q&A** and **coding challenges** (solutions included).  
**Target:** Modern Java (8 → 21), backend interviews (e.g., Zoom R17265).

---

## Table of Contents
1. [Platform & Tooling](#1-platform--tooling)
2. [Syntax, Types & Language Basics](#2-syntax-types--language-basics)
3. [OOP: Classes, Objects, Inheritance, Polymorphism](#3-oop-classes-objects-inheritance-polymorphism)
4. [Object Contracts: equals, hashCode, toString, Comparable, Comparator](#4-object-contracts-equals-hashcode-tostring-comparable-comparator)
5. [Exceptions & Error Handling](#5-exceptions--error-handling)
6. [Generics (with Erasure), Bounds & Wildcards (PECS)](#6-generics-with-erasure-bounds--wildcards-pecs)
7. [Collections Framework (Core & Concurrency-aware)](#7-collections-framework-core--concurrency-aware)
8. [Streams & Functional Style](#8-streams--functional-style)
9. [Concurrency & Multithreading](#9-concurrency--multithreading)
10. [Java Memory Model & GC (High-Level)](#10-java-memory-model--gc-high-level)
11. [I/O & NIO.2 (Files, Paths, Channels, WatchService)](#11-io--nio2-files-paths-channels-watchservice)
12. [Date & Time (java.time)](#12-date--time-javatime)
13. [Reflection, Annotations & Dynamic Proxies](#13-reflection-annotations--dynamic-proxies)
14. [Networking & HTTP (Java 11+ HttpClient)](#14-networking--http-java-11-httpclient)
15. [Security Basics (Hashing, Encryption), Robustness](#15-security-basics-hashing-encryption-robustness)
16. [Testing & Logging (JUnit 5, Mockito, SLF4J)](#16-testing--logging-junit-5-mockito-slf4j)
17. [Build Tools & Packaging (Maven/Gradle, JARs, Fat JARs)](#17-build-tools--packaging-mavengradle-jars-fat-jars)
18. [Modern Java Features (9→21): var, records, sealed, pattern matching](#18-modern-java-features-921-var-records-sealed-pattern-matching)
19. [Design Patterns & Idioms (Builder, Strategy, …)](#19-design-patterns--idioms-builder-strategy-)
20. [Bridge to Spring Boot](#20-bridge-to-spring-boot)
21. [Interview Q&A (Expanded)](#21-interview-qa-expanded)
22. [Coding Exercises (with Solutions)](#22-coding-exercises-with-solutions)
23. [Appendix: Cheatsheets](#23-appendix-cheatsheets)

---

## 1) Platform & Tooling
**Goal:** Know what runs your code and how your tooling fits together.

**JVM / JRE / JDK**
- **JVM** executes **bytecode** emitted by `javac`. Portable across OSes.
- **JRE** = JVM + core libraries. **JDK** = JRE + dev tools (`javac`, `jar`, `jlink`, `jpackage`, javadoc).
- **OpenJDK** is the open-source reference; vendors (Temurin, Oracle, Amazon Corretto) provide builds.

**Build tools**
- **Maven** (XML, `pom.xml`) and **Gradle** (Groovy/Kotlin DSL). Know dependency scopes and how to run tests/build JAR.
- Typical layout (Maven): `src/main/java`, `src/test/java`, `src/main/resources`.

**Commands**
```bash
# Compile & run (classic)
javac Hello.java && java Hello

# Run single-file source (Java 11+)
java Hello.java

# Maven quickies
mvn -q -DskipTests clean package
mvn test
```

**Pitfalls & Tips**
- Match your **language level** (e.g., 17 or 21) with IDE & build tool settings.
- Keep JDK paths clean; avoid mixing multiple JDKs without toolchains.

---

## 2) Syntax, Types & Language Basics
**Primitives & Wrappers**
- `byte, short, int, long, float, double, char, boolean`; wrappers `Byte, Integer, …`
- **Autoboxing** can allocate; avoid in hot loops (`int` faster than `Integer`).

**Literals & Underscores**
```java
int million = 1_000_000;
long big = 9_000_000_000L;
double precise = 1.23e-9;
char han = '汉';
```

**Strings & Immutability**
- Strings are immutable; concatenation creates new objects.
- Use `StringBuilder` for repeated concatenation.

```java
String a = "Zoom";
String b = a + " Video";     // new object
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 3; i++) sb.append(i);
```

**Control Flow (if/switch/loops)**  
**Switch expression** (Java 14+):
```java
int code = 2;
String msg = switch (code) {
  case 1 -> "ONE";
  case 2 -> "TWO";
  default -> "OTHER";
};
```

**Local Type Inference (var, Java 10+)**
```java
var list = new java.util.ArrayList<String>(); // type still checked at compile time
```

**Common Pitfalls**
- Integer division vs double (`5/2 == 2`); cast to double if needed.
- Beware `==` for Strings; use `.equals()`.

---

## 3) OOP: Classes, Objects, Inheritance, Polymorphism
**Encapsulation**
- Keep fields `private`; expose via getters; validate in constructors.

```java
class Account {
  private final String id;
  private int balance;
  public Account(String id, int initial) {
    if (initial < 0) throw new IllegalArgumentException("neg");
    this.id = id; this.balance = initial;
  }
  public synchronized void deposit(int amt){ if (amt<0) throw new IllegalArgumentException(); balance += amt; }
  public synchronized int balance(){ return balance; }
}
```

**Inheritance & Override**
```java
class Animal { void sound(){ System.out.println("..."); } }
class Dog extends Animal {
  @Override void sound(){ System.out.println("Bark"); }
}
// Polymorphism
Animal a = new Dog(); a.sound(); // Bark
```

**Abstract Classes vs Interfaces**
- Abstract classes can hold **state** + partial implementation.
- Interfaces define contracts; since Java 8: **default** and **static** methods.

```java
interface Greeter { void hello(); default void bye(){ System.out.println("bye"); } }
abstract class Shape { abstract double area(); }
```

**Nested/Inner/Anonymous vs Lambdas**
```java
Runnable r1 = new Runnable(){ public void run(){ System.out.println("anon"); }};
Runnable r2 = () -> System.out.println("lambda");
```

**Immutability Pattern**
- `private final` fields, no setters, make defensive copies. Aids thread-safety.

---

## 4) Object Contracts: equals, hashCode, toString, Comparable, Comparator
**Why it matters:** Correct behavior in sets/maps and sorted collections.

**Contracts**
- If `a.equals(b)` then `a.hashCode()==b.hashCode()` must hold.
- Use `Objects.equals/Objects.hash`. Prefer **records** (auto-generated).

```java
class Person implements Comparable<Person> {
  private final String name; private final int age;
  Person(String n, int a){ name=n; age=a; }

  @Override public boolean equals(Object o){
    if(this==o) return true;
    if(!(o instanceof Person p)) return false;
    return age==p.age && java.util.Objects.equals(name,p.name);
  }
  @Override public int hashCode(){ return java.util.Objects.hash(name,age); }

  @Override public int compareTo(Person o){ return Integer.compare(age, o.age); }
  @Override public String toString(){ return name+"("+age+")"; }
}
```

**Comparator**
```java
java.util.Comparator<Person> byName = java.util.Comparator.comparing(p -> p.name);
java.util.Comparator<Person> byAgeDesc = java.util.Comparator.comparingInt((Person p) -> p.age).reversed();
```

**Pitfalls**
- Mutating fields used in equality while object is in a Set/Map breaks membership.
- Inconsistent `compareTo` and `equals` causes undefined sorted set behavior.

---

## 5) Exceptions & Error Handling
**Checked vs Unchecked**
- **Checked** (e.g., `IOException`) must be declared or caught. Use for **recoverable** problems.
- **Unchecked** (`NullPointerException`, `IllegalArgumentException`) signal programmer errors.

**Best practices**
- Fail fast; include context in messages; wrap lower-level exceptions to add detail.
- Avoid empty `catch`; log or rethrow.
- Prefer **try-with-resources** for I/O to auto-close.

```java
try (var br = java.nio.file.Files.newBufferedReader(java.nio.file.Path.of("data.txt"))) {
  System.out.println(br.readLine());
} catch (java.io.IOException e) {
  throw new RuntimeException("Failed reading data.txt", e);
}
```

**Custom Exceptions**
```java
class BadRequestException extends Exception {
  public BadRequestException(String msg){ super(msg); }
}
```

**Suppressed Exceptions**
- When multiple resources fail to close, Java adds **suppressed** exceptions retrievable via `Throwable.getSuppressed()`.

---

## 6) Generics (with Erasure), Bounds & Wildcards (PECS)
**Why:** Compile-time type safety without casts; reused algorithms.

**Basics**
```java
class Box<T> {
  private T value;
  public void set(T v){ this.value = v; }
  public T get(){ return value; }
}
Box<Integer> b = new Box<>();
b.set(42);
Integer v = b.get();
```

**Generic Methods & Bounds**
```java
static <T extends Number> double sum(java.util.List<T> xs){
  return xs.stream().mapToDouble(Number::doubleValue).sum();
}
```

**Wildcards & PECS**
- **Producer Extends** (`? extends T`) → read-only from structure.
- **Consumer Super** (`? super T`) → write into structure.

```java
java.util.List<? extends Number> src = java.util.List.of(1, 2.5);
java.util.List<? super Integer> dst = new java.util.ArrayList<Number>();
// dst.add(3); // OK
```

**Type Erasure Limitations**
- No `new T[]`; no primitive type params; cannot use `instanceof` with type parameter.

**Pitfalls & Tips**
- Prefer method type params to wildcards when complex.
- Capture helper methods can simplify wildcard logic.

---

## 7) Collections Framework (Core & Concurrency-aware)
**Families & Complexity (avg):**
- **List:** `ArrayList` (get O(1), add end amortized O(1)), `LinkedList` (insert/remove O(1) with iterator).
- **Set:** `HashSet` (O(1)), `LinkedHashSet` (predictable iteration), `TreeSet` (O(log n), sorted).
- **Map:** `HashMap`, `LinkedHashMap` (LRU with accessOrder), `TreeMap`.
- **Queue/Deque:** `ArrayDeque`, `PriorityQueue` (heap, O(log n)).

**Examples**

*Frequency Map (word count)*
```java
var freq = new java.util.HashMap<String,Integer>();
for (var w : java.util.List.of("a","b","a")) {
  freq.merge(w, 1, Integer::sum);
}
```

*LRU Cache via LinkedHashMap*
```java
class LRU<K,V> extends java.util.LinkedHashMap<K,V> {
  private final int cap;
  LRU(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(java.util.Map.Entry<K,V> e){ return size() > cap; }
}
```

*PriorityQueue (Top-K)*
```java
var pq = new java.util.PriorityQueue<int[]>((a,b)->a[1]-b[1]); // min-heap on freq
```

**Fail-Fast vs Weakly-Consistent**
- Most `java.util` iterators are **fail-fast**.
- `ConcurrentHashMap` iterators are weakly-consistent (no `ConcurrentModificationException`).

**Immutable/Unmodifiable**
- `List.of(...)`, `Map.of(...)` (Java 9+) are immutable; `Collections.unmodifiableList` wraps but underlying can change.

---

## 8) Streams & Functional Style
**Pipeline anatomy:** Source → intermediate ops → terminal op.
- Intermediate ops are lazy (`map`, `filter`, `flatMap`, `distinct`, `sorted`).
- Terminal ops: `collect`, `reduce`, `forEach`, `count`, `anyMatch`…

**Examples**
```java
var res = java.util.stream.Stream.of("zoom","team","chat","zoom")
  .distinct()
  .filter(s -> s.length()>3)
  .map(String::toUpperCase)
  .sorted()
  .toList();
```

**Collecting & Grouping**
```java
var grouped = java.util.stream.Stream.of("a","aa","b","ccc")
  .collect(java.util.stream.Collectors.groupingBy(String::length));
var joined = java.util.stream.Stream.of("A","B").collect(java.util.stream.Collectors.joining(","));
```

**Reduce**
```java
int sum = java.util.stream.IntStream.rangeClosed(1,10).reduce(0, Integer::sum);
```

**Parallel Streams**
- Good for CPU-heavy tasks on large data; avoid side effects and synchronized collectors.

**Pitfalls**
- Don’t mutate shared state; prefer collectors. Be careful with boxed streams performance.

---

## 9) Concurrency & Multithreading
**Core Concepts**
- **Thread**: unit of execution; **Runnable/Callable** tasks.
- **Race condition**: unsynchronized access to shared state.
- **Happens-before**: ordering/visibility guarantees (locks, volatile, thread start/join).

**Threads & Executors**
```java
var pool = java.util.concurrent.Executors.newFixedThreadPool(4);
var fut = pool.submit(() -> 42);
System.out.println(fut.get());
pool.shutdown();
```

**Synchronization & Volatile**
```java
class Counter {
  private int v;
  synchronized void inc(){ v++; }
  synchronized int get(){ return v; }
}
```

**Locks, Latches, Semaphores**
```java
var lock = new java.util.concurrent.locks.ReentrantLock();
lock.lock();
try { /* critical section */ }
finally { lock.unlock(); }
```

```java
var latch = new java.util.concurrent.CountDownLatch(3);
// await in one thread; countDown in workers
```

**Blocking Queues (Producer–Consumer)**
```java
var q = new java.util.concurrent.ArrayBlockingQueue<Integer>(3);
new Thread(() -> { try{ q.put(1); }catch(Exception ignored){} }).start();
```

**Concurrent Collections**
- `ConcurrentHashMap`, `ConcurrentLinkedQueue`, `CopyOnWriteArrayList` (read-heavy).

**Futures & CompletableFuture**
```java
var price = java.util.concurrent.CompletableFuture.supplyAsync(() -> 199.0);
var discounted = price.thenApply(p -> p * 0.9);
System.out.println(discounted.join());
```

**Virtual Threads (Java 21, Loom)**
```java
Runnable task = () -> System.out.println("vt " + Thread.currentThread());
Thread t = Thread.ofVirtual().start(task);
t.join();
```

**Common Bugs & Fixes**
- Deadlock (acquire locks in consistent order).  
- Visibility issues (fix with `volatile` or synchronization).  
- Too many threads (use bounded thread pools).

---

## 10) Java Memory Model & GC (High-Level)
**Heap vs Stack**
- Heap holds objects; stack holds frames & references.  
**JMM Guarantees**
- `synchronized` and `volatile` create **happens-before** edges.  
**GC Algorithms**
- **G1GC** (default newer JDKs), **ZGC**, **Shenandoah** (low latency), **Parallel GC** (throughput).
**Tuning idea**
- Start with defaults; only tune with evidence. Flags like `-Xms`, `-Xmx`, `-XX:MaxGCPauseMillis`.

**References**
- **Soft** (cache), **Weak** (GC when weakly reachable), **Phantom** (cleanup bookkeeping).  
- `finalize()` deprecated; use **Cleaner** or try-with-resources.

---

## 11) I/O & NIO.2 (Files, Paths, Channels, WatchService)
**Files & Paths**
```java
var p = java.nio.file.Path.of("notes.txt");
java.nio.file.Files.writeString(p, "Hello");
String s = java.nio.file.Files.readString(p);
```

**Buffered I/O vs Channels**
- Channels + `ByteBuffer` for high throughput.
```java
try (var fc = java.nio.channels.FileChannel.open(p)) {
  var buf = java.nio.ByteBuffer.allocate(64);
  fc.read(buf);
}
```

**WatchService (directory watching)**
```java
var ws = java.nio.file.FileSystems.getDefault().newWatchService();
var dir = java.nio.file.Path.of(".");
dir.register(ws, java.nio.file.StandardWatchEventKinds.ENTRY_CREATE);
var key = ws.take(); // blocking
for (var ev : key.pollEvents()) System.out.println(ev.kind()+" "+ev.context());
```

**Serialization**
- Prefer JSON (Jackson) or protobuf; avoid default Java serialization for security.

---

## 12) Date & Time (java.time)
```java
var now = java.time.Instant.now();
var la = java.time.ZonedDateTime.now(java.time.ZoneId.of("America/Los_Angeles"));
var fmt = java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
System.out.println(java.time.LocalDateTime.now().format(fmt));
```

**Pitfalls**
- DST jumps; always store UTC (`Instant`) + zone separately for display.

---

## 13) Reflection, Annotations & Dynamic Proxies
**Reflection**
```java
Class<?> c = Class.forName("java.lang.String");
for (var m : c.getDeclaredMethods()) { /* inspect */ }
```

**Custom Annotation**
```java
import java.lang.annotation.*;
@Retention(RetentionPolicy.RUNTIME) @Target(ElementType.METHOD)
@interface Audited {}

class Service {
  @Audited void work(){ System.out.println("working"); }
}

static void auditCalls(Object target) throws Exception {
  for (var m : target.getClass().getDeclaredMethods()) {
    if (m.isAnnotationPresent(Audited.class)) {
      System.out.println("Audited method: " + m.getName());
    }
  }
}
```

**Dynamic Proxy**
```java
import java.lang.reflect.*;
interface Greeter { void hi(); }
class G implements Greeter { public void hi(){ System.out.println("hi"); } }
Greeter proxy = (Greeter) Proxy.newProxyInstance(
  Greeter.class.getClassLoader(),
  new Class[]{Greeter.class},
  (obj, method, args) -> { System.out.println("before"); return method.invoke(new G(), args); }
);
proxy.hi();
```

---

## 14) Networking & HTTP (Java 11+ HttpClient)
```java
var client = java.net.http.HttpClient.newBuilder()
  .connectTimeout(java.time.Duration.ofSeconds(5)).build();

var req = java.net.http.HttpRequest.newBuilder(java.net.URI.create("https://httpbin.org/get"))
  .header("Accept","application/json").GET().build();

var res = client.send(req, java.net.http.HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
System.out.println(res.body());

// async
client.sendAsync(req, java.net.http.HttpResponse.BodyHandlers.ofString())
      .thenApply(java.net.http.HttpResponse::body)
      .thenAccept(System.out::println).join();
```

---

## 15) Security Basics (Hashing, Encryption), Robustness
**Hashing (MessageDigest)**
```java
var md = java.security.MessageDigest.getInstance("SHA-256");
byte[] hash = md.digest("secret".getBytes(java.nio.charset.StandardCharsets.UTF_8));
System.out.println(java.util.HexFormat.of().formatHex(hash));
```

**Password storage**
- Use **bcrypt/argon2** (via libs), not plain hashes.  
**Crypto**
- Prefer vetted libs; understand symmetric (AES) vs asymmetric (RSA).  
**Input validation & safe deserialization**; denylist dangerous gadgets; prefer JSON.

---

## 16) Testing & Logging (JUnit 5, Mockito, SLF4J)
**JUnit 5**
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
class MathTest {
  @Test void adds(){ assertEquals(4, 2+2); }
}
```

**Mockito (service with repository)**
```java
// pseudo-code
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
  @Mock UserRepo repo;
  @InjectMocks UserService svc;

  @Test void findsUser(){
    when(repo.findById(1L)).thenReturn(Optional.of(new User(1L,"A")));
    assertEquals("A", svc.findName(1L));
  }
}
```

**Logging**
- API: **SLF4J**; Impl: **Logback**. Use parameterized logs: `log.info("user {} logged in", id);`

---

## 17) Build Tools & Packaging (Maven/Gradle, JARs, Fat JARs)
**Maven `pom.xml` (snippet)**
```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId><artifactId>demo</artifactId><version>1.0.0</version>
  <properties><maven.compiler.source>17</maven.compiler.source><maven.compiler.target>17</maven.compiler.target></properties>
  <dependencies><!-- add your deps --></dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId><artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
      </plugin>
      <!-- Shade for fat JAR -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId><artifactId>maven-shade-plugin</artifactId>
        <version>3.5.0</version>
        <executions><execution><phase>package</phase><goals><goal>shade</goal></goals></execution></executions>
      </plugin>
    </plugins>
  </build>
</project>
```

**Gradle (Kotlin DSL)**
```kotlin
plugins { java }
java { toolchain { languageVersion.set(JavaLanguageVersion.of(17)) } }
repositories { mavenCentral() }
dependencies { testImplementation("org.junit.jupiter:junit-jupiter:5.11.0") }
tasks.test { useJUnitPlatform() }
```

---

## 18) Modern Java Features (9→21): var, records, sealed, pattern matching
```java
var xs = java.util.List.of(1,2,3); // local inference
record UserDTO(Long id, String name) {}
sealed interface Shape permits Circle, Rect {}
final class Circle implements Shape {}
final class Rect implements Shape {}

Object o = "hi";
if (o instanceof String s) { System.out.println(s.toUpperCase()); } // pattern var

String block = \"\"\"
  line1
  line2
  \"\"\"; // text blocks (15+)
```

**Why it matters**
- Records reduce boilerplate; sealed types model algebraic hierarchies, improving exhaustiveness checks.

---

## 19) Design Patterns & Idioms (Builder, Strategy, …)
**Builder**
```java
class User {
  private final String name; private final int age;
  private User(Builder b){ this.name=b.name; this.age=b.age; }
  static class Builder {
    private String name; private int age;
    Builder name(String n){ this.name=n; return this; }
    Builder age(int a){ this.age=a; return this; }
    User build(){ return new User(this); }
  }
}
```

**Strategy**
```java
interface Sorter { void sort(int[] a); }
class QuickSort implements Sorter { public void sort(int[] a){ /* ... */ } }
class MergeSort implements Sorter { public void sort(int[] a){ /* ... */ } }
class Context { private Sorter s; void set(Sorter s){this.s=s;} void doSort(int[] a){ s.sort(a);} }
```

**Template Method, Factory, Singleton (double-checked with volatile)**
```java
class Singleton {
  private static volatile Singleton inst;
  private Singleton(){}
  static Singleton get(){
    if(inst==null) synchronized(Singleton.class){ if(inst==null) inst=new Singleton(); }
    return inst;
  }
}
```

**Idioms**
- Prefer composition over inheritance; minimize mutability; favor Optional for returns; guard clauses for validation.

---

## 20) Bridge to Spring Boot
**Why this matters for Zoom back-end rounds**
- All Java language fundamentals above map directly to: REST controllers, DTOs, Services, Repositories (JPA), concurrency (async tasks), and reliability (exceptions, validation).  
- After this doc, learn: `@RestController`, `@Service`, constructor injection, `@Entity`, `@Repository`, `@Transactional`, config `application.yml`, and how to test with `@SpringBootTest`.

---

## 21) Interview Q&A (Expanded)

### Core Language (15)
1. **Difference between `==` and `.equals()` for objects?** `==` compares references; `.equals()` compares value/content when overridden.  
2. **Autoboxing pitfalls?** Extra allocations; `Integer` caching range [-128,127]; `==` surprises. Use primitives in hot paths.  
3. **`final` vs `finally` vs `finalize`?** `final` keyword; `finally` block; `finalize` deprecated cleanup (avoid).  
4. **String immutability benefits?** Security, caching, thread-safety; enables interning.  
5. **Why prefer `StringBuilder` in loops?** Fewer allocations than `+`.  
6. **Overloading vs overriding?** Overload = same name, different params; override = subclass changes behavior with same signature.  
7. **Access modifiers order of visibility?** `private` < package < `protected` < `public`.  
8. **`transient` keyword?** Field not serialized by default serialization.  
9. **`static` vs instance members?** `static` belong to class; shared across instances.  
10. **`enum` advantages?** Type-safe constants; can hold fields/methods.  
11. **What is a record?** Immutable data carrier with auto-generated `equals/hashCode/toString`.  
12. **Sealed types use-case?** Restrict inheritance; model closed polymorphic hierarchies.  
13. **Pattern matching for `instanceof`?** Binds a typed variable directly (`if (o instanceof String s) ...`).  
14. **Text blocks?** Multi-line strings with preserved formatting (Java 15+).  
15. **Why avoid inheritance for code reuse?** Tight coupling; prefer composition.

### Exceptions (8)
1. Checked vs unchecked and when to use each.  
2. Why wrap exceptions with context.  
3. try-with-resources and suppressed exceptions.  
4. Custom exception design (semantics, naming).  
5. Avoid catching `Exception` broadly—why.  
6. Error vs Exception—should you catch `Error`? (No)  
7. When to rethrow vs recover.  
8. How to map exceptions to HTTP responses in REST (Spring `@ControllerAdvice`).

### Generics (8)
1. Type erasure implications (no reified generics).  
2. Wildcards: `? extends` vs `? super` (PECS).  
3. Generic methods vs generic classes.  
4. Why `List<Object>` ≠ `List<String>`.  
5. Bounds (`<T extends Number>` vs multiple bounds).  
6. Limitations: arrays of generics; `instanceof` with param types.  
7. Type inference with method calls.  
8. When to prefer explicit type parameters for readability.

### Collections (10)
1. HashMap vs LinkedHashMap vs TreeMap.  
2. Why equals/hashCode correctness matters.  
3. Fail-fast iterators & when they trigger.  
4. CopyOnWriteArrayList tradeoffs.  
5. PriorityQueue ordering guarantees.  
6. How to implement LRU.  
7. Immutable collections: `List.of`.  
8. ConcurrentHashMap vs synchronizedMap.  
9. When to use Deque vs Stack (Stack is legacy).  
10. Iteration order guarantees of common collections.

### Streams (8)
1. Lazy evaluation & pipeline formation.  
2. Stateless vs stateful intermediate ops.  
3. `collect` vs `reduce`.  
4. Custom collectors (when?).  
5. Parallel streams pitfalls.  
6. `map` vs `flatMap`.  
7. `groupingBy` and downstream collectors.  
8. Performance considerations (boxing/unboxing, spliterators).

### Concurrency (15)
1. `volatile` guarantees (visibility, not atomicity).  
2. Intrinsic locks vs `ReentrantLock`.  
3. Thread safety strategies: confinement, immutability, synchronization.  
4. Producer–consumer patterns (BlockingQueue).  
5. `CompletableFuture` chaining and exception handling.  
6. Deadlock causes and prevention.  
7. Liveness: deadlock vs livelock vs starvation.  
8. Thread pool sizing heuristics.  
9. Virtual threads advantages and caveats.  
10. How `ConcurrentHashMap` achieves concurrency.  
11. Compare synchronized collections vs concurrent ones.  
12. Happens-before examples (start/join, lock/unlock).  
13. Atomic classes and CAS.  
14. ReadWriteLock usage.  
15. Choosing between parallel streams and executors.

### I/O, NIO, HTTP, Date/Time (8)
1. When to use NIO Channels vs classic streams.  
2. WatchService use-case.  
3. Java 11 HttpClient sync vs async.  
4. Time zones & DST handling with `java.time`.  
5. Charset pitfalls; always specify UTF‑8.  
6. Resource leaks and try-with-resources.  
7. JSON vs Java serialization (security).  
8. Backpressure and large file processing strategies.

### JVM/GC (8)
1. Major GC types and trade-offs (G1, ZGC, Shenandoah).  
2. Heap vs stack & escape analysis.  
3. What is a safepoint.  
4. Soft/weak/phantom references.  
5. Why `finalize` is deprecated.  
6. Impact of large heaps on pause times.  
7. Interpret `-Xms/-Xmx` and tuning only with metrics.  
8. How allocation rate and object lifetimes affect GC behavior.

---

## 22) Coding Exercises (with Solutions)

### 1) Reverse Linked List (Iterative & Recursive)
```java
class ListNode { int val; ListNode next; ListNode(int v){val=v;} }

class ReverseList {
  ListNode iterative(ListNode head){
    ListNode prev=null, cur=head;
    while(cur!=null){ ListNode nxt=cur.next; cur.next=prev; prev=cur; cur=nxt; }
    return prev;
  }
  ListNode recursive(ListNode head){
    if(head==null || head.next==null) return head;
    ListNode p = recursive(head.next);
    head.next.next = head; head.next = null; return p;
  }
}
```

### 2) Two Sum
```java
int[] twoSum(int[] a, int target){
  var map = new java.util.HashMap<Integer,Integer>();
  for(int i=0;i<a.length;i++){
    int need = target - a[i];
    if(map.containsKey(need)) return new int[]{map.get(need), i};
    map.put(a[i], i);
  }
  return new int[]{-1,-1};
}
```

### 3) Valid Parentheses
```java
boolean isValid(String s){
  var st=new java.util.ArrayDeque<Character>();
  for(char c: s.toCharArray()){
    switch(c){
      case '(': case '[': case '{': st.push(c); break;
      case ')': if(st.isEmpty()||st.pop()!='(') return false; break;
      case ']': if(st.isEmpty()||st.pop()!='[') return false; break;
      case '}': if(st.isEmpty()||st.pop()!='{') return false; break;
    }
  }
  return st.isEmpty();
}
```

### 4) Group Anagrams (Streams)
```java
java.util.List<java.util.List<String>> group(String[] strs){
  return java.util.Arrays.stream(strs)
   .collect(java.util.stream.Collectors.groupingBy(
      s -> s.chars().sorted()
             .collect(StringBuilder::new,(b,c)->b.append((char)c),StringBuilder::append)
             .toString()
   ))
   .values().stream().toList();
}
```

### 5) Merge Intervals
```java
int[][] merge(int[][] in){
  java.util.Arrays.sort(in, java.util.Comparator.comparingInt(a->a[0]));
  java.util.List<int[]> res=new java.util.ArrayList<>();
  int[] cur=in[0];
  for(int i=1;i<in.length;i++){
    if(in[i][0] <= cur[1]) cur[1] = Math.max(cur[1], in[i][1]);
    else { res.add(cur); cur=in[i]; }
  }
  res.add(cur);
  return res.toArray(new int[0][]);
}
```

### 6) Top K Frequent Elements (PQ)
```java
int[] topK(int[] nums, int k){
  var freq=new java.util.HashMap<Integer,Integer>();
  for(int n:nums) freq.merge(n,1,Integer::sum);
  var pq=new java.util.PriorityQueue<int[]>((a,b)->a[1]-b[1]);
  for(var e:freq.entrySet()){
    pq.offer(new int[]{e.getKey(), e.getValue()});
    if(pq.size()>k) pq.poll();
  }
  int[] out=new int[pq.size()]; int i=0;
  while(!pq.isEmpty()) out[i++]=pq.poll()[0];
  return out;
}
```

### 7) LRU Cache (LinkedHashMap)
```java
class LRUCache<K,V> extends java.util.LinkedHashMap<K,V>{
  private final int cap;
  LRUCache(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(java.util.Map.Entry<K,V> e){ return size()>cap; }
}
```

### 8) Producer–Consumer (BlockingQueue)
```java
class PC {
  private final java.util.concurrent.BlockingQueue<Integer> q = new java.util.concurrent.ArrayBlockingQueue<>(3);
  void start(){
    Thread p = new Thread(() -> { for(int i=0;i<5;i++) try{ q.put(i);}catch(Exception ignored){} });
    Thread c = new Thread(() -> { for(int i=0;i<5;i++) try{ System.out.println("c:"+q.take()); }catch(Exception ignored){} });
    p.start(); c.start();
    try{ p.join(); c.join(); }catch(Exception ignored){}
  }
}
```

### 9) Binary Search
```java
int binarySearch(int[] a, int target){
  int l=0,r=a.length-1;
  while(l<=r){
    int m=l+(r-l)/2;
    if(a[m]==target) return m;
    if(a[m]<target) l=m+1; else r=m-1;
  }
  return -1;
}
```

### 10) Thread-safe Singleton (DCL)
```java
class Singleton {
  private static volatile Singleton inst;
  private Singleton(){}
  static Singleton get(){
    if(inst==null){
      synchronized(Singleton.class){
        if(inst==null) inst=new Singleton();
      }
    }
    return inst;
  }
}
```

### 11) Palindrome Check
```java
boolean isPal(String s){
  int i=0,j=s.length()-1;
  while(i<j){
    if(s.charAt(i++) != s.charAt(j--)) return false;
  }
  return true;
}
```

### 12) Rotate Matrix 90°
```java
void rotate(int[][] m){
  int n=m.length;
  for(int i=0;i<n;i++) for(int j=i;j<n;j++){ int t=m[i][j]; m[i][j]=m[j][i]; m[j][i]=t; }
  for(int i=0;i<n;i++) for(int j=0;j<n/2;j++){ int t=m[i][j]; m[i][j]=m[i][n-1-j]; m[i][n-1-j]=t; }
}
```

### 13) BFS Level Order (Binary Tree)
```java
class TreeNode{ int val; TreeNode left,right; TreeNode(int v){val=v;} }
java.util.List<java.util.List<Integer>> levelOrder(TreeNode root){
  var res=new java.util.ArrayList<java.util.List<Integer>>();
  if(root==null) return res;
  var q=new java.util.ArrayDeque<TreeNode>(); q.add(root);
  while(!q.isEmpty()){
    int sz=q.size(); var lvl=new java.util.ArrayList<Integer>();
    for(int i=0;i<sz;i++){ var n=q.poll(); lvl.add(n.val); if(n.left!=null) q.add(n.left); if(n.right!=null) q.add(n.right); }
    res.add(lvl);
  }
  return res;
}
```

### 14) Dijkstra (Adjacency List, min-heap)
```java
int[] dijkstra(int n, java.util.List<int[]>[] g, int src){
  int INF=1_000_000_000; int[] dist=new int[n];
  java.util.Arrays.fill(dist, INF); dist[src]=0;
  var pq=new java.util.PriorityQueue<int[]>((a,b)->a[1]-b[1]);
  pq.offer(new int[]{src,0});
  while(!pq.isEmpty()){
    var cur=pq.poll(); int u=cur[0], d=cur[1];
    if(d!=dist[u]) continue;
    for(var e: g[u]){
      int v=e[0], w=e[1];
      if(dist[v]>d+w){ dist[v]=d+w; pq.offer(new int[]{v,dist[v]}); }
    }
  }
  return dist;
}
```

### 15) Quickselect (Kth Smallest)
```java
int quickselect(int[] a, int k){ // 0-indexed
  int l=0,r=a.length-1;
  while(l<=r){
    int x=a[(l+r)>>>1]; // pivot value
    int i=l, j=r;
    while(i<=j){
      while(a[i]<x) i++;
      while(a[j]>x) j--;
      if(i<=j){ int t=a[i]; a[i]=a[j]; a[j]=t; i++; j--; }
    }
    if(k<=j) r=j; else if(k>=i) l=i; else return a[k];
  }
  return -1;
}
```

---

## 23) Appendix: Cheatsheets

**Collections complexity (avg)**  
- ArrayList: get O(1), add end amortized O(1), insert mid O(n).  
- LinkedList: get O(n), insert with iterator O(1).  
- HashMap/HashSet: get/put O(1), worst O(n) if many collisions.  
- TreeMap/TreeSet: O(log n).  
- PriorityQueue: offer/poll O(log n).

**Thread states**: NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED.

**Common numeric gotchas**: overflow (`int`), floating-point rounding, use `BigDecimal` for money.

---

**You’ve reached the end.** Practice writing small Java snippets for *each* section to build muscle memory.

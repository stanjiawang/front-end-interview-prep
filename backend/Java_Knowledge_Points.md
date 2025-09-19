# Java Knowledge Points — Explanations, Why It Matters, Pitfalls & Examples

> Target: Modern Java 8 → 21. For each knowledge point you’ll find: **What & How**, **Why it matters (real-world + interview)**, **Pitfalls**, and **Examples**.

---

## 0) Platform & Tooling

### What & How
- **JVM / JRE / JDK**: JVM executes bytecode; JRE = JVM + core libs; JDK = JRE + dev tools (javac, jar, jlink, jpackage, javadoc).
- **OpenJDK distributions**: Temurin, Amazon Corretto, Oracle JDK.
- **Build tools**: Maven (`pom.xml`), Gradle (`build.gradle`/`build.gradle.kts`).
- **Project layout** (Maven): `src/main/java`, `src/test/java`, `src/main/resources`.
- **Compile & run**:
  ```bash
  javac Hello.java && java Hello
  # Java 11+ single-file run:
  java Hello.java
  ```

### Why it matters
- Interviewers expect basic literacy in **tooling and lifecycle** (compile → package → run → test).
- You’ll discuss **build vs runtime**, dependency management, and reproducing builds.

### Pitfalls
- Mixing JDK versions across IDE/CI can break builds.
- Shadowed/duplicate dependencies cause **NoSuchMethodError**/**ClassNotFoundException** at runtime.

### Example
```xml
<!-- Maven: Java 17, JUnit 5 -->
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId><artifactId>demo</artifactId><version>1.0.0</version>
  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId><artifactId>junit-jupiter</artifactId><version>5.11.0</version><scope>test</scope>
    </dependency>
  </dependencies>
  <build><plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId><artifactId>maven-surefire-plugin</artifactId><version>3.2.5</version>
    </plugin>
  </plugins></build>
</project>
```

---

## 1) Syntax, Types & Language Basics

### What & How
- **Primitives**: `byte, short, int, long, float, double, char, boolean`; wrappers add nullability and methods.
- **Literals**: numeric underscores, hex/bin, scientific notation.
- **Strings**: immutable; use `StringBuilder` in loops.
- **Control flow**: `if`, loops, enhanced for, **switch expressions (14+)**.
- **Local type inference (10+)**: `var` for locals (still statically typed).

### Why it matters
- Core fluency improves speed/accuracy when whiteboarding; shows comfort with **language idioms**.

### Pitfalls
- Integer division & overflow; `==` vs `.equals()` for objects; string interning misunderstandings.

### Examples
```java
int million = 1_000_000;
String s = "Zoom";
s += " Video"; // new String created

int code = 2;
String msg = switch (code) {
  case 1 -> "ONE";
  case 2 -> "TWO";
  default -> "OTHER";
};

var list = new java.util.ArrayList<String>(); // var still checked at compile time
```

---

## 2) OOP: Classes, Encapsulation, Inheritance, Polymorphism, Interfaces

### What & How
- **Encapsulation**: private fields, public getters, validation in constructors.
- **Inheritance/Polymorphism**: `@Override` behavior; Liskov substitution awareness.
- **Interfaces vs Abstract classes**: interfaces (contracts, default methods); abstract (state + partial impl).
- **Immutability**: `private final` fields, no setters; defensive copies.

### Why it matters
- Maps directly to **API design**, **domain models**, and **testability**. Interviewers gauge **design clarity**.

### Pitfalls
- Fragile base classes; overusing inheritance vs **composition**; exposing internals.

### Examples
```java
class Account {
  private final String id;
  private int balance;
  public Account(String id, int initial) {
    if (initial < 0) throw new IllegalArgumentException("negative");
    this.id = id; this.balance = initial;
  }
  public synchronized void deposit(int amt){ if (amt<0) throw new IllegalArgumentException(); balance += amt; }
  public synchronized int balance(){ return balance; }
}

interface Greeter { void hello(); default void bye(){ System.out.println("bye"); } }

class Animal { void sound(){ System.out.println("..."); } }
class Dog extends Animal { @Override void sound(){ System.out.println("Bark"); } }
```

---

## 3) Object Contracts: equals, hashCode, toString, Comparable, Comparator

### What & How
- Respect **equals/hashCode** contract; use `Objects.equals/hash`.
- `Comparable` for natural order; `Comparator` for flexible orderings.

### Why it matters
- Correct **Set/Map** behavior; predictable sorting; avoids subtle bugs.

### Pitfalls
- Mutating fields involved in equality while in `HashSet/HashMap` breaks membership.
- Inconsistent `compareTo` vs `equals` → undefined behavior in sorted sets.

### Examples
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
}
var byName = java.util.Comparator.comparing((Person p) -> p.name).thenComparingInt(p -> p.age);
```

---

## 4) Exceptions & Error Handling

### What & How
- **Checked** (recoverable) vs **Unchecked** (programming errors).
- **try-with-resources** for auto-close; wrap with context.

### Why it matters
- Robustness; clean failure modes; maps to REST error handling in Spring (`@ControllerAdvice`).

### Pitfalls
- Swallowing exceptions; catching overly broad types; not closing resources.

### Example
```java
try (var br = java.nio.file.Files.newBufferedReader(java.nio.file.Path.of("data.txt"))) {
  System.out.println(br.readLine());
} catch (java.io.IOException e) {
  throw new RuntimeException("Failed reading data.txt", e);
}
```

---

## 5) Generics (Erasure), Bounds & Wildcards (PECS)

### What & How
- Parametric types for compile-time safety.
- **Bounds**: `<T extends Number>`, wildcards `? extends` (producer), `? super` (consumer).

### Why it matters
- Reusable libraries & type safety; interviewers test PECS intuition.

### Pitfalls
- Type erasure limitations: no `new T[]`, no `instanceof T`.

### Examples
```java
class Box<T> { private T v; void set(T v){ this.v=v; } T get(){ return v; } }
static <T extends Number> double sum(java.util.List<T> xs){
  return xs.stream().mapToDouble(Number::doubleValue).sum();
}
java.util.List<? extends Number> src = java.util.List.of(1, 2.5);
java.util.List<? super Integer> dst = new java.util.ArrayList<Number>();
```

---

## 6) Collections Framework

### What & How
- Families: List/Set/Map/Queue; time complexities; iteration order guarantees.
- **LinkedHashMap** for LRU, **PriorityQueue** for heaps; immutables via `List.of` (9+).

### Why it matters
- Essential for **algorithmic problems** and real-world data modeling.

### Pitfalls
- Fail-fast iterators; autoboxing overhead in hot loops; misuse of `Vector/Stack` (legacy).

### Examples
```java
// frequency map
var freq = new java.util.HashMap<String,Integer>();
for (var w : java.util.List.of("a","b","a")) freq.merge(w, 1, Integer::sum);

// LRU via LinkedHashMap
class LRU<K,V> extends java.util.LinkedHashMap<K,V>{
  private final int cap; LRU(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(java.util.Map.Entry<K,V> e){ return size()>cap; }
}
```

---

## 7) Streams & Functional Style

### What & How
- Streams = lazy pipelines: source → intermediate ops (`map`, `filter`, `flatMap`) → terminal (`collect`, `reduce`).
- Collectors: `groupingBy`, `joining`, `toList`.

### Why it matters
- Concise, expressive data processing; many interviews ask `map/filter/reduce` exercises.

### Pitfalls
- Shared mutable state; parallel streams on I/O-bound tasks; boxing costs.

### Examples
```java
var res = java.util.stream.Stream.of("zoom","team","chat","zoom")
  .distinct().filter(s->s.length()>3).map(String::toUpperCase).sorted().toList();

var grouped = java.util.stream.Stream.of("a","aa","b","ccc")
  .collect(java.util.stream.Collectors.groupingBy(String::length));
```

---

## 8) Concurrency & Multithreading

### What & How
- **Thread**, **Runnable/Callable**, **ExecutorService**, **Future**.
- **Synchronization** & memory visibility; **volatile** vs **synchronized**.
- High-level utilities: `BlockingQueue`, `CountDownLatch`, `Semaphore`, `CompletableFuture`.
- **Virtual threads (Java 21)** reduce thread-management overhead.

### Why it matters
- Backend systems need concurrency; interviews probe **race conditions**, **happens-before**, and design.

### Pitfalls
- Deadlocks; too many threads; forgetting to shut down executors; non-atomic updates.

### Examples
```java
// executor + future
var pool = java.util.concurrent.Executors.newFixedThreadPool(4);
var fut = pool.submit(() -> 42);
System.out.println(fut.get()); pool.shutdown();

// producer-consumer
var q = new java.util.concurrent.ArrayBlockingQueue<Integer>(2);
new Thread(() -> { try { q.put(1); q.put(2); } catch (Exception ignored) {} }).start();
new Thread(() -> { try { System.out.println(q.take()); } catch (Exception ignored) {} }).start();

// virtual thread
Thread t = Thread.ofVirtual().start(() -> System.out.println("hello from VT"));
t.join();
```

---

## 9) Java Memory Model & Garbage Collection

### What & How
- **Heap vs Stack**; **happens-before** rules (lock/unlock, volatile, start/join).
- GCs: **G1** (default newer JDKs), **ZGC**, **Shenandoah** (low-latency), **Parallel** (throughput).

### Why it matters
- Explains weird concurrency bugs; enables performance discussions.

### Pitfalls
- Tuning without metrics; large heaps increasing pause times; misuse of `finalize()` (deprecated).

### Example Flags
```
-Xms512m -Xmx512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

---

## 10) I/O & NIO.2

### What & How
- `Files`, `Path` utilities; buffered I/O; channels + `ByteBuffer` for throughput; directory **WatchService**.

### Why it matters
- File/network pipelines; log processors; ETL.

### Pitfalls
- Resource leaks; wrong charsets; blocking I/O on hot paths.

### Examples
```java
var p = java.nio.file.Path.of("notes.txt");
java.nio.file.Files.writeString(p, "Hello");
String s = java.nio.file.Files.readString(p);

var ws = java.nio.file.FileSystems.getDefault().newWatchService();
var dir = java.nio.file.Path.of(".");
dir.register(ws, java.nio.file.StandardWatchEventKinds.ENTRY_CREATE);
```

---

## 11) Date & Time (java.time)

### What & How
- `Instant`, `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Duration`, `Period`, `DateTimeFormatter`.

### Why it matters
- Time zones & DST correctness; audits and expiration logic.

### Pitfalls
- Storing local time; not specifying zone; DST gaps/overlaps.

### Examples
```java
var now = java.time.Instant.now();
var la = java.time.ZonedDateTime.now(java.time.ZoneId.of("America/Los_Angeles"));
var fmt = java.time.format.DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
System.out.println(java.time.LocalDateTime.now().format(fmt));
```

---

## 12) Reflection, Annotations & Proxies

### What & How
- Inspect classes/methods at runtime; define custom annotations; dynamic proxies for cross-cutting concerns.

### Why it matters
- Foundation for frameworks (Spring, JPA, AOP). Interviewers may ask conceptual questions.

### Pitfalls
- Performance & security if overused; break encapsulation.

### Examples
```java
@Retention(RetentionPolicy.RUNTIME) @Target(ElementType.METHOD)
@interface Audited {}

class Svc { @Audited void work(){ System.out.println("work"); } }
```

---

## 13) Networking & HTTP (Java 11+ HttpClient)

### What & How
- Modern HTTP client supports sync/async, timeouts, headers, bodies.

### Why it matters
- Integrations & microservices. Shows comfort with real backend calls.

### Examples
```java
var client = java.net.http.HttpClient.newBuilder()
  .connectTimeout(java.time.Duration.ofSeconds(5)).build();
var req = java.net.http.HttpRequest.newBuilder(java.net.URI.create("https://httpbin.org/get"))
  .header("Accept","application/json").GET().build();
var res = client.send(req, java.net.http.HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
```

---

## 14) Security & Robustness Basics

### What & How
- Hashing (SHA‑256) vs encryption (AES/RSA); password hashing (bcrypt/argon2 via libs).
- Input validation; safe deserialization; principle of least privilege.

### Why it matters
- Production safety; interview scenarios around **secrets**, **caching**, **auth**.

### Pitfalls
- Rolling your own crypto; storing plain passwords; insecure randomness.

### Example
```java
var md = java.security.MessageDigest.getInstance("SHA-256");
byte[] hash = md.digest("secret".getBytes(java.nio.charset.StandardCharsets.UTF_8));
System.out.println(java.util.HexFormat.of().formatHex(hash));
```

---

## 15) Testing & Logging

### What & How
- **JUnit 5** for tests, **Mockito** for mocks, **SLF4J/Logback** for logging. Parameterized logs avoid string concatenation.

### Why it matters
- Demonstrates engineering discipline & maintainability.

### Pitfalls
- Tests coupled to implementation; flaky sleeps in async tests.

### Example (pseudo for test)
```java
// @ExtendWith(MockitoExtension.class)
// @Mock Repo repo; @InjectMocks Service svc;
```

---

## 16) Build & Packaging

### What & How
- Maven/Gradle basics; fat/uber JAR with shade; `jlink/jpackage` for custom runtimes.

### Why it matters
- Deployability; shows end‑to‑end ownership.

### Pitfalls
- Conflicting transitive dependencies; version drift between environments.

---

## 17) Modern Java Features (9→21)

### What & How
- `var` locals; **text blocks (15+)** for multiline strings; **records (16+)**; **sealed classes (17+)**; **pattern matching** (`instanceof`, switch); **virtual threads (21)**.

### Why it matters
- Cleaner code; faster development; interview brownie points for **current Java** fluency.

### Examples
```java
record UserDTO(Long id, String name) {}
sealed interface Shape permits Circle, Rect {}
final class Circle implements Shape {}
final class Rect implements Shape {}

Object o = "hi";
if (o instanceof String s) { System.out.println(s.toUpperCase()); }

String block = """
  line1
  line2
  """; // text block
```

---

## 18) Patterns & Idioms

### What & How
- **Builder**, **Strategy**, **Factory**, **Template Method**, **Singleton (DCL + volatile)**.
- Prefer **composition**; limit mutability; guard clauses; `Optional` for return types (not fields).

### Why it matters
- Communicates design maturity; aligns with real codebases.

### Examples
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

---

## 19) Bridge to Spring Boot (Preview)

### What & How
- How Java concepts map: OOP → Controllers/Services/Repositories; Exceptions → error responses; Concurrency → async tasks; Records → DTOs.

### Why it matters
- Smooth transition to REST APIs, JPA, microservices in interviews and on the job.
```


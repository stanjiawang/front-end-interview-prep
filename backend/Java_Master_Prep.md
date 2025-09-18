
# Java Master Preparation Guide (Fundamentals → Intermediate → Common Advanced)

This guide is designed to take you from **fundamentals** to **intermediate** and the **most commonly used advanced Java** topics for backend interviews (e.g., Zoom’s R17265). Each section includes **what to know**, **why it matters**, and **example code** you can run. The final sections include **interview Q&A** and **coding exercises with solutions**.

> Target versions: Modern Java (8 → 21). If you’re short on time, prioritize: **OOP → Collections → Generics → Exceptions → Concurrency → Streams → I/O/NIO → Date/Time → JVM & GC → Modern Java features (records, sealed, pattern matching).**

---

## 0) Java Platform Primer: JDK vs JRE vs JVM, Bytecode, Tooling
**What to know**
- **JVM** executes **bytecode** (portable intermediate form compiled from `.java` to `.class`).
- **JRE** = JVM + core libraries (runtime); **JDK** = JRE + development tools (compiler `javac`, packaging `jar`, javadoc).
- **Build tools**: **Maven** (`pom.xml`), **Gradle** (`build.gradle`).  
- **Run**: `javac Hello.java` → `java Hello` (or via build tool).
- **Project structure** (Maven style): `src/main/java`, `src/test/java`.

**Example**
```bash
# Compile & run a single file (Java 11+ supports single-file run):
java Hello.java
```

---

## 1) Syntax & Types

### 1.1 Primitive Types & Ranges
- `byte` (8-bit), `short` (16), `int` (32), `long` (64 `L`), `float` (32 `f`), `double` (64), `char` (UTF-16), `boolean`.
- Prefer `int`/`long` for counts/ids; use `BigDecimal` for money.

```java
long big = 9_000_000_000L;
double d = 1.2;
float f = 1.2f;
char c = '中';
boolean ok = true;
```

### 1.2 Reference Types & Strings
- Objects live on the heap; variables hold **references**.
- **String** is **immutable**; use `StringBuilder` for heavy concatenation.

```java
String s = "Zoom";
s += " Rocks"; // creates new String
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 3; i++) sb.append(i);
System.out.println(sb.toString()); // "012"
```

### 1.3 var (Java 10+), Records (Java 16+)
```java
var list = new java.util.ArrayList<String>(); // local type inference
record Point(int x, int y) {}                // immutable data carrier
```

### 1.4 Control Flow, Switch Expressions (Java 14+)
```java
int code = 2;
String msg = switch (code) {
  case 1 -> "One";
  case 2 -> "Two";
  default -> "Other";
};
```

---

## 2) OOP Essentials

### 2.1 Classes, Constructors, `this`, `static`
```java
class User {
    private final String name;
    private static int count = 0;
    public User(String name) { this.name = name; count++; }
    public String getName() { return name; }
    public static int getCount() { return count; }
}
```

### 2.2 Inheritance, `super`, Override vs Overload
```java
class Animal { void sound() { System.out.println("..."); } }
class Dog extends Animal {
    @Override void sound() { System.out.println("Bark"); }
}
```

### 2.3 Encapsulation, Access Modifiers
- `private` (class), default/package, `protected` (pkg+subclass), `public`.

### 2.4 Interfaces, Abstract Classes, Default Methods
```java
interface Greeter { void hello(); default void bye() { System.out.println("bye"); } }
abstract class Shape { abstract double area(); }
```

### 2.5 Nested, Inner, Local, Anonymous Classes; Lambdas
```java
Runnable r = new Runnable(){ public void run(){ System.out.println("anon"); }};
Runnable rl = () -> System.out.println("lambda");
```

### 2.6 Immutability Pattern
- Make fields `private final`, no setters, defensive copies.

---

## 3) `equals`, `hashCode`, `toString`, `Comparable`, `Comparator`

**Contracts**
- If `a.equals(b)` then `a.hashCode()==b.hashCode()`.
- Use IDE or Lombok to generate; or Java 16+ **records** auto-generate.

```java
class Person implements Comparable<Person> {
  private final String name; private final int age;
  public Person(String n, int a){ name=n; age=a; }
  @Override public int compareTo(Person o){ return Integer.compare(age, o.age); }
  @Override public boolean equals(Object o){ /* check class & fields */ return o instanceof Person p && age==p.age && name.equals(p.name); }
  @Override public int hashCode(){ return java.util.Objects.hash(name, age); }
  @Override public String toString(){ return name+"("+age+")"; }
}
```

---

## 4) Exceptions & Error Handling

- **Checked** (`IOException`) must be declared or caught; **unchecked** (`NullPointerException`) extends `RuntimeException`.
- **try-with-resources** for auto-closing.
- Avoid swallowing; wrap with context.

```java
try (java.io.BufferedReader br = java.nio.file.Files.newBufferedReader(java.nio.file.Path.of("file.txt"))) {
    String line = br.readLine();
} catch (java.io.IOException e) {
    throw new RuntimeException("Reading failed", e);
}
```

Custom exception:
```java
class BadRequestException extends Exception {
    public BadRequestException(String msg){ super(msg); }
}
```

---

## 5) Generics (Type Safety) & Type Erasure

- **Generic classes & methods**, bounds, wildcards, **PECS** (Producer Extends, Consumer Super).
- Erasure: generic info removed at runtime → no `new T[]`.

```java
class Box<T> { private T v; void set(T v){ this.v=v; } T get(){ return v; } }
static <T extends Number> double sum(List<T> list){ return list.stream().mapToDouble(Number::doubleValue).sum(); }
List<? extends Number> src = List.of(1,2,3);   // read-only producer
List<? super Integer> dst = new ArrayList<Number>(); // consumer
```

---

## 6) Collections Framework (Core)

### 6.1 List / Set / Map – Implementations & Traits
- `ArrayList` (fast random access), `LinkedList` (fast insert/remove middle),  
- `HashSet`/`HashMap` (O(1) avg), `LinkedHashMap` (insertion order), `TreeMap`/`TreeSet` (sorted, O(log n)).

```java
List<String> a = new ArrayList<>();
Set<Integer> s = new HashSet<>();
Map<String,Integer> m = new HashMap<>();
```

### 6.2 Equality & Hashing — Pitfalls
- Mutating fields used in `equals/hashCode` breaks set/map membership.
- `subList` view pitfalls: structural changes on parent affect child.

### 6.3 Iteration & Fail-Fast vs Fail-Safe
- Most `java.util` iterators are **fail-fast** (throw `ConcurrentModificationException`).  
- `ConcurrentHashMap` iterators are **weakly consistent**.

### 6.4 LRU Cache via LinkedHashMap
```java
class LRU<K,V> extends LinkedHashMap<K,V> {
  private final int cap;
  LRU(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(Map.Entry<K,V> e){ return size()>cap; }
}
```

---

## 7) Streams & Functional Programming (Java 8+)

### 7.1 Stream Pipeline: Source → Intermediate → Terminal
```java
List<Integer> out = List.of(1,2,3,4,5).stream()
  .filter(x -> x%2==0)
  .map(x -> x*x)
  .collect(java.util.stream.Collectors.toList());
```

### 7.2 Common Ops: `map`, `filter`, `flatMap`, `collect`, `groupingBy`
```java
var grouped = java.util.stream.Stream.of("a","aa","b","ccc")
  .collect(java.util.stream.Collectors.groupingBy(String::length));
```

### 7.3 Parallel Streams – Use with Care
- Good for CPU-bound + large datasets; beware contention & side-effects.

### 7.4 Functional Interfaces & Method References
- `Function`, `Supplier`, `Consumer`, `Predicate`.
```java
List<String> xs = List.of("A","B");
xs.forEach(System.out::println); // method ref
```

---

## 8) Concurrency Essentials

### 8.1 Threads, Runnable, Callable/Future
```java
Thread t = new Thread(() -> System.out.println("Hello from thread"));
t.start(); t.join();
```

```java
var pool = java.util.concurrent.Executors.newFixedThreadPool(4);
java.util.concurrent.Future<Integer> f = pool.submit(() -> 42);
System.out.println(f.get());
pool.shutdown();
```

### 8.2 Synchronization, `volatile`, Locks
- `synchronized` provides **mutual exclusion** + **happens-before** on monitor actions.
- `volatile` guarantees **visibility** but not atomicity.
- Use `ReentrantLock`/`ReadWriteLock` for advanced control.

```java
class Counter {
  private int v;
  synchronized void inc(){ v++; }
  synchronized int get(){ return v; }
}
```

### 8.3 Atomic Types, Concurrent Collections, Queues
```java
java.util.concurrent.atomic.AtomicInteger ai = new java.util.concurrent.atomic.AtomicInteger(0);
ai.incrementAndGet();
java.util.concurrent.BlockingQueue<Integer> q = new java.util.concurrent.ArrayBlockingQueue<>(10);
```

### 8.4 Producer–Consumer with `BlockingQueue`
```java
class PC {
  final java.util.concurrent.BlockingQueue<Integer> q = new java.util.concurrent.LinkedBlockingQueue<>();
  void produce(){ for(int i=0;i<5;i++){ try{ q.put(i); }catch(Exception ignored){} } }
  void consume(){ for(int i=0;i<5;i++){ try{ System.out.println(q.take()); }catch(Exception ignored){} } }
}
```

### 8.5 `CompletableFuture` (Async Pipelines)
```java
var price = java.util.concurrent.CompletableFuture.supplyAsync(() -> 199.0);
var result = price.thenApply(p -> p * 0.9).join();
```

### 8.6 Virtual Threads (Project Loom, Java 21)
- `Thread.ofVirtual().start(...)` → lightweight threads, simplify concurrency.

```java
try (var scope = java.lang.Thread.ofVirtual().factory()) { /* marker */ }
```

Minimal usage:
```java
Runnable task = () -> System.out.println("vt " + Thread.currentThread());
Thread t1 = Thread.ofVirtual().start(task);
t1.join();
```

### 8.7 Deadlock, Livelock, Starvation (Know definitions & patterns to avoid)

---

## 9) I/O & NIO.2 (Files, Paths, Channels)

### 9.1 Files & Paths
```java
var p = java.nio.file.Path.of("notes.txt");
java.nio.file.Files.writeString(p, "Hello");
String s = java.nio.file.Files.readString(p);
```

### 9.2 Buffered I/O vs Channels
- For high throughput, use **NIO Channels/ByteBuffer**.

### 9.3 Serialization (Caution)
- Prefer JSON (Jackson) over native serialization; if used, implement `Serializable` and define `serialVersionUID`.

---

## 10) Date & Time (java.time)

- Prefer `Instant`, `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Duration`, `Period`.
```java
var now = java.time.Instant.now();
var pst = java.time.ZonedDateTime.now(java.time.ZoneId.of("America/Los_Angeles"));
```

Formatting:
```java
var fmt = java.time.format.DateTimeFormatter.ISO_LOCAL_DATE_TIME;
System.out.println(java.time.LocalDateTime.now().format(fmt));
```

---

## 11) Reflection, Annotations, Proxies

### 11.1 Reflection Basics
```java
Class<?> clazz = Class.forName("Person");
java.lang.reflect.Method[] ms = clazz.getDeclaredMethods();
```

### 11.2 Custom Annotation
```java
import java.lang.annotation.*;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Audited {}
```

### 11.3 Dynamic Proxy (AOP-style)
```java
import java.lang.reflect.*;
interface Service { void work(); }
class Svc implements Service{ public void work(){ System.out.println("work"); } }
Service proxy = (Service) Proxy.newProxyInstance(
  Service.class.getClassLoader(),
  new Class[]{Service.class},
  (obj, m, args) -> { System.out.println("before"); return m.invoke(new Svc(), args); }
);
proxy.work();
```

---

## 12) Modern Java Features (9 → 21)

- **Modules** (Java 9): strong encapsulation of packages (rare in small apps).  
- **`var`** (10): local inference.  
- **`Optional`** (8+): avoid `null` returns.  
- **Switch expressions** (14+), **Text Blocks** (15+) for multiline strings.  
- **Records** (16+): concise immutable data carriers.  
- **Sealed classes** (17+): controlled inheritance.
- **Pattern matching**: `instanceof` pattern (16+) & switch pattern (previewed, 21 stabilized).

```java
record UserDTO(Long id, String name) {}
sealed interface Shape permits Circle, Rect {}
final class Circle implements Shape {}
final class Rect implements Shape {}
```

---

## 13) Memory Model, GC, Tuning (High-Level)

**Heap vs Stack**, **Escape analysis**, **Safepoints**.  
**GCs**: **G1** (default modern), **Parallel**, **ZGC**, **Shenandoah** (low-latency).

**Flags** (example; don’t memorize exact values, know purpose):
```txt
-Xms1g -Xmx1g -XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

**References**: strong, **soft** (cache), **weak** (maps), **phantom** (cleanup).  
**`finalize()` is deprecated** → use **Cleaner** or try-with-resources.

---

## 14) Networking & HTTP

### 14.1 Java 11+ HttpClient
```java
var client = java.net.http.HttpClient.newHttpClient();
var req = java.net.http.HttpRequest.newBuilder(java.net.URI.create("https://example.com")).GET().build();
var res = client.send(req, java.net.http.HttpResponse.BodyHandlers.ofString());
System.out.println(res.body());
```

### 14.2 Sockets (Basics)
- `ServerSocket`, `Socket` for TCP server/client (know it exists; rarely coded in interviews).

---

## 15) Security & Robustness Basics

- Use **`BigDecimal`** for currency.  
- Avoid SQL injection; parameterize queries (JDBC/ORM).  
- Hashing vs encryption; store passwords with **bcrypt/argon2** (via libraries).  
- Validate inputs; avoid deserialization of untrusted data.

---

## 16) Testing & Logging

- **JUnit 5**: `@Test`, `assertEquals`, lifecycle annotations.  
- **Mockito**: mock collaborators, `when/thenReturn`.  
- Logging: **SLF4J** API + **Logback** impl.

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
class CalcTest {
  @Test void add(){ assertEquals(4, 2+2); }
}
```

---

## 17) Packaging & Distribution

- **JAR** (plain or **fat/uber JAR** with dependencies).  
- **`jlink`/`jpackage`** to create custom runtimes/native installers.

---

## 18) Idioms, Patterns, & Pitfalls

- **Builder pattern** for complex construction.
- **Try-with-resources** everywhere for I/O.  
- **Prefer composition over inheritance**.  
- **Avoid excessive mutability**; prefer **immutable DTOs**.  
- **Autoboxing pitfalls** (`Integer` vs `int`).  
- **Floating-point** precision; prefer `BigDecimal`.  
- **`Optional`** for return types (not fields).

Builder example:
```java
class User2 {
  private final String name; private final int age;
  private User2(Builder b){ this.name=b.name; this.age=b.age; }
  static class Builder { String name; int age; Builder name(String n){this.name=n; return this;} Builder age(int a){this.age=a; return this;} User2 build(){ return new User2(this);} }
}
```

---

# INTERVIEW PREP

## A) Frequently Asked Concept Questions (with crisp answers)

1. **`==` vs `.equals()`**  
   - `==` compares references (for objects), `.equals()` compares value/content (when overridden).  
2. **`final` vs `finally` vs `finalize`**  
   - `final`: constant/immutable or prevents override; `finally`: always runs after try/catch; `finalize`: deprecated cleanup hook (avoid).  
3. **Checked vs unchecked exceptions**  
   - Checked must be declared/caught; unchecked are runtime errors (programming bugs).  
4. **`volatile` vs `synchronized`**  
   - `volatile` → visibility (no atomicity); `synchronized` → mutual exclusion + visibility.  
5. **`HashMap` vs `ConcurrentHashMap`**  
   - `HashMap` is not thread-safe; `ConcurrentHashMap` provides concurrent reads/writes with segmentation.  
6. **When to use `LinkedList`?**  
   - Rarely; only when frequent insert/delete in middle and iteration dominates random access.  
7. **`Comparable` vs `Comparator`**  
   - `Comparable` defines natural ordering; `Comparator` defines external ordering strategies.  
8. **Why immutability?**  
   - Thread-safety, simpler reasoning, safer caching and hashing.  
9. **Parallel streams caveats**  
   - Not for I/O-bound or small collections; beware contention and side effects.  
10. **Java Memory Model (brief)**  
   - Defines visibility/ordering rules; `happens-before` relations guarantee visibility across threads.

---

## B) Coding Exercises (Solutions Included)

### B.1 Reverse a Linked List
```java
class ListNode { int val; ListNode next; ListNode(int v){val=v;} }
class Sol {
  ListNode reverse(ListNode head){
    ListNode prev=null, cur=head;
    while(cur!=null){ ListNode nxt=cur.next; cur.next=prev; prev=cur; cur=nxt; }
    return prev;
  }
}
```

### B.2 Valid Parentheses
```java
boolean isValid(String s){
  java.util.Deque<Character> st = new java.util.ArrayDeque<>();
  for(char c: s.toCharArray()){
    if(c=='('||c=='['||c=='{') st.push(c);
    else{
      if(st.isEmpty()) return false;
      char o = st.pop();
      if((o=='('&&c!=')')||(o=='['&&c!=']')||(o=='{'&&c!='}')) return false;
    }
  }
  return st.isEmpty();
}
```

### B.3 Two Sum (HashMap)
```java
int[] twoSum(int[] nums, int target){
  java.util.Map<Integer,Integer> map=new java.util.HashMap<>();
  for(int i=0;i<nums.length;i++){
    int need=target-nums[i];
    if(map.containsKey(need)) return new int[]{map.get(need),i};
    map.put(nums[i],i);
  }
  return new int[]{-1,-1};
}
```

### B.4 Anagrams Grouping (Streams)
```java
List<List<String>> group(String[] strs){
  return java.util.Arrays.stream(strs)
    .collect(java.util.stream.Collectors.groupingBy(
      s -> s.chars().sorted().collect(StringBuilder::new, (b,c)->b.append((char)c), StringBuilder::append).toString()
    ))
    .values().stream().toList();
}
```

### B.5 Top K Frequent Elements
```java
int[] topK(int[] nums, int k){
  Map<Integer,Integer> freq=new HashMap<>();
  for(int n:nums) freq.merge(n,1,Integer::sum);
  PriorityQueue<int[]> pq=new PriorityQueue<>((a,b)->a[1]-b[1]);
  for(var e:freq.entrySet()){
    pq.offer(new int[]{e.getKey(),e.getValue()});
    if(pq.size()>k) pq.poll();
  }
  int[] res=new int[pq.size()]; int i=0;
  while(!pq.isEmpty()) res[i++]=pq.poll()[0];
  return res;
}
```

### B.6 Producer–Consumer (BlockingQueue)
```java
class PC2 {
  private final java.util.concurrent.BlockingQueue<Integer> q = new java.util.concurrent.ArrayBlockingQueue<>(3);
  void start(){
    Thread p = new Thread(() -> { for(int i=0;i<5;i++) try{ q.put(i);}catch(Exception ignored){} });
    Thread c = new Thread(() -> { for(int i=0;i<5;i++) try{ System.out.println("cons "+q.take()); }catch(Exception ignored){} });
    p.start(); c.start();
    try{ p.join(); c.join(); }catch(Exception ignored){}
  }
}
```

### B.7 LRU Cache (LinkedHashMap)
```java
class LRUCache<K,V> extends LinkedHashMap<K,V>{
  private final int cap;
  LRUCache(int cap){ super(16,0.75f,true); this.cap=cap; }
  @Override protected boolean removeEldestEntry(Map.Entry<K,V> e){ return size()>cap; }
}
```

### B.8 Thread-safe Singleton (Double-checked locking)
```java
class Singleton {
  private static volatile Singleton inst;
  private Singleton(){}
  public static Singleton get(){
    if(inst==null){
      synchronized(Singleton.class){
        if(inst==null) inst=new Singleton();
      }
    }
    return inst;
  }
}
```

### B.9 Binary Search
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

### B.10 Merge Intervals
```java
int[][] merge(int[][] in){
  java.util.Arrays.sort(in, java.util.Comparator.comparingInt(x->x[0]));
  java.util.List<int[]> res=new java.util.ArrayList<>();
  int[] cur=in[0];
  for(int i=1;i<in.length;i++){
    if(in[i][0]<=cur[1]) cur[1]=Math.max(cur[1], in[i][1]);
    else { res.add(cur); cur=in[i]; }
  }
  res.add(cur);
  return res.toArray(new int[0][]);
}
```

---

## C) Behavioral & Design Prompts (1‑liners to rehearse)
- **Scaling a read-heavy service**: caching with `ConcurrentHashMap`/Caffeine + database read replicas; discuss consistency & invalidation.  
- **Threading bug you fixed**: describe visibility/atomicity issue and how `volatile`/locks/immutability solved it.  
- **Design a rate limiter**: token bucket with `ScheduledExecutorService` or `Guava RateLimiter` (if allowed), thread-safe structure.

---

## D) 1‑Hour Daily Drills (Optional)
- **15m** flashcards (concept Q&A above)  
- **20m** code one exercise (B1–B10) in pure Java  
- **15m** read a `java.util.concurrent` or `java.time` class javadoc  
- **10m** summarize what you learned

---

### Final Tips
- Prefer **clear**, **correct**, **idiomatic** Java over clever tricks.
- In concurrency: favor **immutability** and **message passing** or **high-level concurrency utils**.
- In interviews: when asked about trade‑offs, mention **time/space complexity**, **thread‑safety**, and **maintainability**.

Good luck — you’ve got this! 💪

# Java Interview Q&A — Fully Answered

> Each question includes a **complete answer** and, where helpful, **examples** or discussion of trade‑offs.

---

## Core Language

**Q1. `==` vs `.equals()` for objects? When would they differ?**  
**Answer:** `==` compares **references** (same object identity); `.equals()` compares **logical equality** (contents) when a class overrides it (e.g., `String`, `Integer`, most domain classes). Two distinct `String` objects with the same characters return `true` for `.equals()` but `false` for `==`. For **interned** strings (`String.intern()` or compile‑time constants), `==` can be true because references point to the same interned instance. In collections, `.equals()` and `hashCode()` must be consistent to avoid broken behavior.

**Q2. What does `final` mean for variables, methods, and classes?**  
**Answer:** For **variables**, `final` means the reference cannot be reassigned (object may still be mutable). For **methods**, prevents overriding in subclasses. For **classes**, prevents inheritance. `final` supports thread‑safety for immutable objects (all fields `final` set in constructor → safe publication).

**Q3. Explain autoboxing/unboxing and a common pitfall.**  
**Answer:** Autoboxing converts primitives ↔ wrapper types automatically (e.g., `int` ↔ `Integer`). Pitfalls: (1) **Performance** due to allocations; (2) `Integer` caching (‑128..127) can make `==` comparisons appear to “work” in that range but fail outside; (3) `NullPointerException` when unboxing a `null` `Integer`.

**Q4. Overloading vs overriding?**  
**Answer:** **Overloading** = same method name, different parameter list in the same class (compile‑time). **Overriding** = subclass provides a new implementation with the **same signature** (runtime dispatch). Overriding must respect covariance rules and visibility (cannot reduce visibility).

**Q5. Why are Strings immutable?**  
**Answer:** Security (classloader/resource names), safe interning and caching, thread‑safety. Mutation would break many optimizations and invariants. Use `StringBuilder` for heavy concatenation to avoid intermediate garbage.

---

## Exceptions & Error Handling

**Q6. Checked vs unchecked exceptions; when to use each?**  
**Answer:** **Checked** signal **recoverable** conditions (I/O failures) and must be declared or caught; **unchecked** (`RuntimeException`) represent programming errors (null dereference, illegal args). Use checked when callers are expected to handle; unchecked for contract violations. In layered systems, map exceptions to meaningful errors (e.g., HTTP status in a REST API).

**Q7. What is try‑with‑resources and why is it important?**  
**Answer:** It automatically closes resources implementing `AutoCloseable`. Prevents leaks for files/sockets/jdbc. It preserves suppressed exceptions so you don’t lose failure context during close.

---

## Generics

**Q8. Why can’t you do `new T[]` or `if (x instanceof T)`?**  
**Answer:** Due to **type erasure**, generic type parameters are not reified at runtime, so array element type and `instanceof` checks are not available for `T`. Use `List<T>` instead of arrays; pass a `Class<T>` token if you need runtime checks.

**Q9. Explain PECS (`? extends` vs `? super`).**  
**Answer:** **Producer Extends**: `List<? extends Number>` produces Numbers safely (read‑only); **Consumer Super**: `List<? super Integer>` can **consume** Integers (write). This rule ensures type safety in generic APIs.

---

## Collections

**Q10. HashMap vs LinkedHashMap vs TreeMap?**  
**Answer:** `HashMap` — O(1) average ops, no order; `LinkedHashMap` — predictable iteration order (insertion or access order, enables LRU); `TreeMap` — O(log n), sorted by key. Choose based on ordering and performance requirements.

**Q11. Why are good `equals/hashCode` implementations critical for sets/maps?**  
**Answer:** They determine bucket placement and equality. Broken implementations cause duplicate entries, missing lookups, or infinite loops in edge cases. Always base them on immutable fields or avoid mutating those fields while the object is in a hashed collection.

---

## Streams

**Q12. `map` vs `flatMap` with an example.**  
**Answer:** `map` transforms each element 1→1; `flatMap` transforms each element into a stream and **flattens** (1→N).  
Example: Tokenize words into letters: `words.stream().flatMap(w -> w.chars().mapToObj(c -> (char)c))` produces a stream of letters.

**Q13. When would you avoid parallel streams?**  
**Answer:** For I/O‑bound tasks, small datasets, shared mutable state, or when the cost of splitting/merging exceeds benefit. Also when order must be preserved and combining is expensive.

---

## Concurrency

**Q14. What does `volatile` guarantee? How is it different from `synchronized`?**  
**Answer:** `volatile` ensures **visibility** (reads see latest writes) and prevents instruction reordering for that variable, but it does **not** provide atomicity for compound actions (like `count++`). `synchronized` provides **mutual exclusion** and establishes **happens‑before** relationships. Use atomic classes or locks for compound updates.

**Q15. How does `ConcurrentHashMap` achieve thread safety?**  
**Answer:** It uses striped bins and CAS operations (since Java 8, tree bins for high collision), avoiding a single global lock. Reads are usually lock‑free; writes use fine‑grained synchronization/CAS. Iterators are **weakly consistent** (no `ConcurrentModificationException`).

**Q16. What causes deadlocks and how to avoid them?**  
**Answer:** Cyclic lock dependencies (A → B and B → A). Prevent by (1) acquiring locks in a **consistent global order**, (2) using timeouts/tryLock, (3) minimizing lock scope, (4) preferring non‑blocking algorithms when possible.

**Q17. What are virtual threads and when would you use them?**  
**Answer:** Java 21’s lightweight threads multiplexed over carrier threads. They make **concurrency cheap** for I/O‑bound workloads (microservices) where you want thread‑per‑request style without the memory overhead of platform threads.

---

## I/O, HTTP, Date/Time

**Q18. When do you choose NIO Channels over classic streams?**  
**Answer:** For high throughput, zero‑copy (`FileChannel.transferTo`), non‑blocking I/O, and when working with **selectors**. Streams are simpler for small/medium blocking I/O.

**Q19. Explain Java 11 HttpClient sync vs async.**  
**Answer:** Sync: `client.send(req, BodyHandlers.ofString())` blocks; Async: `client.sendAsync(...)` returns `CompletableFuture`, enabling non‑blocking pipelines and composition (`thenApply`, `allOf`).

**Q20. Handling time zones and DST properly?**  
**Answer:** Store instants in UTC (`Instant`), convert to user zones (`ZonedDateTime`) for display. Use `DateTimeFormatter` with explicit zone/locale. Beware DST gaps/overlaps (e.g., `LocalDateTime` may not exist/ambiguous).

---

## JVM & GC

**Q21. G1 vs ZGC vs Shenandoah trade‑offs?**  
**Answer:** **G1** balances throughput and pause times; **ZGC**/**Shenandoah** aim for **very low pauses** on large heaps with concurrent marking/compaction. Choose based on latency goals and JDK availability; always profile before tuning.

**Q22. Why is `finalize()` deprecated? Alternatives?**  
**Answer:** Unreliable, unpredictable, and dangerous (resurrection). Use **try‑with‑resources**, `Cleaner`, or explicit `close()` methods for releasing resources deterministically.

---

## Security, Testing, Build

**Q23. How would you store passwords?**  
**Answer:** Never plain. Use a strong password hashing function **(bcrypt/argon2/scrypt)** with **salt** and appropriate work factor (via vetted libraries). Store only the hash, not the password.

**Q24. Unit testing best practices?**  
**Answer:** Deterministic tests with clear Arrange‑Act‑Assert; isolate unit under test (mock external deps); avoid sleeping for async (use latches/futures); use coverage as a guide, not a target. Log with parameters instead of string concatenation.

**Q25. What’s an uber JAR and when is it useful?**  
**Answer:** A single JAR that includes application **and** dependencies (via Maven Shade/Gradle Shadow). Useful for simple deployments or fat‑jar microservices; be mindful of **relocation** to avoid classpath conflicts.

---

## Modern Java Features

**Q26. Records vs POJOs?**  
**Answer:** Records are concise, immutable data carriers with compiler‑generated `equals/hashCode/toString`. Great for DTOs. They can declare methods/validation but fields are final.

**Q27. Sealed classes use‑case?**  
**Answer:** Restrict subclassing to a known set, enabling **exhaustive handling** (e.g., pattern‑matching switch) and better modeling of algebraic data types.

**Q28. Pattern matching for `instanceof` and switch?**  
**Answer:** `if (obj instanceof String s)` binds `s`. Pattern‑matching switch enables concise, exhaustive handling of types/conditions (stabilized versions vary by JDK; know the idea).

---

(You can request expansions on any section, but this should cover the majority of “most asked” questions with complete answers.)


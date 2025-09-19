# Spring Boot Interview Q&A — Fully Answered

> Each question includes a **complete answer** and, where helpful, **examples** or configuration snippets.

---

## Core & Auto-Configuration

**Q1. What does `@SpringBootApplication` actually enable?**  
**Answer:** It composes **`@Configuration`** (class provides bean definitions), **`@EnableAutoConfiguration`** (registers auto-config classes that create beans based on classpath & properties), and **`@ComponentScan`** (scans the package of the class and subpackages for stereotypes like `@Component`, `@Service`, `@Repository`, `@Controller`). Boot’s auto-config inspects the **classpath** (e.g., presence of `spring-web`, `DataSource`) and **application properties** to create sensible defaults (e.g., embedded Tomcat, Jackson `ObjectMapper`, `DataSource`). You can override by defining your own beans or using properties.

**Q2. How does Spring Boot decide which beans to create?**  
**Answer:** Through **conditional annotations** on auto-config classes: `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc. If conditions match (e.g., `ObjectMapper` not already present), the auto-config registers a bean. User-defined beans **win** when `@ConditionalOnMissingBean` is used.

**Q3. Component scanning rules and common pitfalls?**  
**Answer:** By default, the package of the main application class and subpackages are scanned. If your components are outside this hierarchy, they won’t be discovered unless you set explicit `@ComponentScan(basePackages=...)`. Pitfalls: duplicate component names, ambiguity with multiple candidates (resolve via `@Primary` or `@Qualifier`).

---

## Web & Validation

**Q4. Difference between `@Controller` and `@RestController`?**  
**Answer:** `@RestController` = `@Controller` + `@ResponseBody` on all handlers → return values are serialized (JSON by default). `@Controller` is for MVC views (e.g., Thymeleaf).

**Q5. How do you validate request bodies and return useful error messages?**  
**Answer:** Put validation annotations on DTOs (e.g., `@NotBlank`, `@Positive`), annotate controller params with `@Valid`, and define a `@ControllerAdvice` to transform `MethodArgumentNotValidException` into consistent error JSON including field messages.

**Q6. How to implement global exception handling?**  
**Answer:** Create a `@ControllerAdvice` with `@ExceptionHandler` methods. Return a structured response with a clear error code, message, and details. Map domain exceptions (e.g., NotFound) to HTTP 404, validation to 400, etc.

---

## Data & Transactions

**Q7. Where should `@Transactional` live and why?**  
**Answer:** On **service layer** methods to wrap business operations. Repositories should stay thin. Transactions across multiple repository calls should be in the service. Pitfall: **self-invocation** — calling another method within the same class bypasses proxies, so the inner `@Transactional` won’t apply.

**Q8. Lazy vs Eager loading — how to avoid N+1?**  
**Answer:** Default to **LAZY** for collections; use `@EntityGraph` or JPQL `join fetch` to fetch related data when needed. Alternatively, map to DTOs with projections. Beware serializing entities with lazy associations directly to JSON (use DTOs).

**Q9. Paging and sorting with Spring Data JPA?**  
**Answer:** Methods receive a `Pageable` or `Sort` parameter. Example: `repo.findByNameContainingIgnoreCase(q, PageRequest.of(0, 20, Sort.by("price").descending()))`. `Page<T>` includes total counts and page metadata.

---

## Configuration & Profiles

**Q10. How do profiles work and what’s the precedence of properties?**  
**Answer:** Profiles allow environment-specific beans or properties (`@Profile("dev")`, `application-dev.yml`). Precedence: command-line args > environment variables > profile properties > application.yml. Use `@ConfigurationProperties` for type-safe settings. Secrets should come from env vars or vault, not source control.

**Q11. What’s `@ConfigurationProperties` vs `@Value`?**  
**Answer:** `@Value` injects individual values; `@ConfigurationProperties` binds groups of related properties into a typed bean. Prefer `@ConfigurationProperties` for structured config, validation, and IDE support.

---

## Security

**Q12. How do you secure a stateless REST API?**  
**Answer:** Disable CSRF, use a `SecurityFilterChain` with JWT filter (or Basic for demos), require authentication for protected routes, and apply method-level security where necessary. Ensure CORS configured for browsers. Keep secrets/keys outside the codebase.

**Q13. Roles vs authorities?**  
**Answer:** Roles are a convention that prepend `ROLE_` in authorities. `hasRole('ADMIN')` checks for authority `ROLE_ADMIN`. Use consistent mapping; store authorities in tokens or DB.

---

## Testing

**Q14. When to use `@WebMvcTest`, `@DataJpaTest`, and `@SpringBootTest`?**  
**Answer:** `@WebMvcTest` focuses on MVC layer with mocked service beans (fast); `@DataJpaTest` configures JPA with an embedded DB (fast repository tests); `@SpringBootTest` boots the full context (slow, for full integration). Choose the smallest scope needed for your test goals.

**Q15. How do you test JPA code realistically?**  
**Answer:** Use **Testcontainers** to spin up a real Postgres/MySQL in tests. Configure datasource to the container, run migrations, and assert repository behavior with actual SQL features. This avoids discrepancies with H2/in-memory DBs.

---

## Observability

**Q16. What does Actuator provide and how do you secure it?**  
**Answer:** Health checks, metrics, env info, and more. Expose only necessary endpoints (e.g., `/actuator/health`, `/actuator/prometheus`). Secure others behind authentication or network rules. Use Micrometer to export metrics to Prometheus/Grafana.

---

## Async & Scheduling

**Q17. How does `@Async` work under the hood?**  
**Answer:** It uses a Spring proxy wrapping the bean; async methods are executed on a `TaskExecutor`. The method must be `public` and invoked via the proxy (self-invocation won’t be proxied). Return types can be `void`, `CompletableFuture<T>`, or `ListenableFuture<T>`.

**Q18. Scheduling with `@Scheduled` gotchas?**  
**Answer:** Use `fixedRate`, `fixedDelay`, or cron. Handle exceptions inside the job (won’t propagate to callers). For concurrency control, use `@SchedulerLock` (ShedLock) when multiple instances run the same job.

---

## HTTP Clients

**Q19. RestTemplate vs WebClient vs OpenFeign?**  
**Answer:** RestTemplate (blocking, simple), WebClient (non-blocking, reactive, good for high concurrency/latency), Feign (declarative clients auto-wired via interfaces). Choose based on performance/concurrency and your stack (WebFlux vs MVC).

**Q20. How to handle retries/timeouts?**  
**Answer:** Configure client timeouts; add **resilience** with Resilience4j (retry, circuit breaker, bulkhead). Avoid unbounded retries; include jitter and backoff.

---

## Deployment

**Q21. How do you package & run a Boot app in Docker?**  
**Answer:** Build fat jar with `spring-boot-maven-plugin` (`mvn package`), then Dockerfile copies the jar and defines entrypoint. Externalize config via env vars and mount secrets. Define liveness/readiness probes in K8s or healthcheck in Docker.

**Q22. How do you tune a Boot service for performance?**  
**Answer:** Measure first. Configure connection pools (HikariCP), enable HTTP compression if helpful, use caching for hot reads, batch writes, lazy logging, and consider GC flags depending on workload. Profile endpoints and DB queries (SQL plan, indexes).

---

## Advanced

**Q23. How do you override or disable auto‑configuration?**  
**Answer:** Use `spring.autoconfigure.exclude=*FullyQualifiedAutoConfigClass*` or `@EnableAutoConfiguration(exclude = ...)`. For customizing, define your own bean; because of `@ConditionalOnMissingBean`, your bean will take precedence.

**Q24. Explain how `@ConfigurationProperties` validation works.**  
**Answer:** Add `@Validated` on the properties class and use Jakarta Validation annotations (`@NotBlank`, `@Min`) on fields. Boot will fail startup if binding violates constraints, providing early feedback.

**Q25. How to handle database migrations?**  
**Answer:** Use **Flyway** or **Liquibase**. Versioned scripts run on startup to move schema forward automatically. Essential for CI/CD and team collaboration.

**Q26. What are `@EntityGraph` and projections?**  
**Answer:** `@EntityGraph` specifies which relationships to fetch eagerly in a particular query, avoiding N+1 while keeping default LAZY. Projections (interface or class based) fetch only the fields you need directly from JPA, reducing serialization costs.

**Q27. How would you implement file upload/download securely?**  
**Answer:** Validate content type/size; store outside web root or in object storage (e.g., S3); scan for malware if necessary; generate signed URLs; set correct `Content-Disposition` and cache headers. Limit rate and authenticate access.

---

(Ask for more Q&A if you need deeper dives into Security, WebFlux, or Cloud patterns.)


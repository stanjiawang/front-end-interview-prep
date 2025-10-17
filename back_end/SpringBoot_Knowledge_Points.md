# Spring Boot Knowledge Points — Explanations, Why It Matters, Pitfalls & Examples

> Target: Spring Boot 3.x (Spring Framework 6), Java 17+. For each topic you’ll find: **What & How**, **Why it matters**, **Pitfalls**, and **Examples**.

---

## 0) Spring, Boot & Auto-Configuration Primer

### What & How
- **Spring** = dependency injection (IoC container), AOP, MVC/WebFlux, Data, Security, etc.
- **Spring Boot** = conventions + **auto‑configuration** + **starters** to reduce boilerplate.
- **Starters**: curated dependencies (`spring-boot-starter-web`, `spring-boot-starter-data-jpa`, …).
- **Auto‑configuration**: `@SpringBootApplication` (meta of `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`) activates conditional beans based on classpath & properties.

### Why it matters
- Interviewers expect you to know **how Boot wires things**, where config comes from, and how to override defaults.

### Pitfalls
- Conflicting starters or classpath versions cause **auto‑config clashes**.
- Component scanning might miss beans if packages aren’t under the main class’s package.

### Example
```java
@SpringBootApplication
public class App {
  public static void main(String[] args) {
    SpringApplication.run(App.class, args);
  }
}
```

**Maven (snippet)**
```xml
<properties>
  <java.version>17</java.version>
  <spring-boot.version>3.3.2</spring-boot.version>
</properties>
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>${spring-boot.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

---

## 1) Beans, DI & Config

### What & How
- **Stereotypes**: `@Component`, `@Service`, `@Repository`, `@Controller/@RestController`.
- **DI**: Prefer **constructor injection** (testable, immutable).  
- **Configuration**: `@Configuration` + `@Bean` methods define beans programmatically.
- **Property binding**: `@ConfigurationProperties(prefix = "app")` binds YAML/properties → POJOs.

### Why it matters
- Everything in Spring is a bean; clean DI leads to testable apps and clear boundaries.

### Pitfalls
- Field injection hinders testing; circular dependencies; forgetting `@EnableConfigurationProperties` (Boot auto‑detects in many cases).

### Examples
```java
@Service
public class PriceService {
  private final TaxClient taxClient;
  public PriceService(TaxClient taxClient){ this.taxClient = taxClient; }
  public BigDecimal total(BigDecimal base){ return base.add(taxClient.tax(base)); }
}

@Configuration
@EnableConfigurationProperties(AppProps.class)
public class AppConfig {
  @Bean Clock clock(){ return Clock.systemUTC(); }
}

@ConfigurationProperties(prefix = "app")
public record AppProps(String region, int cacheTtlSeconds) {}
```

`application.yml`
```yaml
app:
  region: us-west-2
  cache-ttl-seconds: 300
```

---

## 2) Web (Spring MVC): Controllers, Validation, Exception Handling

### What & How
- **REST** controllers: `@RestController`, `@RequestMapping`, `@GetMapping`/`@PostMapping` etc.
- **Validation**: Jakarta Validation (`@NotNull`, `@Email`, `@Size`), enable with `@Validated`.
- **Exception handling**: `@ControllerAdvice` + `@ExceptionHandler` → consistent error JSON.

### Why it matters
- Core of service development and the **most common interview coding exercise**.

### Pitfalls
- Missing `@RequestBody`; validation not triggered without `@Valid`; ambiguous handler mappings.

### Examples
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
  private final ProductService service;
  public ProductController(ProductService service){ this.service = service; }

  @GetMapping("/{id}")
  public ProductDto byId(@PathVariable Long id){ return service.findById(id); }

  @PostMapping
  @ResponseStatus(HttpStatus.CREATED)
  public ProductDto create(@Valid @RequestBody CreateProduct req){ return service.create(req); }
}

record CreateProduct(
  @NotBlank String name,
  @Positive BigDecimal price
){}
```

**Exception handler**
```java
@ControllerAdvice
public class GlobalErrors {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  Map<String, Object> onValidation(MethodArgumentNotValidException ex){
    var errors = ex.getBindingResult().getFieldErrors().stream()
      .collect(java.util.stream.Collectors.toMap(
        fe -> fe.getField(), fe -> fe.getDefaultMessage(), (a,b)->a));
    return Map.of("error","validation_failed","fields",errors);
  }
}
```

---

## 3) Data Access: Spring Data JPA

### What & How
- **Entity** mapping: `@Entity`, `@Id`, `@GeneratedValue`, relationships (`@OneToMany`, `@ManyToOne`).
- **Repository**: `JpaRepository<T,ID>` generates CRUD; derived query methods; `@Query` for JPQL/SQL.
- **Transactions**: `@Transactional` on service layer; readOnly optimizations; propagation.

### Why it matters
- Persistent models are foundational; interviewers test mapping, transactions, and query reasoning.

### Pitfalls
- N+1 queries; lazy vs eager loading; transactional boundaries; DTO vs entity leakage to API.

### Examples
```java
@Entity
@Table(name="products")
public class Product {
  @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
  private Long id;
  private String name;
  private BigDecimal price;
  // getters/setters
}

public interface ProductRepo extends JpaRepository<Product, Long> {
  List<Product> findByNameContainingIgnoreCase(String q, Pageable pageable);
}

@Service
public class ProductService {
  private final ProductRepo repo;
  public ProductService(ProductRepo repo){ this.repo=repo; }

  @Transactional
  public ProductDto create(CreateProduct req){
    Product p = new Product();
    p.setName(req.name()); p.setPrice(req.price());
    return toDto(repo.save(p));
  }

  @Transactional(readOnly=true)
  public ProductDto findById(Long id){
    Product p = repo.findById(id).orElseThrow(() -> new NoSuchElementException("not found"));
    return toDto(p);
  }

  private ProductDto toDto(Product p){ return new ProductDto(p.getId(), p.getName(), p.getPrice()); }
}

public record ProductDto(Long id, String name, BigDecimal price){}
```

**Pagination & Sorting**
```java
Page<Product> page = repo.findAll(PageRequest.of(0, 20, Sort.by("price").descending()));
```

---

## 4) JSON (Jackson), DTO Mapping & Validation

### What & How
- **Jackson** auto-configured; configure with `ObjectMapper` bean or properties.
- DTO mapping: manual mapping, **MapStruct** for compile-time mapping.
- Validation annotations on DTOs; groups for create vs update.

### Why it matters
- Clean boundaries prevent leaking entities and simplify compatibility.

### Pitfalls
- Infinite recursion on bi‑directional relations; fix with DTOs or `@JsonManagedReference/@JsonBackReference` or `@JsonIgnore`.

### Examples
```java
@Configuration
public class JacksonCfg {
  @Bean
  ObjectMapper objectMapper(){
    return JsonMapper.builder()
      .enable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS) // example
      .build();
  }
}
```

---

## 5) Configuration, Profiles & Externalization

### What & How
- `application.yml`, profile-specific `application-dev.yml` etc. Activate via `spring.profiles.active=dev`.
- **Configuration properties** (`@ConfigurationProperties`) for typed settings.
- Secret management via env vars, vaults.

### Why it matters
- Real apps vary per environment; interviews evaluate how you separate **config from code**.

### Pitfalls
- Storing secrets in git; mis-merging multi-document YAML; profile precedence confusion.

### Example `application.yml`
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/app
    username: app
    password: ${DB_PASSWORD:password}
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
server:
  port: 8080
logging:
  level:
    root: INFO
    com.example: DEBUG
```

---

## 6) Testing Spring Boot

### What & How
- **Unit testing** with JUnit 5 + Mockito (mock collaborators).  
- **Slice tests**: `@WebMvcTest` (MVC only), `@DataJpaTest` (JPA only).  
- **Integration tests**: `@SpringBootTest` loads full context; use **Testcontainers** for real DBs.

### Why it matters
- Confident changes; many teams require coverage and CI stability.

### Pitfalls
- Overusing `@SpringBootTest` (slow); not resetting DB between tests; fragile mocks tied to implementation.

### Examples
```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
  @Autowired MockMvc mvc;
  @MockBean ProductService service;

  @Test void getsById() throws Exception {
    when(service.findById(1L)).thenReturn(new ProductDto(1L,"X", new BigDecimal("10.00")));
    mvc.perform(get("/api/products/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("X"));
  }
}
```

```java
@DataJpaTest
class ProductRepoTest {
  @Autowired ProductRepo repo;
  @Test void savesAndFinds(){
    var p = new Product(); p.setName("A"); p.setPrice(new BigDecimal("9.99"));
    var saved = repo.save(p);
    assertThat(repo.findById(saved.getId())).isPresent();
  }
}
```

---

## 7) Actuator, Metrics & Health

### What & How
- **Actuator** exposes health, metrics, info, env endpoints.  
- Integrates with **Micrometer** → Prometheus/Grafana.

### Why it matters
- Operations & SRE compatibility; interviewers expect basic observability.

### Pitfalls
- Exposing sensitive endpoints without security; high‑cardinality metrics.

### Example
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
`application.yml`
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
```

---

## 8) Spring Security (Boot 3)

### What & How
- Stateless JWT or session security using the **SecurityFilterChain** bean (no WebSecurityConfigurerAdapter in Spring 6).
- Method security: `@EnableMethodSecurity`, `@PreAuthorize("hasRole('ADMIN')")`.

### Why it matters
- Most services need authN/authZ; even high-level understanding is prized in interviews.

### Pitfalls
- CSRF confusion (disable for stateless APIs); mixing roles (`ROLE_` prefix) vs authorities.

### Examples
```java
@Configuration
@EnableMethodSecurity
public class SecurityCfg {
  @Bean
  SecurityFilterChain filter(HttpSecurity http) throws Exception {
    http.csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
          .requestMatchers("/actuator/health").permitAll()
          .anyRequest().authenticated())
        .httpBasic(Customizer.withDefaults());
    return http.build();
  }
}
```

---

## 9) Async & Scheduling

### What & How
- `@EnableAsync` + `@Async` returns `CompletableFuture` or void.  
- `@EnableScheduling` + `@Scheduled(cron = "...")` for periodic tasks.

### Why it matters
- Background work (emails, cache warmups), decoupling request/response latency.

### Pitfalls
- Using default executor (unbounded) → define custom `TaskExecutor`; exception handling in async methods.

### Examples
```java
@Configuration
@EnableAsync
@EnableScheduling
public class AsyncSchedCfg {
  @Bean TaskExecutor taskExecutor(){
    var exec = new ThreadPoolTaskExecutor();
    exec.setCorePoolSize(4); exec.setMaxPoolSize(8); exec.initialize();
    return exec;
  }

  @Async public CompletableFuture<String> compute(){ return CompletableFuture.supplyAsync(() -> "ok"); }

  @Scheduled(fixedDelayString = "PT30S")
  public void tick(){ /* do periodic work */ }
}
```

---

## 10) HTTP Clients: RestTemplate, WebClient, OpenFeign

### What & How
- **RestTemplate** (blocking) — stable but in maintenance.  
- **WebClient** (reactive) — non‑blocking, supports backpressure.  
- **OpenFeign** — declarative HTTP client with interface annotations.

### Why it matters
- Service‑to‑service calls are common; knowing trade‑offs shows seniority.

### Pitfalls
- Thread starvation when using blocking clients on limited pools; incorrect timeouts/retries.

### Examples
```java
// WebClient
@Bean WebClient webClient(WebClient.Builder b){ return b.baseUrl("https://api.example.com").build(); }

// Feign (with starter): define interface
@FeignClient(name = "tax", url = "${tax.url}")
interface TaxClient {
  @GetMapping("/tax?amount={amount}")
  BigDecimal tax(@PathVariable BigDecimal amount);
}
```

---

## 11) Docker & Deployment Basics

### What & How
- Package as fat jar (`spring-boot-maven-plugin`), Dockerize, configure health & env vars.

### Why it matters
- End‑to‑end ownership and cloud readiness (ECS/Kubernetes).

### Pitfalls
- Large images, wrong JVM flags, missing health checks.

### Example Dockerfile
```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/app.jar app.jar
ENV JAVA_OPTS="-XX:+UseG1GC -Xms256m -Xmx512m"
EXPOSE 8080
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar app.jar"]
```

---

## 12) Performance Tips

### What & How
- Use pagination; index your DB; avoid N+1; cache hot reads (Caffeine/Redis); connection pools (HikariCP).

### Why it matters
- Interviewers love **practical scaling** strategies and trade‑offs.

### Pitfalls
- Caching without invalidation; overusing parallel streams on I/O.

---

(Ask me to expand any section further with deeper dives or a mini-project.)


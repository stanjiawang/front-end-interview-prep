# Spring Boot Coding Problems — With Explanations, Tests & Pitfalls

> These mini‑exercises mimic common interview tasks. Each includes **approach**, **code**, **complexity**, and **pitfalls**. Use Spring Boot 3.x, Java 17+.

---

## 1) Build a CRUD REST API (Products)

**Approach:** Entity + Repository + Service + Controller. Add validation, error handling, and tests.

**Complexity:** DB ops O(1) average; end‑to‑end complexity dominated by DB.

**Pitfalls:** Returning entities directly (lazy loading issues), missing validation, not handling 404/400.

**Code**

`Product.java`
```java
@Entity
@Table(name="products")
public class Product {
  @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
  private Long id;
  @NotBlank private String name;
  @Positive private BigDecimal price;
  // getters/setters
}
```

`ProductRepo.java`
```java
public interface ProductRepo extends JpaRepository<Product, Long> {
  Page<Product> findByNameContainingIgnoreCase(String q, Pageable pageable);
}
```

`ProductService.java`
```java
@Service
public class ProductService {
  private final ProductRepo repo;
  public ProductService(ProductRepo repo){ this.repo=repo; }

  @Transactional
  public ProductDto create(@Valid CreateProduct req){
    var p=new Product(); p.setName(req.name()); p.setPrice(req.price());
    p=repo.save(p);
    return dto(p);
  }

  @Transactional(readOnly=true)
  public ProductDto findById(Long id){
    var p=repo.findById(id).orElseThrow(() -> new NoSuchElementException("not found"));
    return dto(p);
  }

  @Transactional(readOnly=true)
  public Page<ProductDto> search(String q, Pageable pageable){
    return repo.findByNameContainingIgnoreCase(q==null?"":q, pageable).map(this::dto);
  }

  private ProductDto dto(Product p){ return new ProductDto(p.getId(), p.getName(), p.getPrice()); }
}
```

`ProductController.java`
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
  private final ProductService service;
  public ProductController(ProductService service){ this.service = service; }

  @PostMapping @ResponseStatus(HttpStatus.CREATED)
  public ProductDto create(@Valid @RequestBody CreateProduct req){ return service.create(req); }

  @GetMapping("/{id}")
  public ProductDto byId(@PathVariable Long id){ return service.findById(id); }

  @GetMapping
  public Page<ProductDto> search(@RequestParam(required=false) String q,
                                 @PageableDefault(size=20, sort = "price", direction = Sort.Direction.DESC) Pageable pageable){
    return service.search(q, pageable);
  }
}
```

`Dtos.java`
```java
public record CreateProduct(@NotBlank String name, @Positive BigDecimal price) {}
public record ProductDto(Long id, String name, BigDecimal price) {}
```

`GlobalErrors.java`
```java
@ControllerAdvice
public class GlobalErrors {
  @ExceptionHandler(NoSuchElementException.class)
  @ResponseStatus(HttpStatus.NOT_FOUND)
  Map<String,String> notFound(NoSuchElementException ex){ return Map.of("error","not_found","message", ex.getMessage()); }
}
```

**Tests** (slice MVC)
```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
  @Autowired MockMvc mvc;
  @MockBean ProductService service;

  @Test void createProduct() throws Exception {
    when(service.create(new CreateProduct("A", new BigDecimal("10.00"))))
        .thenReturn(new ProductDto(1L,"A",new BigDecimal("10.00")));
    mvc.perform(post("/api/products")
        .contentType(MediaType.APPLICATION_JSON)
        .content("{\"name\":\"A\",\"price\":10.00}"))
      .andExpect(status().isCreated())
      .andExpect(jsonPath("$.id").value(1));
  }
}
```

---

## 2) Transactions & Concurrency: Safe Balance Transfer

**Approach:** Service method with `@Transactional`; lock rows to prevent lost updates (JPA pessimistic locking) or use optimistic versioning.

**Pitfalls:** Self‑invocation (proxy not applied), missing isolation for read‑modify‑write sequences.

```java
@Entity
class Account {
  @Id @GeneratedValue Long id;
  @Version Long v;
  BigDecimal balance;
}

@Service
public class TransferService {
  private final AccountRepo repo;

  @Transactional
  public void transfer(long from, long to, BigDecimal amt){
    var a = repo.findById(from).orElseThrow();
    var b = repo.findById(to).orElseThrow();
    if (a.getBalance().compareTo(amt) < 0) throw new IllegalStateException("insufficient");
    a.setBalance(a.getBalance().subtract(amt));
    b.setBalance(b.getBalance().add(amt));
  }
}
```

---

## 3) File Upload/Download Endpoint

**Approach:** Use `MultipartFile` for upload, validate content and size; for download, stream with `ResponseEntity` and correct headers.

**Pitfalls:** Storing untrusted files under web root; missing content type; memory spikes if reading whole file.

```java
@PostMapping(value="/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public Map<String,String> upload(@RequestPart("file") MultipartFile file) throws IOException {
  if(file.getSize() > 5*1024*1024) throw new ResponseStatusException(HttpStatus.BAD_REQUEST,"too large");
  Path target = Path.of("uploads", UUID.randomUUID()+ "-" + file.getOriginalFilename());
  Files.createDirectories(target.getParent());
  file.transferTo(target);
  return Map.of("ok","true","path", target.toString());
}

@GetMapping("/download/{name}")
public ResponseEntity<Resource> download(@PathVariable String name) throws IOException {
  Path p = Path.of("uploads", name);
  Resource r = new UrlResource(p.toUri());
  return ResponseEntity.ok()
    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\""+p.getFileName()+"\"")
    .contentType(MediaType.APPLICATION_OCTET_STREAM)
    .body(r);
}
```

---

## 4) Resilient Outbound Call with WebClient + Resilience4j

**Approach:** Configure timeouts, retry with backoff, and a circuit breaker; return fallback on failure.

**Pitfalls:** Unbounded retries; blocking on non‑blocking client; missing timeouts.

```java
@Bean WebClient webClient(WebClient.Builder b){
  return b.baseUrl("https://api.example.com").build();
}

@Service
public class TaxClient {
  private final WebClient wc;
  public TaxClient(WebClient wc){ this.wc=wc; }

  public Mono<BigDecimal> tax(BigDecimal amount){
    return wc.get().uri(uriBuilder -> uriBuilder.path("/tax").queryParam("amount", amount).build())
      .retrieve()
      .bodyToMono(BigDecimal.class)
      .timeout(Duration.ofSeconds(3))
      .retryWhen(Retry.backoff(2, Duration.ofMillis(200)))
      .onErrorReturn(BigDecimal.ZERO);
  }
}
```

---

## 5) Scheduled Job with Idempotency

**Approach:** `@Scheduled` task that processes a batch; use a DB table/lock to ensure only one instance processes the same item (idempotency key).

**Pitfalls:** Duplicate processing on multiple nodes; long‑running jobs overlap.

```java
@Scheduled(cron = "0 */5 * * * *") // every 5 minutes
public void processPending(){
  List<JobItem> items = repo.findPending();
  for (var it : items) {
    if (repo.claim(it.getId())==1) { // update where status=PENDING
      try { doWork(it); repo.complete(it.getId()); }
      catch (Exception e) { repo.fail(it.getId(), e.getMessage()); }
    }
  }
}
```

---

## 6) Configuration Properties with Validation

**Approach:** Strongly typed config with validation; fail fast on startup if invalid.

**Pitfalls:** Missing `@Validated`; wrong prefix; forgetting to register properties class.

```java
@ConfigurationProperties(prefix = "billing")
@Validated
public record BillingProps(@NotBlank String region, @Min(1) int retries) {}
```

`application.yml`
```yaml
billing:
  region: us-west-2
  retries: 3
```

---

## 7) Actuator Custom Health Indicator

**Approach:** Implement `HealthIndicator` to report dependency status.

```java
@Component
public class S3Health implements HealthIndicator {
  @Override public Health health(){
    boolean ok = checkS3(); // ping or list-bucket
    return ok ? Health.up().withDetail("s3","ok").build()
              : Health.down().withDetail("s3","fail").build();
  }
}
```

---

## 8) Method Security

**Approach:** Enable method security and restrict administrative operations by role.

```java
@EnableMethodSecurity
@Service
public class AdminService {
  @PreAuthorize("hasRole('ADMIN')")
  public void deleteAll(){ /* ... */ }
}
```

---

## 9) Caching with Caffeine

**Approach:** Auto-config cache manager and annotate service methods.

**Pitfalls:** Stale data, cache key design, invalidation.

```java
@Configuration
@EnableCaching
public class CacheCfg {
  @Bean CacheManager cacheManager(){
    return new CaffeineCacheManager("prices");
  }
}

@Service
public class PriceService {
  @Cacheable("prices")
  public BigDecimal getPrice(String sku){ /* compute */ return BigDecimal.TEN; }
}
```

---

## 10) Testcontainers for Postgres

**Approach:** Spin up Postgres for integration tests.

```java
@Testcontainers
@SpringBootTest
class RepoIT {
  @Container static PostgreSQLContainer<?> db = new PostgreSQLContainer<>("postgres:16-alpine");
  @DynamicPropertySource
  static void props(DynamicPropertyRegistry r){
    r.add("spring.datasource.url", db::getJdbcUrl);
    r.add("spring.datasource.username", db::getUsername);
    r.add("spring.datasource.password", db::getPassword);
  }
  @Autowired ProductRepo repo;
  @Test void saves(){ /* ... */ }
}
```

---

**Tip:** Convert these into a small portfolio service you can reference in your interview.

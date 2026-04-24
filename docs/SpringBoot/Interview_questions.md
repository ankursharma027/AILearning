# Spring Boot Interview Questions

A collection of common Spring Boot interview questions with simple, easy-to-understand answers.

---

## 1. What is Spring Boot and how is it different from Spring Framework?

**Spring Framework** is a big toolkit for building Java apps. It's powerful but needs a lot of setup — you have to configure XML files, add dependencies manually, and set up servers yourself.

**Spring Boot** makes all of this easier. It:

- **Auto-configures** things for you based on what's on the classpath
- Comes with an **embedded server** (Tomcat) so you don't need to deploy a WAR file
- Has **starter dependencies** (e.g., `spring-boot-starter-web`) that bundle everything you need
- Follows **convention over configuration** — sensible defaults so you write less boilerplate

> Think of Spring as a raw engine, and Spring Boot as a car ready to drive.

---

## 2. What does `@SpringBootApplication` do?

It's a shortcut annotation that combines three annotations into one:

| Annotation | What it does |
|---|---|
| `@Configuration` | Marks the class as a source of bean definitions |
| `@EnableAutoConfiguration` | Tells Spring Boot to auto-configure beans based on classpath |
| `@ComponentScan` | Scans the current package (and sub-packages) for Spring components |

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

## 3. What is the difference between `@Controller` and `@RestController`?

| | `@Controller` | `@RestController` |
|---|---|---|
| Returns | A view name (HTML page) | Data directly (JSON/XML) |
| Used for | MVC web apps | REST APIs |
| Extra annotation needed | `@ResponseBody` on each method | Not needed — it's built in |

```java
// Returns a view template (e.g., Thymeleaf)
@Controller
public class WebController {
    @GetMapping("/home")
    public String home() { return "home"; }
}

// Returns JSON directly
@RestController
public class ApiController {
    @GetMapping("/users")
    public List<User> getUsers() { return userService.findAll(); }
}
```

---

## 4. What is the difference between `@RequestParam`, `@PathVariable`, and `@RequestBody`?

| Annotation | Where it reads from | Example URL |
|---|---|---|
| `@PathVariable` | Part of the URL path | `/users/42` |
| `@RequestParam` | Query string in the URL | `/users?role=admin` |
| `@RequestBody` | The request body (JSON) | POST with JSON payload |

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }

@GetMapping("/users")
public List<User> filter(@RequestParam String role) { ... }

@PostMapping("/users")
public User create(@RequestBody User user) { ... }
```

---

## 5. How does Spring Boot auto-configuration work?

When your app starts, Spring Boot looks at:

1. What's on your **classpath** (which JARs you added)
2. What **beans** you've already defined
3. What **properties** are set

Then it automatically configures things. For example, if `spring-boot-starter-data-jpa` is on the classpath and a datasource URL is in `application.properties`, Spring Boot will auto-configure a `DataSource`, `EntityManagerFactory`, and transaction manager.

All auto-configurations are listed in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.

You can disable one like this:

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
```

---

## 6. What is `application.properties` vs `application.yml`?

Both files configure your Spring Boot app. They do the same thing, just in different formats.

```properties
# application.properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
```

```yaml
# application.yml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
```

`yml` is easier to read for nested configs. `properties` is simpler for flat configs. Pick one — don't use both.

---

## 7. How do you handle exceptions globally in Spring Boot?

Use `@RestControllerAdvice` with `@ExceptionHandler`. This is a single place to handle all exceptions, so you don't repeat error-handling logic in every controller.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGeneral(Exception ex) {
        return ResponseEntity.status(500).body("Something went wrong");
    }
}
```

---

## 8. How do you implement pagination in Spring Boot?

Use Spring Data's `Pageable`. The client sends `page`, `size`, and optionally `sort` as query params.

```java
// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findAll(Pageable pageable);
}

// Controller
@GetMapping("/users")
public Page<User> getUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}
```

Call it like: `GET /users?page=0&size=10&sort=name,asc`

The response includes the data plus metadata (total pages, total elements, current page).

---

## 9. What are Spring Boot Actuator endpoints?

Actuator adds ready-made endpoints for **monitoring and managing** your app in production.

| Endpoint | What it shows |
|---|---|
| `/actuator/health` | Is the app up? DB connected? |
| `/actuator/metrics` | Memory, CPU, request counts |
| `/actuator/env` | All environment properties |
| `/actuator/beans` | All Spring beans in the context |
| `/actuator/info` | Custom app info |

Add it with:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Expose endpoints in `application.properties`:
```properties
management.endpoints.web.exposure.include=health,metrics,info
```

---

## 10. How do you secure a Spring Boot REST API?

The most common approach for REST APIs is **JWT (JSON Web Token)**:

1. User logs in → server returns a JWT token
2. Client sends the token in every request header: `Authorization: Bearer <token>`
3. Server validates the token and allows/denies the request

Other options:

| Method | When to use |
|---|---|
| JWT + Spring Security | Most REST APIs |
| OAuth2 / OpenID Connect | Login with Google, Okta, Keycloak |
| Basic Auth | Simple internal/dev tools only |

Fine-grained control with annotations:
```java
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/users/{id}")
public void deleteUser(@PathVariable Long id) { ... }
```

---

## 11. What is `@Transactional` and when should you use it?

`@Transactional` wraps a method in a database transaction. If anything fails inside, the whole thing rolls back — no partial saves.

```java
@Transactional
public void transferMoney(Long fromId, Long toId, double amount) {
    debit(fromId, amount);   // If this fails...
    credit(toId, amount);    // ...this gets rolled back too
}
```

Use it on **service layer methods**, not controllers or repositories.

---

## 12. What is the difference between `REQUIRED` and `REQUIRES_NEW` transaction propagation?

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | Joins the existing transaction. If none exists, creates one. |
| `REQUIRES_NEW` | Always starts a brand-new transaction. Pauses the outer one. |

Use `REQUIRES_NEW` when an operation (like audit logging) **must commit independently**, even if the outer transaction fails.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAuditLog(String action) {
    // This commits even if the calling method rolls back
}
```

---

## 13. What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?

All four are just `@Component` under the hood — they all register a bean. The difference is **semantic and functional**.

| Annotation | Layer | Extra behavior |
|---|---|---|
| `@Component` | Generic | None |
| `@Service` | Business logic | None (just clarity) |
| `@Repository` | Data access | Translates DB exceptions to Spring exceptions |
| `@Controller` | Web layer | Handles HTTP requests |

---

## 14. What is dependency injection and how does Spring do it?

**Dependency Injection (DI)** means you don't create objects yourself — Spring creates them and hands them to you.

There are three ways to inject:

```java
// 1. Constructor injection (recommended)
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

// 2. Field injection (convenient but harder to test)
@Autowired
private PaymentService paymentService;

// 3. Setter injection
@Autowired
public void setPaymentService(PaymentService ps) {
    this.paymentService = ps;
}
```

**Constructor injection is preferred** because it makes dependencies explicit and easy to unit test.

---

## 15. What is Spring Data JPA and how does it reduce boilerplate?

Spring Data JPA lets you interact with a database without writing SQL for basic operations. You just define an interface, and Spring generates the implementation.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // These are auto-generated — no SQL needed:
    // findAll(), findById(), save(), delete(), count() ...

    // Custom query just by method name:
    List<User> findByLastName(String lastName);
    Optional<User> findByEmail(String email);
}
```

For complex queries, use `@Query`:
```java
@Query("SELECT u FROM User u WHERE u.age > :age")
List<User> findUsersOlderThan(@Param("age") int age);
```

---

## 16. What is the difference between `@Bean` and `@Component`?

| | `@Component` | `@Bean` |
|---|---|---|
| Applied to | Class | Method inside a `@Configuration` class |
| When to use | Your own classes | Third-party classes you can't modify |

```java
// @Component — you own the class
@Component
public class EmailService { ... }

// @Bean — third-party class, you control creation
@Configuration
public class AppConfig {
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper().findAndRegisterModules();
    }
}
```

---

## 17. How do Spring Boot profiles work?

Profiles let you have **different configs for different environments** (dev, test, prod).

```properties
# application-dev.properties
spring.datasource.url=jdbc:h2:mem:testdb

# application-prod.properties
spring.datasource.url=jdbc:mysql://prod-server/mydb
```

Activate a profile:
```properties
# application.properties
spring.profiles.active=dev
```

Or via command line:
```bash
java -jar myapp.jar --spring.profiles.active=prod
```

You can also annotate beans to only load in specific profiles:
```java
@Profile("dev")
@Bean
public DataSource h2DataSource() { ... }
```

---

## 18. What is `@Cacheable` and how do you enable caching in Spring Boot?

`@Cacheable` stores the result of a method so it doesn't run again for the same input.

```java
// Enable caching in your main class or config
@EnableCaching

// Cache the result — method only runs once per unique userId
@Cacheable("users")
public User getUserById(Long userId) {
    return userRepository.findById(userId).orElseThrow();
}

// Clear the cache when user is updated
@CacheEvict(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}
```

By default Spring uses an in-memory cache. For production, swap in Redis:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

---

## 19. What is the difference between `@OneToMany` lazy vs eager loading?

This is about **when** Hibernate fetches related data from the database.

| | Lazy (default for `@OneToMany`) | Eager (default for `@ManyToOne`) |
|---|---|---|
| When data loads | Only when you access it | Immediately with the parent |
| Performance | Better — avoids unnecessary queries | Can be slow if data is large |

```java
@OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
private List<Order> orders;  // Only loaded when you call getOrders()
```

**Common problem:** "LazyInitializationException" — happens when you access lazy data outside a transaction. Fix: use `@Transactional`, fetch joins, or DTOs.

---

## 20. How do you write unit tests in Spring Boot?

Spring Boot provides `@SpringBootTest` for integration tests and `@WebMvcTest` / `@DataJpaTest` for slice tests.

```java
// Test just the web layer (fast — no full context)
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        when(userService.findById(1L)).thenReturn(new User("Alice"));

        mockMvc.perform(get("/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

For pure service tests, use plain JUnit + Mockito without any Spring context — much faster.

---

## 21. What is a Spring Bean and what is the Bean lifecycle?

A **Spring Bean** is simply any object that Spring creates and manages for you. You don't use `new MyClass()` — Spring does it.

**Bean lifecycle (simplified):**

1. Spring reads your config (`@Component`, `@Bean`, XML, etc.)
2. Creates the bean (calls the constructor)
3. Injects dependencies (`@Autowired`)
4. Calls `@PostConstruct` method (if any) — your setup code runs here
5. Bean is **ready to use**
6. When app shuts down, calls `@PreDestroy` method (if any) — cleanup runs here
7. Bean is destroyed

```java
@Component
public class DatabasePool {

    @PostConstruct
    public void init() {
        System.out.println("Connection pool created!");  // runs after bean is ready
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing connections...");  // runs before app shuts down
    }
}
```

---

## 22. What are Bean scopes in Spring Boot?

Bean scope controls **how many instances** Spring creates and **how long they live**.

| Scope | How many instances | When to use |
|---|---|---|
| `singleton` (default) | One shared instance for the whole app | Stateless services, repositories |
| `prototype` | New instance every time it's requested | Stateful objects, non-thread-safe classes |
| `request` | One per HTTP request | Web apps — request-specific data |
| `session` | One per user session | Web apps — user-specific data |

```java
@Component
@Scope("prototype")
public class ReportGenerator {
    // New instance created every time this bean is injected or requested
}
```

> **Default is singleton** — one object shared everywhere. This is why your `@Service` classes must be **stateless** (no instance variables that change per request).

---

## 23. What happens when two beans of the same type exist? How do you fix it?

Spring gets confused if there are two beans of the same type and you try to inject one — it throws `NoUniqueBeanDefinitionException`.

**Fix 1 — Use `@Primary`** to mark one bean as the default choice:
```java
@Bean
@Primary
public PaymentGateway stripeGateway() { return new StripeGateway(); }

@Bean
public PaymentGateway paypalGateway() { return new PaypalGateway(); }
```

**Fix 2 — Use `@Qualifier`** to pick a specific bean by name:
```java
@Autowired
@Qualifier("paypalGateway")
private PaymentGateway paymentGateway;
```

**Fix 3 — Use `@Profile`** so only one bean exists per environment:
```java
@Bean
@Profile("prod")
public PaymentGateway stripeGateway() { ... }

@Bean
@Profile("dev")
public PaymentGateway mockGateway() { ... }
```

---

## 24. What is `@Lazy` and when would you use it?

By default, Spring creates **all singleton beans at startup**. `@Lazy` tells Spring to wait and only create the bean the first time it's actually needed.

```java
@Component
@Lazy
public class HeavyReportService {
    // This bean won't be created until someone first uses it
    public HeavyReportService() {
        System.out.println("HeavyReportService created!");
    }
}
```

**When to use it:**
- The bean is **expensive to create** (e.g., loads a big model, opens connections)
- The bean is **rarely used** (no point paying the startup cost every time)
- To **break circular dependency** issues as a last resort

> Avoid overusing `@Lazy` — it hides startup problems. If a bean fails to create, you won't find out until it's first used (possibly in production).

---

## 25. What is a circular dependency and how do you fix it?

A **circular dependency** happens when Bean A needs Bean B, and Bean B also needs Bean A. Spring can't figure out which one to create first.

```
UserService → OrderService → UserService  ← circular!
```

Spring will throw: `BeanCurrentlyInCreationException`

**Fix 1 — Refactor (best approach):** Extract the shared logic into a third bean that both can depend on.

```
UserService → CommonService ← OrderService  ✓
```

**Fix 2 — Use `@Lazy` on one injection:** Spring creates a proxy first, breaks the cycle.

```java
@Service
public class UserService {

    private final OrderService orderService;

    public UserService(@Lazy OrderService orderService) {
        this.orderService = orderService;
    }
}
```

**Fix 3 — Use setter injection instead of constructor injection:** Spring can create both beans first, then wire them together.

> Circular dependencies are usually a sign the code design needs improvement. Prefer fixing the design over patching with `@Lazy`.

---

## 26. What is a Circuit Breaker and why do you need it?

Imagine your app calls an external service (payment API, shipping service, etc.) and that service goes down. Without a circuit breaker, every request to your app will hang waiting for a timeout — slowing everything down and possibly crashing your whole system.

A **Circuit Breaker** sits in front of that call and monitors failures. When too many failures happen, it **"opens the circuit"** and stops calling the broken service entirely. Instead it immediately returns a fallback response.

**Three states:**

| State | What happens |
|---|---|
| **Closed** (normal) | Calls go through. Failures are counted. |
| **Open** (tripped) | Calls are blocked. Fallback is returned immediately. |
| **Half-Open** (testing) | A few calls are allowed through to check if service recovered. |

```
Closed → (too many failures) → Open → (wait, then test) → Half-Open → (success) → Closed
                                                                       → (still failing) → Open
```

> Think of it like a fuse box — when there's too much load, the fuse blows to protect the rest of the system. Once the problem is fixed, you reset it.

The most popular library for this in Spring Boot is **Resilience4j**.

---

## 27. How do you implement a Circuit Breaker in Spring Boot using Resilience4j?

**Step 1 — Add the dependency:**
```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**Step 2 — Annotate your method:**
```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallbackPayment")
    public String processPayment(String orderId) {
        // Calls external payment API — might fail
        return externalPaymentApi.charge(orderId);
    }

    // Called automatically when circuit is open or call fails
    public String fallbackPayment(String orderId, Exception ex) {
        return "Payment service is down. Please try again later.";
    }
}
```

**Step 3 — Configure it in `application.yml`:**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10          # look at last 10 calls
        failure-rate-threshold: 50       # open circuit if 50% fail
        wait-duration-in-open-state: 10s # wait 10s before trying again
        permitted-calls-in-half-open-state: 3  # test with 3 calls
```

**What this config means in plain English:**
- Look at the last **10 calls**
- If **5 or more** failed (50%), trip the circuit open
- Wait **10 seconds**, then let **3 test calls** through
- If those succeed, close the circuit again

---

## 28. What is a `pom.xml` and what does it do?

`pom.xml` stands for **Project Object Model**. It is the heart of any Maven-based Spring Boot project. It tells Maven:

- What your project is (name, version, group)
- What libraries it needs (dependencies)
- How to build it (plugins, Java version)
- Where to find libraries (repositories)

```xml
<project>
    <groupId>com.example</groupId>       <!-- your company/org name -->
    <artifactId>my-app</artifactId>      <!-- your project name -->
    <version>1.0.0</version>             <!-- your app version -->
    <packaging>jar</packaging>           <!-- output: jar or war -->

    <dependencies>
        <!-- libraries your app needs -->
    </dependencies>
</project>
```

> Think of `pom.xml` like a shopping list — you write what you need, and Maven goes and fetches it for you.

---

## 29. What is `spring-boot-starter-parent` and why do you need it?

`spring-boot-starter-parent` is a special parent POM provided by Spring Boot. When you set it as your parent, you inherit:

- **Pre-set versions** for hundreds of common libraries (no need to specify versions yourself)
- **Sensible build defaults** (Java version, encoding, plugin config)
- **Dependency management** so all libraries are compatible with each other

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<dependencies>
    <!-- No version needed — parent manages it -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

> Without parent, you'd have to manually pick matching versions for every library — and version conflicts are a nightmare to debug.

---

## 30. What are Spring Boot Starters and how do they work?

A **starter** is a pre-packaged bundle of dependencies for a specific feature. Instead of adding 5–10 individual libraries, you add one starter and get everything you need.

| Starter | What it includes |
|---|---|
| `spring-boot-starter-web` | Spring MVC, Tomcat, Jackson (JSON) |
| `spring-boot-starter-data-jpa` | Hibernate, Spring Data, JDBC |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-test` | JUnit, Mockito, AssertJ |
| `spring-boot-starter-actuator` | Health checks, metrics endpoints |

```xml
<!-- This one line gives you a full REST API stack -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> Starters follow the naming pattern: `spring-boot-starter-{feature}`.

---

## 31. What is the difference between `<dependencies>` and `<dependencyManagement>`?

This is a common confusion, especially in multi-module projects.

| | `<dependencies>` | `<dependencyManagement>` |
|---|---|---|
| What it does | Actually adds the library to the project | Just declares the version — doesn't add the library |
| Where used | Any module that needs the library | Parent POM to centrally control versions |
| Does it get downloaded? | Yes | No — only a reference |

```xml
<!-- In parent pom.xml — sets the version centrally -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- In child pom.xml — no version needed, inherits from parent -->
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>
```

---

## 32. What are dependency scopes in Maven and what does each one mean?

Scope controls **when** a dependency is available during the build and runtime.

| Scope | Available at | Included in final JAR? | Common use |
|---|---|---|---|
| `compile` (default) | Everywhere | Yes | Most libraries |
| `runtime` | Runtime only, not compile | Yes | JDBC drivers |
| `test` | Tests only | No | JUnit, Mockito |
| `provided` | Compile time only | No | Servlet API (server provides it) |
| `optional` | Compile time only | No | Flagged as not required by consumers |

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>   <!-- only available in src/test, not in production -->
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>  <!-- not needed to compile, only to run -->
</dependency>
```

---

## Quick Reference Cheat Sheet

| Concept | Key Point |
|---|---|
| `@SpringBootApplication` | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| Auto-configuration | Configured by classpath + properties, can be excluded |
| `@RestController` | `@Controller` + `@ResponseBody` |
| `@Transactional` | Rolls back everything if one step fails |
| `REQUIRED` | Join existing transaction |
| `REQUIRES_NEW` | Start fresh transaction, pause existing |
| `@ControllerAdvice` | Central place for exception handling |
| `Pageable` | Built-in pagination support |
| Profiles | Different configs per environment (dev/prod) |
| `@Cacheable` | Cache method results, avoid repeated DB calls |
| Bean lifecycle | Constructor → `@PostConstruct` → in use → `@PreDestroy` |
| Bean scopes | `singleton` (default, one shared), `prototype` (new each time) |
| `@Primary` | Default bean when multiple of same type exist |
| `@Qualifier` | Pick a specific bean by name when multiple exist |
| `@Lazy` | Delay bean creation until first use |
| Circular dependency | Refactor first; use `@Lazy` or setter injection as fallback |
| Circuit Breaker | Stops calling a failing service; returns fallback instead |
| CB States | Closed (normal) → Open (blocked) → Half-Open (testing) |
| Resilience4j | `@CircuitBreaker(name=..., fallbackMethod=...)` |
| `pom.xml` | Project config — dependencies, build settings, metadata |
| `starter-parent` | Inherits version management + build defaults from Spring Boot |
| Starters | Bundled dependencies for a feature (`spring-boot-starter-web`) |
| `dependencyManagement` | Declares versions centrally, doesn't add the library |
| Dependency scope | `compile` (default), `test` (tests only), `runtime`, `provided` |

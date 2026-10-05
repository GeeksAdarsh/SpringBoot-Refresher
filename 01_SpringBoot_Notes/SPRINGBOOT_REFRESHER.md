# Spring Boot Refresher — Learn It Again Through CodeCampus

You said you learned Spring Boot before but forgot it. This file re-teaches it using **real files from this project** instead of toy examples — so every concept has a concrete "oh, that's why that line exists" moment attached to it.

Read top to bottom once. Afterward, use the **Cheat Sheet** at the very bottom as your quick-reference card.

---

## 0. The One-Sentence Version

Spring Boot's whole philosophy: **you write plain Java classes and put annotations on them; Spring wires everything together for you at startup.**

This is called **Inversion of Control (IoC)** — instead of your code creating its own dependencies (`new AuthService()`), you just *declare* what you need and Spring hands you an already-built instance. Handing you that pre-built instance is called **Dependency Injection (DI)**.

---

## 1. The Entry Point

```java
@SpringBootApplication
@EnableScheduling
@EnableCaching
public class CodeCampusApplication {
    public static void main(String[] args) {
        SpringApplication.run(CodeCampusApplication.class, args);
    }
}
```

`SpringApplication.run(...)` does three things on startup:
- Starts an **embedded Tomcat** web server (no separate server install needed — this is why "Spring Boot" needs nothing but a JAR to run)
- **Scans your whole package** for classes annotated `@Component`, `@Service`, `@Repository`, `@RestController`, `@Configuration` and registers them as **beans** (objects managed by Spring, created once, reused everywhere)
- Reads `application.properties` and injects those values wherever `@Value("${...}")` is used

`@EnableCaching` and `@EnableScheduling` are opt-in switches — Spring Boot doesn't turn on caching or background jobs unless you explicitly ask.

---

## 2. The Layered Architecture (this project's folders ARE the architecture)

```
HTTP Request
     │
     ▼
┌─────────────┐   "What URL was hit? What HTTP method? Deserialize the JSON body."
│  controller/ │   @RestController classes — thin, no business logic, just routes
└──────┬──────┘
       ▼
┌─────────────┐   "Actual business rules: validate, calculate, decide, orchestrate."
│  service/    │   @Service classes — the brains of the app
└──────┬──────┘
       ▼
┌─────────────┐   "Talk to the database." Just interfaces — Spring writes the implementation.
│ repository/  │   extends JpaRepository<Entity, IdType>
└──────┬──────┘
       ▼
┌─────────────┐   "What a table looks like." Plain Java classes mapped to SQL tables.
│  model/      │   @Entity classes (Hibernate/JPA)
└─────────────┘
```

Every feature in this app (auth, tests, questions, proctoring...) follows this exact same 4-layer pattern. Once you understand it for one feature, you understand it for all 15.

**Why the layers are separated:** Controllers shouldn't know SQL, services shouldn't know HTTP status codes, repositories shouldn't know business rules. Each layer has ONE job. This is why `QuestionController` is short but does a lot — it just delegates everything to `QuestionService`.

**The one exception in this repo:** `FeedbackController` talks directly to `FeedbackRepository` with no `FeedbackService` in between. That's fine — the service layer is only needed when there's actual *logic* to put in it. Simple CRUD doesn't always need one.

---

## 3. Annotation Glossary (every annotation used in this repo, explained)

| Annotation | Found in | What it actually does |
|---|---|---|
| `@SpringBootApplication` | `CodeCampusApplication.java` | Marks the entry point; triggers auto-configuration + component scanning |
| `@EnableCaching` | `CodeCampusApplication.java` | Turns on Spring's caching abstraction so `@Cacheable`/`@CacheEvict` work anywhere in the app |
| `@EnableScheduling` | `CodeCampusApplication.java` | Turns on support for `@Scheduled` background jobs (cron-style tasks) |
| `@RestController` | every file in `controller/` | Marks a class whose methods return JSON directly (combines `@Controller` + `@ResponseBody`) |
| `@RequestMapping("/api/tests")` | top of controllers | Sets the base URL path for all methods in that class |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` | controller methods | Maps a specific HTTP verb + sub-path to a Java method |
| `@PathVariable` | e.g. `getById(@PathVariable Long id)` | Extracts a value from the URL path itself, e.g. `/api/tests/{id}` → the `5` in `/api/tests/5` |
| `@RequestBody` | e.g. `create(@RequestBody CodingTestDTO dto)` | Deserializes the incoming JSON body into a Java object automatically (via Jackson) |
| `@RequestParam` | e.g. `list(@RequestParam(defaultValue="30") int limit)` | Reads a query string value, e.g. `?limit=10` |
| `@Valid` | e.g. `register(@Valid @RequestBody RegisterRequest request)` | Runs Bean Validation rules (like `@NotBlank`, `@Email` on the DTO fields) before the method body even runs — invalid requests get auto-rejected with a 400 |
| `@AuthenticationPrincipal` | e.g. `executeCode(..., @AuthenticationPrincipal User user)` | Spring Security automatically injects the **currently logged-in user** (extracted from the JWT) as a method parameter — no manual token parsing needed in controllers |
| `@PreAuthorize("hasRole('ADMIN')")` | class or method level | Blocks the request with a 403 *before* the method runs, unless the authenticated user has that role. This is how RBAC (role-based access control) is enforced |
| `@Service` | every file in `service/` | Marks a class as a Spring-managed bean containing business logic |
| `@Component` | e.g. `JwtAuthenticationFilter`, `DataInitializer` | Generic "let Spring manage this object" annotation, used when `@Service`/`@Repository`/`@Controller` don't quite fit |
| `@Repository` (implicit via `JpaRepository`) | `repository/` interfaces | Marks a data-access interface; Spring Data JPA auto-generates the implementation at runtime — you never write the SQL |
| `@Entity` | every file in `model/` (except enums) | Marks a plain class as mapped to a database table |
| `@Table(name = "users")` | on entities | Explicitly names the SQL table (otherwise Hibernate guesses from the class name) |
| `@Id` / `@GeneratedValue` | on the id field | Marks the primary key and tells the DB to auto-increment it |
| `@Column`, `@Enumerated`, `@ManyToOne`, `@ElementCollection` | on entity fields | Fine-tune how a Java field maps to a SQL column or relationship (e.g. `Submission.user` → a foreign key to the `users` table) |
| `@Configuration` | `SecurityConfig.java` | Marks a class that produces `@Bean`s (manually configured objects) instead of being auto-scanned |
| `@Bean` | inside `SecurityConfig` | Manually registers an object (like `PasswordEncoder`) into Spring's container so it can be injected elsewhere |
| `@EnableWebSecurity` / `@EnableMethodSecurity` | `SecurityConfig.java` | Turns on Spring Security's filter chain and enables `@PreAuthorize` checks respectively |
| `@Transactional` | e.g. `CodingTestService`, `BatchSectionSeeder` | Wraps a method in a database transaction — if anything throws an exception mid-method, **all** DB changes in that method are rolled back automatically |
| `@Cacheable("allQuestions")` / `@CacheEvict` | `QuestionService.java` | Caches a method's return value in memory so repeated calls skip re-querying the DB; evicted (cleared) whenever a question is created/edited |
| `@RestControllerAdvice` + `@ExceptionHandler` | `GlobalExceptionHandler.java` | A single class that catches exceptions thrown *anywhere* in any controller and converts them into a clean JSON error response with the right HTTP status |
| `@Value("${jwt.secret}")` | e.g. `JwtService.java` | Injects a value from `application.properties` (or an environment variable override) directly into a field |

---

## 4. Dependency Injection in Practice

You'll notice almost every class in `service/` and `controller/` looks like this:

```java
@Service
public class PlagiarismService {
    private final SubmissionRepository submissionRepository;
    private final PlagiarismReportRepository reportRepository;

    public PlagiarismService(SubmissionRepository submissionRepository, PlagiarismReportRepository reportRepository) {
        this.submissionRepository = submissionRepository;
        this.reportRepository = reportRepository;
    }
}
```

There's no `@Autowired` annotation here — that's intentional and is the **modern** Spring style. When a class has exactly **one constructor**, Spring automatically injects the required beans through it. You never call `new PlagiarismService(...)` yourself; Spring builds it once at startup and hands the same instance to anything that needs it.

> **Why this matters for debugging:** if Spring can't find a bean to inject (e.g. you forgot `@Service` on a class, or there are two beans of the same type with no way to pick), the app **fails at startup** with a `NoSuchBeanDefinitionException` or `NoUniqueBeanDefinitionException` — never at runtime when a button is clicked. See `DEBUGGING.md` §2.

---

## 5. JPA / Hibernate — How Java Objects Become SQL Tables

`spring.jpa.hibernate.ddl-auto=update` in `application.properties` means: on every startup, Hibernate looks at your `@Entity` classes and **automatically creates/alters** database tables to match them. This is why you never had to hand-write `CREATE TABLE` statements for `users`, `questions`, `submissions`, etc. — they're generated from `User.java`, `Question.java`, `Submission.java`.

Relationships map like this:

```java
@ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "user_id")
private User user;
```

This single field on `Submission` becomes a `user_id` foreign-key column pointing at the `users` table. `FetchType.LAZY` means Hibernate won't actually load the full `User` row from the DB until you call `submission.getUser()` — it saves memory/queries when you don't need the related data.

> **Why this matters for debugging:** if you try to access a `LAZY` relationship *after* the database session has closed (e.g. inside a JSON serializer, outside a `@Transactional` method), you'll hit the infamous `LazyInitializationException`. See `DEBUGGING.md` §5.

---

## 6. Spring Security + JWT — How Login/Auth Actually Works

This app uses **stateless** authentication — the server keeps **zero** session state. Every single request must carry proof of identity (the JWT) in its headers.

```
1. User logs in with username+password
     → AuthController.login() → AuthService.login()
     → Spring's AuthenticationManager checks the password against the BCrypt hash in the DB
     → JwtService.generateToken(user) signs a token containing { username, role, expiry }
     → Token is returned to the browser as a string

2. Browser stores the token (localStorage, see AuthContext.jsx)

3. EVERY subsequent request:
     → Axios interceptor (api.js) attaches header: Authorization: Bearer <token>
     → JwtAuthenticationFilter (runs before your controller, on every request)
         - reads the header
         - JwtService validates the signature + expiry
         - loads the User from the DB via UserDetailsServiceImpl
         - tells Spring Security "this request is authenticated as this user"
     → Now @AuthenticationPrincipal and @PreAuthorize work in your controller
```

`SecurityConfig.java` is the rulebook that decides, for every URL pattern, who's allowed in:

```java
.requestMatchers("/api/auth/**").permitAll()                                    // anyone
.requestMatchers("/api/admin/**").hasRole("ADMIN")                               // admins only
.requestMatchers("/api/proctoring/**").hasAnyRole("TEACHER","ADMIN","COMPANY")   // staff only
.requestMatchers("/api/**").authenticated()                                      // everything else: must be logged in
```

The **filter chain** is the key mental model: before your `@RestController` method ever runs, the request passes through a chain of filters (CORS check → JWT parsing → authorization check). If any filter rejects it, your controller code never even executes.

---

## 7. DTOs vs Entities — Why Both Exist

You'll notice `CodingTest` (an `@Entity`) and `CodingTestDTO` (a plain class in `dto/`) both exist and look similar. This is deliberate:
- **Entities** (`model/`) are shaped around the *database* — they have lazy-loaded relationships, JPA annotations, and internal fields you don't want to expose.
- **DTOs** (`dto/`) are shaped around the *API contract* — exactly what JSON goes in/out over HTTP. Services convert between the two (`toDTO()` / `fromDTO()` style methods), so a database schema change doesn't automatically break the API, and sensitive fields (like a password hash) never accidentally leak into a JSON response.

---

## 8. Cheat Sheet (quick recall card)

| If you see... | It means... |
|---|---|
| `@RestController` | This class handles HTTP requests and returns JSON |
| `@Service` | This class holds business logic, injected into controllers |
| `extends JpaRepository<X, Long>` | Spring auto-generates `save()`, `findById()`, `findAll()`, `delete()` — plus any custom method you declare by name, e.g. `findByUsername(String username)` |
| `@Entity` | This class = one database table |
| `@Autowired`-free constructor | Spring still injects dependencies — just implicitly, via the single constructor |
| `@PreAuthorize(...)` | "You need this role to even reach this method" |
| `@Transactional` | "If anything fails halfway through, undo all DB changes made so far in this method" |
| `@Valid` on a `@RequestBody` | "Reject the request with 400 before my code even runs, if the DTO's validation annotations fail" |
| `ResponseEntity<X>` | The wrapper controllers return — lets you control the HTTP status code + body together |
| `${SOME_VAR:default}` in `.properties` | "Use env var `SOME_VAR` if set, otherwise fall back to `default`" |

---

## Where to Go Next

- **`PROJECT_FLOW.md`** — see these exact concepts fire in sequence for real buttons (Login, Run Code, Submit Test...)
- **`DEBUGGING.md`** — what to do when any of this breaks
- **`FOLDER_STRUCTURE.md`** — every file's individual purpose

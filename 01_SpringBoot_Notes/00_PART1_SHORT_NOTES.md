# Part 1: Short Notes (only important stuff)

Short and easy version of all 4 files in this folder.
Focus = what Windy Street (JD) asks + what is on your resume.

> **JD wants:** REST APIs, database design, React, debugging, Git, testing, cloud (AWS), CI/CD.
> **Your resume has:** Spring Boot + MySQL at Telgoo5, CodeCampus project, AWS + Terraform pipeline.
> Every section below has a 👉 line: how to connect it to your work.

---

## 1. What is Spring Boot? (easy words)

Spring Boot is a Java tool to build backend apps fast.
You write normal Java classes and put **annotations** (like `@Service`) on them.
Spring **creates the objects and connects them** for you.

- **Bean** = an object Spring creates and manages. Made once, used everywhere.
- **IoC** = Spring makes the objects, not you. (No `new MyService()`.)
- **DI (Dependency Injection)** = Spring **gives** your class the objects it needs, through the constructor.

```java
@Service
public class OrderService {
    private final OrderRepository repo;
    public OrderService(OrderRepository repo) {  // Spring passes repo here
        this.repo = repo;
    }
}
```

When the app starts, Spring Boot:
1. Starts a built-in web server (**Tomcat**). No separate server needed.
2. Finds all classes with annotations and makes beans.
3. Reads settings from `application.properties`.

👉 *"I use Spring Boot every day at Telgoo5 to build REST APIs for inventory."*

---

## 2. The 4 Layers (asked in almost every interview)

```
Request → Controller → Service → Repository → Database
```

| Layer | What it does | Example |
|---|---|---|
| **Controller** | Gets the request, sends back JSON | `@RestController` |
| **Service** | Business logic (rules, checks, calculations) | `@Service` |
| **Repository** | Reads/writes the database | `extends JpaRepository` |
| **Entity** | A Java class = a DB table | `@Entity` |

**Why separate?** Each layer does one job. Easy to read, test and change.

👉 *JD link:* their FDD platform will have the same layers: e.g. upload GL file (controller) → check and map accounts (service) → save rows (repository).

---

## 3. Annotations You Must Know

| Annotation | Easy meaning |
|---|---|
| `@SpringBootApplication` | Start point of the app |
| `@RestController` | This class handles API calls and returns JSON |
| `@RequestMapping("/api/orders")` | Base URL for the class |
| `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` | Read, create, update, delete |
| `@PathVariable` | Take value from URL: `/orders/5` → `5` |
| `@RequestParam` | Take value from `?page=2` |
| `@RequestBody` | Turn JSON body into a Java object |
| `@Valid` | Check input (e.g. `@NotBlank`, `@Email`). Bad input → **400** |
| `@Service` / `@Component` / `@Repository` | "Spring, please manage this class" |
| `@Configuration` + `@Bean` | Make an object by hand and give it to Spring |
| `@Entity`, `@Id`, `@GeneratedValue` | Table, primary key, auto number |
| `@ManyToOne` + `@JoinColumn` | Foreign key (link to another table) |
| `@Transactional` | All or nothing: if one step fails, **undo all DB changes** |
| `@PreAuthorize("hasRole('ADMIN')")` | Only this role can call it, else **403** |
| `@Cacheable` / `@CacheEvict` | Keep result in memory / clear it after an update |
| `@RestControllerAdvice` | One place to handle all errors |
| `@Value("${key}")` | Read a value from properties file |
| `@Scheduled(cron = "0 0 2 * * *")` | Run a job on a timer (here: every day 2 AM) |

👉 *Resume link:* `@Scheduled` = your **cron jobs** at Telgoo5 (sync, reports, reconciliation). `@Transactional` = needed for **money data** in their FDD product.

---

## 4. Database with JPA / Hibernate

- **JPA/Hibernate** = turns Java objects into SQL for you.
- `JpaRepository` gives free methods: `save()`, `findById()`, `findAll()`, `delete()`.
- Custom query just by name: `findByTenantIdAndStatus(...)`. Spring writes the SQL.
- `ddl-auto=update` = auto-create tables from entities. Good for dev only. In production use **Flyway/Liquibase** (migration scripts).

**LAZY vs EAGER**
- LAZY = load linked data **only when you use it** (saves memory).
- EAGER = load everything at once.

**2 common problems:**
| Problem | Easy meaning | Fix |
|---|---|---|
| **N+1 query** | 1 query for list + 1 extra query for each row = slow | `JOIN FETCH` |
| **LazyInitializationException** | Used LAZY data after DB session closed | Convert to DTO inside `@Transactional` |

👉 *Resume link:* "I optimized MySQL queries: used `EXPLAIN`, added indexes on `tenant_id`, `status`, `created_at`, used pagination, fixed N+1 with `JOIN FETCH`."
👉 *JD link:* for money use `BigDecimal` in Java and `DECIMAL(18,2)` in DB. **Never `double`.**

---

## 5. Login: Spring Security + JWT (very important)

**JWT** = a signed token. It has username, role and expiry. Server does not store sessions (**stateless**).

```
LOGIN
1. User sends username + password
2. Server checks password with BCrypt hash
3. Server makes a JWT and sends it back
4. Browser saves the token

EVERY NEXT REQUEST
5. Browser sends header: Authorization: Bearer <token>
6. JWT filter checks the token (signature + expiry)
7. Role is checked (@PreAuthorize)
8. Only then the controller runs
```

Easy points to say:
- Passwords saved as **BCrypt hash**, never plain text.
- CSRF is off because we use tokens, not cookies.
- **Filter chain** = checks that run before the controller. If one fails, request stops.

👉 *Resume link:* CodeCampus has 4 roles (Admin, Teacher, Student, Company) with JWT + `@PreAuthorize`.
👉 *JD link:* their app has roles too: **Staff / Manager / Partner**. Same idea.

---

## 6. DTO vs Entity

- **Entity** = matches the DB table.
- **DTO** = matches the JSON the API sends/receives.

**Why both?** If the DB changes, the API doesn't break. Secret fields (like password) never go out.

---

## 7. How a Request Flows (say this for "explain your project")

```
React button click
 → Axios sends API call with JWT
 → Spring Security checks token + role
 → Controller → Service → Repository → MySQL
 → JSON comes back → React updates the screen
```

Ports: React **5173** → Spring Boot **8080** → MySQL **3306**.
The frontend **never** talks to the DB directly.

---

## 8. Multi-Tenant + Cron Jobs (your Telgoo5 strength, JD loves this)

**Multi-tenant** = one app, many customers. Each customer sees only its own data.
- Every table has `tenant_id`.
- Tenant comes from the logged-in user's token.
- Every query filters by `tenant_id`, done in one central place so nobody forgets.

👉 *JD link:* each CPA firm / client must see only its own data. Same problem.

**Cron jobs:**
- **Idempotent** = run it 2 times, result is same as 1 time. (Use unique keys, upsert.)
- **Overlap guard** = if the old run is still going, don't start a new one. (Use a lock, e.g. **ShedLock**, important when many servers run.)

👉 *JD link:* GL import must never create duplicate rows if run again = idempotent.

---

## 9. External API Calls (your FedEx work)

- Set a **timeout** on every call.
- **Retry with backoff** (wait 1s, 2s, 4s...) only for temporary errors (5xx, timeout). Not for 4xx.
- Still failing? Save as "pending", log it, a job retries later. Data is never lost.
- Cache the OAuth token, refresh before it expires.

👉 *JD link:* they connect to **QuickBooks / Xero / NetSuite** APIs. Same pattern.

---

## 10. CodeCampus Project (quick facts)

| Feature | Easy explanation |
|---|---|
| **Run code** | Code runs in **Judge0 (Docker sandbox)**, 10 sec timeout, max 12 runs at once (Semaphore). Not saved |
| **Submit code** | Runs all test cases (also hidden ones), saves result + score |
| **Submit test** | Can submit only once. Second time → **409** |
| **Proctoring** | Detects tab switch, fullscreen exit, takes webcam photos. Too many tab switches → auto-submit |
| **Plagiarism** | Mix of 3 checks: Jaccard + N-gram + LCS. Above 70% → flagged |
| **MCQ** | Answer checked **on server**. Never send correct answer to browser |
| **Forgot password** | Random token, valid 30 min, sent by email |
| **Cache** | Question list cached, cleared when a question is edited |

👉 *JD link:* plagiarism text matching = **account mapping** ("Rent - Warehouse" → "Rent Expense"). Mention this!

---

## 11. HTTP Status Codes

| Code | Easy meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 400 | Your input is wrong |
| 401 | Not logged in / token expired |
| 403 | Logged in, but not allowed |
| 404 | Not found |
| 409 | Conflict (already exists / already submitted) |
| 500 | Server error, check logs |

**401 vs 403:** 401 = "Who are you?" 403 = "I know you, but you can't enter."

---

## 12. Debugging (JD: "debug issues across front end, back end, integrations")

Check in this order:
1. **Browser Console**: any JavaScript error?
2. **Network tab**: what was sent? what status came back? (solves most bugs)
3. **Backend logs**: read the **bottom** of the stack trace
4. **Database**: is the data really saved?

| Error | Easy meaning |
|---|---|
| `Port 8080 already in use` | Old app still running, stop it |
| `NoSuchBeanDefinitionException` | Forgot `@Service` / `@Component` |
| `NoUniqueBeanDefinitionException` | 2 beans of same type, use `@Primary` / `@Qualifier` |
| `Communications link failure` | MySQL is down or wrong password |
| `DataIntegrityViolationException` | Duplicate value (e.g. same email) |
| `ExpiredJwtException` | Token too old, login again |

👉 *Story to tell (STAR):* "An API was slow for one tenant. I checked CloudWatch logs and slow query log, found a missing index on a big table, added a composite index, and response time dropped from X sec to Y ms."

---

## 13. Testing (JD asks, prepare this)

- **Unit test** = test one class alone. Use **JUnit + Mockito** (fake the repository).
- **Integration test** = test with real Spring + DB. Use `@SpringBootTest`.
- **Postman** = test APIs by hand.

```java
@Test
void returnsOrder() {
    when(repo.findById(1L)).thenReturn(Optional.of(new Order(1L)));
    assertEquals(1L, service.getOrder(1L).getId());
}
```

---

## 14. Deploy (your AWS pipeline project)

"On every PR, **GitHub Actions** builds and tests. On merge, it logs into AWS with **OIDC** (no stored keys), builds a **Docker** image tagged with the **git commit ID**, pushes to **ECR**, deploys to **ECS Fargate** behind a **load balancer**. If the new version is unhealthy, it **rolls back automatically**. All infra is written in **Terraform**."

👉 *JD link:* JD wants CI/CD + AWS. This is a big plus. Say it with confidence.

---

## 15. 30-Second Recap (read before the interview)

- Spring Boot = annotations + Spring wires objects (DI).
- 4 layers: Controller → Service → Repository → Entity.
- `@Transactional` = all or nothing. Money = `BigDecimal`.
- JWT = stateless login; 401 = not logged in, 403 = not allowed.
- Fix slow SQL: `EXPLAIN`, indexes, pagination, `JOIN FETCH`.
- Multi-tenant = `tenant_id` on every query.
- Cron jobs = idempotent + overlap lock.
- API calls = timeout + retry with backoff.
- Debug order: Console → Network → Logs → DB.

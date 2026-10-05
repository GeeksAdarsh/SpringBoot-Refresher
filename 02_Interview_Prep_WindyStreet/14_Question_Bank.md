# Question Bank: Basic, Project and Scenario Questions (with Answers)

Questions come in 3 types:
1. **Basic questions:** "What is a REST API?", "How does a website work?"
2. **Project questions:** taken from your resume.
3. **Scenario questions ("what if..."):** "What will you do if the API is slow?"

Answers are short so you can say them in 20 to 40 seconds. Read the question, try answering **out loud first**, then check.

> 🎯 Where you see `[ ]`, put your real numbers or details. Real numbers make answers believable.

---

# PART A: How a Website Works (Basic Flow)

### A1. What happens when you type a URL (like `www.google.com`) and press Enter?
1. **DNS lookup:** the browser finds the IP address of the domain (like a phone book).
2. **TCP connection** to that IP, then a **TLS handshake** for HTTPS (sets up encryption).
3. The browser sends an **HTTP request** (`GET /`).
4. The request may pass a **CDN / load balancer**, then reaches the **web server** (Nginx) and the **app server** (Spring Boot / Node).
5. The backend runs logic, reads the **database**, and sends back an **HTTP response** (HTML/JSON + status code).
6. The browser **renders** the page: reads HTML → builds the DOM → applies CSS → runs JS → loads images, etc.

### A2. Explain the full flow of a full-stack app (frontend → backend → database).
> "The user clicks a button in React. React calls an API using Axios/fetch, for example `POST /api/orders`, with JSON and the JWT token. The request reaches Spring Boot. The security filter checks the token, the controller receives it, the service applies business logic, and the repository saves to MySQL. The response comes back as JSON with a status code, and React updates the state so the UI changes."

**Short form:** `UI → API call → Security → Controller → Service → Repository → DB → JSON → UI update`

### A3. Frontend vs Backend vs Database?
- **Frontend:** what the user sees (React, HTML, CSS). Runs in the browser.
- **Backend:** business logic, security, APIs (Spring Boot, Node). Runs on the server.
- **Database:** stores data permanently (MySQL, MongoDB).

### A4. HTTP vs HTTPS?
HTTPS = HTTP + **TLS encryption**. Data can't be read or changed in the middle. It uses an SSL/TLS certificate. Port 80 (HTTP) vs 443 (HTTPS).

### A5. What is DNS?
Converts a domain name (`google.com`) into an IP address (`142.250.x.x`).

### A6. What is a web server vs an application server?
- **Web server** (Nginx, Apache): serves static files, handles HTTPS, forwards requests (reverse proxy).
- **App server** (Tomcat inside Spring Boot): runs your Java code.

### A7. What is a reverse proxy / load balancer?
- **Reverse proxy:** sits in front of the backend, receives all requests and forwards them (Nginx).
- **Load balancer:** spreads requests over many servers (AWS ALB) so no single server gets overloaded.

### A8. Cookies vs Session vs localStorage vs JWT?
| | Where stored | Notes |
|---|---|---|
| Cookie | Browser, sent with every request automatically | Can be `HttpOnly` (JS can't read it, safer) |
| Session | Server stores user data; browser holds a session id cookie | Easy logout, needs server memory |
| localStorage | Browser only, JS can read it | Stays after closing; not sent automatically |
| JWT | A signed token (often in localStorage or a cookie) | Server doesn't store it; stateless |

### A9. What is CORS? How did you fix it?
The browser blocks a page from `localhost:5173` calling an API on `localhost:8080` (a different origin) unless the server allows it. Fix it on the **backend**: a CORS config in Spring Security (`allowedOrigins`), or use a **proxy** (the Vite proxy in CodeCampus, Nginx in production).

### A10. Client-side rendering vs server-side rendering?
- **CSR (React + Vite):** the browser downloads JS and builds the page. Fast after the first load.
- **SSR (Next.js):** the server sends ready HTML. Faster first load and better for SEO.
- **SSG:** HTML is built once at build time (fastest, for static pages).

### A11. What is a CDN?
Servers around the world that keep copies of static files (JS, CSS, images) close to users so they load faster (CloudFront).

### A12. What is caching? Where can you cache?
Saving results so you don't compute them again. Layers: **browser cache**, **CDN**, **app cache (Redis)**, **DB query cache**.

---

# PART B: REST API Basic Questions

### B1. What is an API?
A way for two programs to talk. For example, the frontend asks the backend for data through an API.

### B2. What is REST? What are its rules?
REST = a style for building APIs over HTTP. Rules:
1. **Client-server:** frontend and backend are separate.
2. **Stateless:** each request has everything needed (token); the server doesn't remember the last request.
3. **Resources by URL:** `/api/users/5`.
4. **Standard methods:** GET, POST, PUT, PATCH, DELETE.
5. **Cacheable** responses.
6. **Uniform interface:** the same style everywhere, usually JSON.

### B3. REST vs SOAP?
REST: light, JSON, uses HTTP methods, easy. SOAP: XML only, strict rules, heavier (older banking systems).

### B4. GET vs POST?
GET reads data; data goes in the URL; can be cached; idempotent. POST creates data; data goes in the body; not idempotent.

### B5. PUT vs PATCH?
PUT replaces the **whole** resource. PATCH updates **only some** fields.

### B6. What is idempotent?
Doing it once or many times gives the same result. GET, PUT, DELETE are idempotent; POST is not.

### B7. Path variable vs Query parameter vs Request body?
- **Path:** `/users/5` identifies one resource → `@PathVariable`
- **Query:** `/users?page=1&city=Delhi` for filtering/sorting/paging → `@RequestParam`
- **Body:** JSON data for create/update → `@RequestBody`

### B8. What are HTTP headers? Give examples.
Extra information with the request/response: `Content-Type: application/json`, `Authorization: Bearer <token>`, `Accept`, `Cache-Control`.

### B9. Important status codes?
200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests, 500 Server Error, 502 Bad Gateway, 503 Service Unavailable.

### B10. 401 vs 403?
401 = not logged in / bad token. 403 = logged in but not allowed.

### B11. How do you design a good REST API?
Nouns in URLs (`/orders`, not `/getOrders`), correct methods and status codes, pagination, filtering, versioning (`/api/v1`), validation, the same error format everywhere, DTOs, security, and Swagger documentation.

### B12. How do you secure an API?
HTTPS, JWT/OAuth authentication, role-based authorization, input validation, rate limiting, no secrets in code, parameterized queries (stops SQL injection), CORS rules, logging and monitoring.

### B13. What is pagination? Why?
Return data in small pages (`?page=0&size=20`) instead of lakhs of rows at once. It saves memory, speeds up the response, and makes the UI faster. In Spring: `Pageable` → `Page<T>`.

### B14. How do you version an API?
URL (`/api/v1/users`), header (`Accept-Version: v1`), or query param. The URL way is the most common.

### B15. How do you handle errors in Spring Boot?
Custom exceptions plus `@RestControllerAdvice` with `@ExceptionHandler`, returning the same JSON error format with the right status code.

### B16. How do you validate input?
`@Valid` on `@RequestBody` with annotations like `@NotNull`, `@NotBlank`, `@Email`, `@Size`, `@Positive` on the DTO. Errors → 400.

### B17. What is Swagger / OpenAPI?
Auto-generated API docs where you can test APIs in the browser. Spring: `springdoc-openapi`, open `/swagger-ui.html`.

### B18. How do you test an API?
Postman (manual), JUnit + MockMvc (controller tests), integration tests with a real DB (Testcontainers), and Swagger UI.

### B19. What is rate limiting?
Limiting how many requests a user can make (e.g. 100 per minute). Extra requests get 429. Protects against abuse.

### B20. What is an API Gateway?
A single entry point in front of many services: routing, auth, rate limiting, logging (AWS API Gateway, Spring Cloud Gateway).

### B21. Synchronous vs asynchronous API?
- **Sync:** the client waits for the result (normal GET).
- **Async:** the server says "accepted" (202) and does the work in the background; the client checks status later or gets a webhook.

### B22. What is a webhook?
The other system **calls your API** when something happens (e.g. FedEx or a payment gateway notifies you that a status changed). The opposite of polling.

### B23. What is JWT? What's inside?
JSON Web Token = `header.payload.signature`. The payload has the user id, role and expiry. The signature (made with a secret key) proves it wasn't changed. It's **encoded, not encrypted**, so never put passwords in it.

### B24. Access token vs refresh token?
Access token: short life (15 to 60 minutes), sent on every request. Refresh token: long life, used only to get a new access token.

### B25. What is OAuth 2.0?
A standard for letting an app access another service on the user's behalf without sharing the password ("Login with Google", the FedEx/QuickBooks APIs). The app gets an access token.

---

# PART C: Spring Boot and Java Basic Questions

### C1. What is Spring Boot? Why use it over Spring?
Spring Boot = Spring + **auto-configuration** + an **embedded server** (Tomcat) + **starters**. No XML, quick setup, production-ready (Actuator).

### C2. What does `@SpringBootApplication` do?
It combines `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`.

### C3. What is Dependency Injection / IoC?
Spring creates objects (beans) and **gives** them to the classes that need them, instead of the class using `new`. This makes code loosely coupled and easy to test. Constructor injection is the best way.

### C4. `@Component` vs `@Service` vs `@Repository` vs `@Controller`?
All create beans. `@Service` = business logic, `@Repository` = DB (also converts DB exceptions), `@Controller`/`@RestController` = web layer. `@Component` = general.

### C5. `@Controller` vs `@RestController`?
`@RestController` = `@Controller` + `@ResponseBody`, so it returns JSON directly instead of a view.

### C6. Bean scopes?
`singleton` (default, one per app), `prototype` (new each time), `request`, `session`.

### C7. What is `application.properties` / profiles?
Configuration (DB url, port). Profiles (`application-dev.yml`, `application-prod.yml`) give different settings per environment, chosen with `spring.profiles.active`.

### C8. What is JPA vs Hibernate vs Spring Data JPA?
JPA = the rules (specification). Hibernate = the tool that follows the rules. Spring Data JPA = makes it easy (`JpaRepository`, method names like `findByEmail`).

### C9. `save()` vs `saveAndFlush()`?
`save` may wait to write to the DB until the transaction ends; `saveAndFlush` writes immediately.

### C10. What is `@Transactional`?
All DB operations inside succeed together or roll back together. By default it rolls back on unchecked exceptions (RuntimeException).

### C11. Lazy vs Eager loading? N+1 problem?
Lazy loads related data when needed; Eager loads it right away. N+1 = 1 query for a list + 1 query per item. Fix: `JOIN FETCH` / `@EntityGraph`.

### C12. How does Spring Security work?
A **filter chain** checks every request before the controller. In my project, a `JwtAuthenticationFilter` reads the token, validates it, and sets the user in the `SecurityContext`. Then URL rules and `@PreAuthorize` check roles.

### C13. What is Spring Boot Actuator?
Ready-made endpoints for monitoring: `/actuator/health`, `/metrics`, `/info`.

### C14. OOP 4 pillars (with an example)?
- **Encapsulation:** private fields + getters/setters (a `User` class hides its password field).
- **Inheritance:** `Admin extends User`.
- **Polymorphism:** the same method behaves differently (`PaymentService.pay()` for card vs UPI).
- **Abstraction:** show only what's needed (an interface `NotificationService` with Email/SMS implementations).

### C15. Interface vs Abstract class?
Interface: only a contract (can have default methods); a class can implement many. Abstract class: can have state and constructors; a class extends only one.

### C16. `==` vs `.equals()` in Java?
`==` compares references (same object); `.equals()` compares values. Always override `hashCode()` along with `equals()`.

### C17. String vs StringBuilder?
String is **immutable** (a new object on every change). StringBuilder is mutable and faster for many changes (loops).

### C18. Checked vs Unchecked exceptions?
Checked = the compiler forces you to handle them (`IOException`). Unchecked = runtime errors (`NullPointerException`, `IllegalArgumentException`).

### C19. ArrayList vs LinkedList? HashMap vs HashSet?
ArrayList: fast read by index. LinkedList: fast insert/delete in the middle. HashMap: key → value. HashSet: unique values only (uses a HashMap inside).

### C20. Java 8 features?
Lambdas, Streams API, Optional, functional interfaces, default methods, the new Date/Time API.

```java
List<String> names = users.stream()
    .filter(u -> u.getAge() > 18)
    .map(User::getName)
    .sorted()
    .collect(Collectors.toList());
```

### C21. SOLID principles (short)?
- **S**ingle responsibility: one class, one job.
- **O**pen/closed: add new features without changing old code (new adapter class).
- **L**iskov: a child class can replace its parent safely.
- **I**nterface segregation: small interfaces, not one big one.
- **D**ependency inversion: depend on interfaces, not concrete classes (DI does this).

### C22. Design patterns you've used?
Singleton (Spring beans), Factory, Builder (Lombok `@Builder`), Strategy (different payment/shipping methods), Adapter (third-party API wrappers), Repository, DTO.

### C23. Monolith vs Microservices?
Monolith = one deployable app (simple). Microservices = many small services that talk over APIs (scale and deploy separately, but more complex: network calls, tracing, data consistency).

---

# PART D: React and Frontend Basic Questions

### D1. What is React? Why use it?
A JS library for building UI with reusable components. It uses a Virtual DOM for fast updates and has a big ecosystem.

### D2. State vs Props?
Props = data from the parent (read-only). State = the component's own data, which can change and causes a re-render.

### D3. What is useEffect? What does the dependency array do?
It runs side effects (API calls, timers). `[]` = run once; `[x]` = run when x changes; no array = every render. Return a function for cleanup.

### D4. How do you call an API in React?
`useEffect` + `axios`/`fetch`, keeping `loading`, `error` and `data` in state. In bigger apps, React Query.

### D5. Why do we need `key` in lists?
It helps React find which item changed. Use a unique id, not the index.

### D6. What is Redux? When do you use it?
A global store for shared state (cart, user). Flow: action → reducer → new state → UI updates. Use it when many components share complex state; otherwise use Context.

### D7. Context API vs Redux?
Context: simple sharing (auth, theme). Redux: big apps with lots of updates, middleware and dev tools.

### D8. How do you make React fast?
`React.memo`, `useMemo`, `useCallback`, lazy loading, pagination/virtualization for big lists, debounced search, avoiding unnecessary state.

### D9. How do you protect routes (logged-in only pages)?
A `ProtectedRoute` component checks the auth state/token; if the user isn't logged in, it redirects to `/login`. The backend must **also** check, because frontend checks can be bypassed.

### D10. Next.js vs React?
Next.js is a framework on top of React with routing, SSR/SSG, API routes and image optimization built in.

---

# PART E: Database Basic Questions

### E1. SQL vs NoSQL? When do you use which?
SQL: fixed structure, relations, ACID (orders, money, inventory). NoSQL: flexible JSON, easy horizontal scaling (catalogs, logs). I used MySQL at Telgoo5 and MongoDB in EasyShop.

### E2. What is normalization?
Splitting data into tables so it isn't repeated (1NF, 2NF, 3NF).

### E3. What is an index? Downside?
A lookup structure (B-tree) that makes searches fast. Downside: slower writes and extra space.

### E4. Primary vs Foreign key?
Primary = unique id of a row. Foreign = a column pointing to another table's primary key.

### E5. Types of JOIN?
INNER (matches only), LEFT (all left + matches), RIGHT, FULL, SELF.

### E6. WHERE vs HAVING?
WHERE filters rows before GROUP BY; HAVING filters groups after it.

### E7. What is a transaction / ACID?
A group of DB steps treated as one: Atomic, Consistent, Isolated, Durable.

### E8. What is a stored procedure? (on your resume!)
SQL code saved in the DB that you call by name: `CALL get_low_stock(10);`. Good for repeated complex DB logic; the downside is that business logic gets split between the app and the DB.

### E9. How do you optimize a slow query? (on your resume!)
`EXPLAIN` → add the right index → avoid `SELECT *` → avoid functions on indexed columns in WHERE → paginate → fix N+1 → cache frequent reads in Redis.

### E10. What is Redis? Why use it?
An in-memory key-value store, very fast. Used for caching, sessions, rate limiting, and locks.

---

# PART F: Project Questions (from your resume)

## F1. CodeCampus (Coding and Assessment Platform)

1. **Explain CodeCampus in 1 minute.**
   > "A full-stack coding platform. Students solve problems and take timed contests and MCQ tests. Teachers create content, and companies can assess candidates. React frontend, Spring Boot backend, MySQL, JWT security with 4 roles. Code runs in Judge0 sandboxes in Docker in 5 languages. It also has proctoring and plagiarism detection."
2. **Draw the architecture.** React (Vite) → Spring Boot REST API → MySQL; Spring Boot → Judge0 (Docker) for code execution.
3. **Flow when a student clicks "Submit"?**
   > "React sends `POST /api/submissions` with the code, language and problem id plus the JWT. The filter checks the token and role. The controller calls the service, which saves the submission as PENDING and puts it into the thread pool. A worker thread sends the code with each test case to Judge0, compares the output with the expected output, and saves the result (passed count, time, status). The frontend polls or gets the final result and shows the verdict."
4. **Why a thread pool? Why bounded?** So 500 submissions at once don't create 500 threads or use up all memory. The bounded queue plus a rejection policy gives back-pressure.
5. **What if code runs an infinite loop?** Judge0's time limit kills it, plus my per-submission timeout (`future.get(timeout)`), so the worker thread is freed.
6. **How is untrusted code kept safe?** It runs in an isolated Docker sandbox with CPU, memory and time limits and no network access, never on the main server.
7. **How does login and JWT work?** Login → BCrypt password check → JWT with user id + role → frontend stores it → sends `Authorization: Bearer` → filter validates it on each request.
8. **How does role-based access work?** The role is in the JWT; `@PreAuthorize("hasRole('TEACHER')")` and URL rules in `SecurityConfig`. The frontend also hides buttons, but the backend is the real guard.
9. **How does tab-switch detection work?** The browser `visibilitychange` / `blur` events are sent to the backend and counted; after N switches, warn or auto-submit.
10. **Explain the plagiarism logic.** Normalize code (remove spaces, comments, rename variables to tokens) → Jaccard similarity of token sets + n-gram overlap + LCS ratio → a combined score → flag above a threshold like 80%.
11. **Database tables?** users, problems, test_cases, contests, submissions, mcq_questions, results, proctoring_events.
12. **Biggest challenge?** Handling many submissions at once during a contest without crashing → solved with a bounded thread pool, timeouts and an async result flow.
13. **What would you improve?** A message queue (RabbitMQ/SQS) instead of an in-memory queue so jobs survive a restart, WebSocket for live results, more tests, and monitoring.
14. **How would you scale it to 10,000 users?** Stateless backend behind a load balancer, more Judge0 workers, a queue, Redis cache for problem lists, DB indexes, and a CDN for the frontend.

## F2. EasyShop (E-Commerce, Next.js + MongoDB)

1. **Explain EasyShop.** A full-stack shop with login, product search/filter, cart and orders, using Next.js + Redux + Node + MongoDB, Dockerized, deployed via GitHub Actions.
2. **Flow when the user clicks "Place Order"?** Redux cart → `POST /api/orders` with JWT → server checks stock and price again (never trust the client price) → saves the order → reduces stock → returns the order id → clears the cart → shows the success page.
3. **How does product search work?** Query params (`?q=shoe&category=men&minPrice=500`) → MongoDB query with filters + text index → paginated result.
4. **Why MongoDB here?** Products have flexible attributes (size, color, specs), so a document model fits.
5. **What if two users buy the last item at the same time?** Use an atomic update: `updateOne({_id, stock: {$gte: 1}}, {$inc: {stock: -1}})`. Only one succeeds; the other gets "out of stock". (In SQL: a transaction with a row lock or optimistic locking.)
6. **Where is the cart stored?** In Redux on the client (+ localStorage for refresh), and synced to the DB for logged-in users.
7. **What is a rolling update / zero downtime?** New containers start and pass health checks before old ones stop.

## F3. AWS CI/CD Pipeline (Terraform, ECS Fargate)

1. **Explain the pipeline step by step.** PR → GitHub Actions builds and tests → merge → OIDC login to AWS → Docker build → push to ECR tagged with the git SHA → approval → deploy to ECS Fargate behind an ALB → circuit breaker rolls back if health checks fail.
2. **Why OIDC instead of access keys?** No long-lived secrets stored in GitHub; AWS gives short-lived credentials only to this repo/branch.
3. **What happens if the new version is broken?** ALB health checks fail → the ECS deployment circuit breaker stops it and rolls back to the last working task definition automatically.
4. **What is Terraform state?** A file tracking what Terraform created; kept in S3 with DynamoDB locking so two people don't apply at once.
5. **ECS vs EKS vs EC2?** ECS = AWS's simple container service; EKS = managed Kubernetes; EC2 = plain VMs you manage.
6. **What is ALB?** Application Load Balancer: routes HTTP traffic to healthy containers and does health checks.

## F4. Telgoo5 (Current Job)

1. **What does your company do?** It provides a telecom BSS/OSS platform (billing, orders, inventory, customer management) to MVNOs, which are mobile brands that rent network from big operators. It's multi-tenant SaaS.
2. **What exactly do you work on?** Inventory APIs for SIM cards and device trade-ins, warehouse and order flows, FedEx shipping integration, scheduled sync/reconciliation jobs, frontend deployment on Amplify, Lambda functions, CI/CD, **cloud infrastructure and cost optimization, and live production operations**.
3. **Explain the device trade-in flow.** Customer requests a trade-in → device details and condition saved → shipping label generated (FedEx) → device received at the warehouse → inspected/graded → credit given → inventory updated. *(Adjust to your real flow.)*
4. **What is multi-tenancy, and how is data kept separate?** One app, many MVNOs; every query is filtered by `tenant_id` taken from the logged-in context; plus role checks.
5. **How do you handle "lakhs of requests"?** Indexed queries, pagination, connection pooling (HikariCP), caching, stateless servers that scale horizontally, async/background jobs for heavy work.
6. **How does the FedEx integration handle failures?** Timeouts, retry with exponential backoff for 5xx/timeouts, no retry for 4xx, saving a failed status, and a scheduled retry job. Logged for support.
7. **Cron jobs: how do you prevent duplicates?** An overlap guard (lock/ShedLock) + idempotent logic (upsert, unique keys, last-processed marker).
8. **What is in your API documentation?** Endpoint, method, request/response examples, error codes, auth (Swagger + release notes).
9. **Biggest production issue you solved?** *(Prepare one real STAR story. See `10_HR_And_Project_Answers.md`.)*
10. **How does your team work?** Jira sprints, standups, a feature branch per ticket, PR with review, GitHub Actions checks, deploy to staging then production.

## F5. Cloud Infrastructure, Cost Optimization and Live Operations (Telgoo5)

> ✏️ Fill in your real numbers (e.g. "reduced the monthly AWS bill by about [X]%", "[N] services"). Talk only about what you really did.

1. **What cloud work did you do?**
   > "Apart from development, I worked on our AWS infrastructure: deploying and maintaining services, monitoring with CloudWatch, managing IAM access, and handling live production operations like deployments, incidents and log checks. I also worked on cost optimization to reduce our AWS bill."
2. **How did you reduce cloud cost? (most likely question)** Pick the ones you really did:
   - **Right-sizing:** checked CPU/memory in CloudWatch and moved over-sized EC2/RDS instances to smaller types.
   - **Stopping unused resources:** idle EC2 instances, unattached EBS volumes, old snapshots, unused Elastic IPs, idle load balancers.
   - **Scheduling:** dev/staging servers shut down at night and on weekends (Lambda + EventBridge cron).
   - **Auto scaling:** scale out at peak, scale in at low traffic, instead of always running at peak size.
   - **Savings Plans / Reserved Instances** for steady workloads; **Spot instances** for batch jobs.
   - **S3 lifecycle rules:** move old files to S3 Infrequent Access / Glacier, delete old logs.
   - **CloudWatch log retention:** keep logs for 30 days instead of forever.
   - **Serverless (Lambda)** for small, rarely used tasks instead of always-on servers.
   - **Data transfer:** CloudFront caching, keeping services in the same region/AZ.
   - **Tagging + Cost Explorer / AWS Budgets** to see which team/service costs what, with alerts.
3. **How did you find what was costing money?** AWS Cost Explorer grouped by service and tags, Trusted Advisor / Compute Optimizer suggestions, and CloudWatch usage metrics.
4. **Result?** "We reduced the monthly bill by about [X]% / [amount] without hurting performance."
5. **What are "live operations"?** Keeping production healthy: deployments and rollbacks, monitoring dashboards and alerts, checking logs, handling incidents, fixing data issues, running and watching scheduled jobs, and communicating with the support/delivery team.
6. **How do you monitor production?** CloudWatch metrics (CPU, memory, latency, 5xx errors), log groups, alarms → email/Slack, health check endpoints (`/actuator/health`).
7. **What do you do when an alert fires at night?** See part G, scenario G6.
8. **How do you deploy without downtime?** Rolling / blue-green deployments behind a load balancer with health checks; database changes made backward-compatible first.
9. **IAM best practices?** Least privilege, roles instead of access keys, MFA, no root account use, separate roles per service.
10. **What is a VPC / public vs private subnet?** A VPC is your private network in AWS. Public subnet: has internet access (load balancer). Private subnet: no direct internet (app servers, DB). The DB should always be private.
11. **How does this help Windy Street?** "A new platform's cloud bill grows fast with big data processing. I can help keep it efficient from day one, and I'm comfortable owning deployments and production support."

## F6. Rnpsoft Internship (Ms. Kalinga AI)

1. **What was Ms. Kalinga AI?** A voice-powered news assistant (1,000+ readers). Spring Boot REST APIs served news content to the frontend, with a voice output feature, deployed on Linux.
2. **What was your role?** Designing and building APIs, integrating them with the frontend, Postman testing, Linux deployment, and working in Agile sprints.
3. **How did you deploy on Linux?** Built the JAR with Maven → copied it to the server → ran it as a `systemd` service (or with `nohup java -jar`) → Nginx in front → checked the logs. *(Say what you really did.)*
4. **Where was the "AI" part?** *(Be honest: text-to-speech API, speech recognition, or summarization. Know exactly what was used.)*

---

# PART G: Scenario / "What If" Questions (Conditional Questions)

**Answer formula:** *First I check → then I find the cause → then I fix → then I prevent it from happening again.* Always mention **communication** (inform the team) for production issues.

### G1. An API that was fast is now very slow. What do you do?
1. Check monitoring: is it all APIs or one? Since when? Any recent deploy?
2. Check logs + CloudWatch: CPU, memory, DB connections, error rate.
3. Check the DB: slow query log, `EXPLAIN`, missing index, table grew big, locks.
4. Check external calls (e.g. FedEx slow?) → add timeouts.
5. Fix: index, pagination, cache, async processing, or roll back the recent deploy.
6. Prevent: alerts on latency, load tests.

### G2. The website is down in production. What do you do?
1. **Inform** the team/lead right away.
2. Check: health check, load balancer targets, recent deployment, server/container status, DB status.
3. If a recent deploy caused it → **roll back first**, investigate later.
4. Check logs for the error (out of memory, DB connection failure, config/secret missing).
5. After fixing: a root cause analysis (RCA) document + an action to prevent it.

### G3. It works on your machine but not in production.
Differences in: environment variables/config, DB data, versions (Java/Node), network/firewall/security groups, CORS/origin, file paths, time zone, memory limits. Compare configs and check production logs.

### G4. A user says "I clicked Pay once, but I was charged twice."
Cause: double click or retry with no protection. Fix: disable the button after the first click, plus an **idempotency key** on the backend (same key = return the old result, don't charge again), plus a unique constraint in the DB. Refund the duplicate and check logs to confirm.

### G5. Two users edit the same record at the same time. What happens? How do you stop overwriting?
Without protection, the last save wins and the first user's change is lost. Fix: **optimistic locking** (`@Version` column). The second save gets **409 Conflict** → show "This record was updated by someone else, please reload."

### G6. A cron job ran twice and created duplicate data.
Immediate: find and clean the duplicates (with a backup first). Fix: an overlap guard / distributed lock (ShedLock), an idempotent design (upsert, unique key), and a "last run" status table. This is exactly what I did at Telgoo5.

### G7. A third-party API (FedEx) is down. What does your system do?
Timeout → retry with exponential backoff → if it still fails, save the request as "pending", show the user "processing", and let a scheduled job retry later. A **circuit breaker** (Resilience4j) stops calling the failing API for some time so our system doesn't slow down. Alert the team.

### G8. A user uploads a very big file (1 GB) and the request times out.
Don't process it in the request. Upload directly to S3 (pre-signed URL) → return 202 + job id → a background worker processes it in chunks → the UI shows progress by polling. Set file size limits.

### G9. The database CPU is at 100%.
Find the top slow queries (slow query log / Performance Insights) → add indexes / fix bad queries → check for N+1 → add caching for repeated reads → read replicas for reporting queries → scale up only if needed.

### G10. The server runs out of memory (OutOfMemoryError).
Check heap usage in monitoring, take a heap dump, look for: loading too many rows at once (use pagination/streaming), caches with no limit, memory leaks (static collections, unclosed resources). Fix the code; then adjust `-Xmx` if truly needed.

### G11. The AWS bill suddenly went up 40% this month. What do you do?
Open Cost Explorer → group by service/tag → find what grew (e.g. a forgotten EC2, NAT gateway data, logs, S3) → check whether it's expected (more traffic) or waste → stop/right-size → set **AWS Budgets alerts** so the team knows early next time.

### G12. You pushed code and it broke the main branch / staging.
Tell the team immediately → revert the commit (`git revert`) to unblock others → fix it on a branch with a test → make sure CI runs tests before merging next time.

### G13. You're given a task you don't know how to do.
Understand the requirement → ask clarifying questions → research (docs, existing code) → make a small proof of concept → share the approach with a senior before building fully → give honest time estimates.

### G14. You can't finish the task by the deadline.
Tell the lead **early**, not on the last day. Explain the reason, what's done, and what's left. Suggest options: deliver the core part first, get help, or move the deadline.

### G15. A senior says your approach is wrong in a code review.
Listen and ask "why" to understand. If they're right, fix it and learn. If I have a good reason, explain politely with facts. The goal is the best code, not winning.

### G16. A requirement is unclear.
Don't guess. Ask the product owner with specific questions and examples, write down the agreed behavior in the ticket, and confirm edge cases before coding.

### G17. You find a bug in someone else's code in production.
If it's urgent → inform the lead/owner immediately with details (steps, logs, impact). If small → raise a ticket or fix it with a PR and tag the owner. Never blame.

### G18. A page with 1 lakh rows is very slow in the UI.
Backend pagination, server-side filtering/sorting, virtualized table on the frontend, debounced search, `useMemo` for totals, load only the needed columns.

### G19. Login works, but after some time all API calls give 401.
The JWT expired. Fix: an Axios interceptor catches 401 → calls the refresh token API → retries the request; if the refresh fails → logout and redirect to login.

### G20. A customer says their data is showing in another customer's account (multi-tenant leak).
This is **critical**: escalate immediately (security incident). Stop the leak (disable the feature / hotfix), find the query missing the `tenant_id` filter, fix it centrally, audit the logs for what was exposed, and add tests that check tenant isolation.

### G21. Deployment succeeded but users still see the old frontend.
Browser/CDN cache. Fix: file names with hashes (Vite does this), invalidate the CloudFront/Amplify cache, set correct cache headers for `index.html` (no-cache).

### G22. How would you design a feature from scratch? (e.g. "Add an export to Excel button")
Clarify the requirement (which data, format, size) → design the API (`GET /api/reports/{id}/export`) → backend generates the file (Apache POI / openpyxl), async if big → frontend button + loading state → tests → docs → PR.

### G23. (Windy Street style) Totals in the report don't match the source Excel by a small amount.
Likely: floating point rounding, missing or duplicate rows, a date/period filter difference, or wrong account mapping. Check row counts, compare totals per account/month to find where the difference starts, use DECIMAL types, and add a consistency check so it's caught automatically next time.

### G24. (Windy Street style) A new client's GL file has different column names than expected.
Don't hard-code columns. Use an adapter per source system plus a column-mapping step (auto-detect + let the user confirm the mapping), validate, and show clear row-level errors.

---

# PART H: Quick Rapid-Fire (1-line answers)

| Question | Answer |
|---|---|
| What is Maven? | A build tool: manages dependencies (`pom.xml`), builds the JAR |
| JAR vs WAR? | JAR runs alone (embedded Tomcat); WAR deploys to an external server |
| Default Spring Boot port? | 8080 |
| What is Lombok? | Generates getters, setters, constructors via annotations |
| What is Docker image vs container? | Image = template; container = running instance |
| What is Kubernetes? | Runs and manages many containers (scaling, healing) |
| What is Nginx? | Web server / reverse proxy / load balancer |
| What is a cron expression? | A schedule string, e.g. `0 */5 * * * *` = every 5 minutes (Spring) |
| What is Postman? | Tool to test APIs manually |
| What is Git? | Version control to track code changes and work in a team |
| What is CI/CD? | Auto build/test on every change; auto deploy after it passes |
| What is a microservice? | A small independent service that does one job and talks over APIs |
| What is a message queue? | A buffer between services for async work (SQS, RabbitMQ, Kafka) |
| What is Terraform? | Infrastructure as code |
| What is IAM? | AWS users, roles and permissions |
| What is Lambda cold start? | First call is slow because AWS starts a new container |
| What is S3? | AWS object/file storage |
| What is CloudWatch? | AWS logs, metrics and alarms |
| What is HikariCP? | Fast DB connection pool (Spring Boot default) |
| What is BCrypt? | A slow, salted password hashing algorithm |

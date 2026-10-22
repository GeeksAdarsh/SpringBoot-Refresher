# My Projects + Stories (Deep, Easy to Remember)

Based on your resume (`Adarsh_Singh_Full-Stack-Developer-01.pdf`).
Use this for: "Tell me about yourself", "Explain your project", "Tell me a problem you solved".

> ⚠️ **Very important:** The stories below are built from your resume points. **Change every `[ ]` to your real numbers and details** before the interview. If something did not happen exactly like this, change it to what really happened. Interviewers ask follow-up questions, and a true story is easy to defend.

---

## 0. Memory Map (remember only these words)

| # | Story name | Project | One-line memory |
|---|---|---|---|
| 1 | **SIM Module** | Telgoo5 | I built SIM inventory APIs from requirement to production |
| 2 | **Trade-in Module** | Telgoo5 | I built device trade-in flow: request → ship → inspect → credit |
| 3 | **Bulk Upload** | Telgoo5 | Lakhs of SIMs uploaded without crashing (batch insert) |
| 4 | **Safe Delete** | Telgoo5 | Hard delete lost data → I made soft delete + audit |
| 5 | **Slow API** | Telgoo5 | Missing index → added composite index → fast |
| 6 | **Double Job** | Telgoo5 | Cron job ran twice → lock + idempotent |
| 7 | **FedEx Down** | Telgoo5 | FedEx failed → retry + pending queue, no order lost |
| 8 | **Tenant Leak** | Telgoo5 | One query missed `tenant_id` → central filter |
| 9 | **Client Deadline** | Telgoo5 | Delivered module on time by splitting work in sprints |
| 10 | **Ms. Kalinga AI** | Rnpsoft | Voice news assistant for 1,000+ readers |
| 11 | **CodeCampus Engine** | Project | Thread pool + Docker sandbox for code runs |
| 12 | **AWS Pipeline** | Project | Push code → auto build → deploy → auto rollback |

Trick: **"Build, Upload, Delete, Slow, Double, FedEx, Leak, Deadline"**: 8 Telgoo5 stories.

---

## 1. Tell Me About Yourself (Infosys version, 60 sec)

> "Hi, I'm Adarsh Kumar Singh. I did my B.Tech in Computer Science from Sharda University. I'm a **Java backend developer** with about **one year of experience**.
>
> Right now I work at **Telgoo5** as a Full Stack Developer. Telgoo5 gives a telecom software platform to mobile companies called MVNOs. I work on the **inventory module**, mainly **SIM cards and device trade-in**. I build **REST APIs in Spring Boot**, design **MySQL tables**, optimize slow queries, integrated the **FedEx API** for shipping, and built **scheduled jobs** for sync and reconciliation. I also handle **production issues** and deploy with **GitHub Actions on AWS**.
>
> Before that I was a **Java intern at Rnpsoft**, where I built a voice-based news assistant in Spring Boot.
>
> I also built **CodeCampus**, a coding test platform with Spring Boot, JWT roles and a **multithreaded code runner**.
>
> I'm applying to Infosys because this role is **Java, Spring Boot and Microservices**, which is exactly my daily work, and I want to work on bigger enterprise projects."

---

## 2. Telgoo5: Full Project Explanation (Deep)

### 2.1 What is the company/product? (say it simply)

- Telgoo5 makes a **BSS/OSS platform** for **MVNOs**.
- **MVNO** = a mobile company that does not own towers. It rents network from big operators and sells SIMs under its own brand.
- **BSS/OSS** = the software to run a telecom business: customers, orders, billing, inventory, activation.
- **Multi-tenant** = one platform, many MVNO clients. Each client sees only its own data.

### 2.2 My part (what I own)

**Inventory module**, mainly 2 sub-modules:
1. **SIM card inventory:** add SIMs, assign SIMs to a client/warehouse/dealer, track status (in stock → assigned → activated → deactivated).
2. **Device trade-in:** a customer sends an old phone, we track it and give credit.

Plus: FedEx integration, cron jobs, production support, deployments.

### 2.3 Tech stack

| Layer | Tech |
|---|---|
| Backend | Java, Spring Boot, Spring MVC, Spring Data JPA, Hibernate, Spring Security |
| DB | MySQL |
| Frontend | React (some modules) |
| Jobs | Spring `@Scheduled`, crontab |
| Integration | FedEx REST API |
| Deploy | GitHub Actions, AWS (Amplify, Lambda, EC2, S3, CloudWatch) |
| Tools | Git, Maven, Postman, Swagger, Jira |
| Team | Agile sprints, PR code reviews, [team size: X] devs |

### 2.4 Architecture (draw this if they give a board)

```
React UI / Client systems
        │  REST (JSON + JWT)
        ▼
Spring Boot Inventory Service
  Controller → Service → Repository
        │                   │
        │                   ▼
        │                 MySQL (every table has tenant_id)
        │
        ├──► FedEx API (shipping labels, tracking)
        └──► Scheduled jobs (sync, reports, reconciliation)

Logs + monitoring: CloudWatch
```

### 2.5 Main tables (easy to remember)

| Table | Main columns |
|---|---|
| `sim_inventory` | id, tenant_id, iccid (SIM number), status, warehouse_id, assigned_to, created_at |
| `warehouse` | id, tenant_id, name, location |
| `inventory_order` | id, tenant_id, status, created_by, created_at |
| `trade_in_request` | id, tenant_id, customer_id, device_model, imei, status, credit_amount |
| `shipment` | id, order_id, tracking_number, label_url, status |
| `audit_log` | id, entity, entity_id, action, old_value, new_value, done_by, done_at |

**Indexes:** `(tenant_id, status)`, `(tenant_id, created_at)`, unique `iccid`, unique `imei`.

### 2.6 Main APIs I built (examples)

```
POST   /api/inventory/sims/bulk-upload     upload CSV of SIMs
GET    /api/inventory/sims?status=&page=   list with filters + pagination
PUT    /api/inventory/sims/{id}/assign     assign SIM to warehouse/dealer
DELETE /api/inventory/sims/{id}            soft delete
POST   /api/trade-in                       create trade-in request
PUT    /api/trade-in/{id}/inspect          update after inspection
GET    /api/shipments/{id}/track           FedEx tracking status
```

### 2.7 One full flow (say this when they ask "explain one flow")

**Device trade-in flow:**
1. Customer creates trade-in request → `POST /api/trade-in` → saved with status `REQUESTED`.
2. System calls **FedEx** to create a shipping label → label sent to customer → status `LABEL_CREATED`.
3. Cron job checks **FedEx tracking** every [X] min → status `IN_TRANSIT` → `RECEIVED`.
4. Warehouse inspects phone → `PUT /inspect` → status `INSPECTED`, credit amount set.
5. Credit given → status `COMPLETED`. Every change saved in `audit_log`.

Easy memory: **Request → Label → Transit → Received → Inspected → Credit**.

### 2.8 Achievements (put real numbers)

- Built **[X] REST APIs** for SIM and trade-in modules, handling **lakhs of requests** in production.
- Reduced slow API time from **[X] sec to [Y] ms** with indexes + pagination.
- Bulk upload of **[X] lakh SIMs** without memory issues.
- **Automated shipping** with FedEx: removed manual label creation for the ops team (saved about **[X] hours/week**).
- Built **[X] cron jobs**, zero duplicate runs after adding locks.
- Fixed **[X] production issues**; on-call / support for live releases.
- Delivered **[X] client features** on time in sprints.

---

## 3. Telgoo5 Stories (STAR format, easy words)

**STAR** = **S**ituation (what was the problem) → **T**ask (my job) → **A**ction (what I did) → **R**esult (what improved).
Say each story in about 1 minute.

### Story 1: SIM Module, built and delivered to the client
- **S:** A client (MVNO) needed a way to manage SIM cards: upload, assign to warehouses, and track status. Before this, they used Excel sheets.
- **T:** I was given the SIM inventory module from requirement to production.
- **A:**
  1. Joined requirement calls with the BA and understood the SIM statuses.
  2. Designed tables (`sim_inventory`, `warehouse`) with `tenant_id` and indexes.
  3. Built APIs: Controller → Service → Repository, with DTOs and validation (`@Valid`).
  4. Added role checks (only admin/warehouse users can assign).
  5. Tested with Postman, wrote JUnit tests for the service, raised a PR, fixed review comments.
  6. QA tested on staging, then released to production with GitHub Actions.
- **R:** Client moved from Excel to the system. [X] SIMs managed in the first month, no major bugs after release.

### Story 2: Trade-in Module (end to end ownership)
- **S:** Client wanted customers to send old phones and get credit.
- **T:** Build the full flow with shipping.
- **A:** Designed the status flow (Request → Label → Transit → Received → Inspected → Credit), built APIs, connected FedEx for labels and tracking, wrote a cron job to update tracking automatically, and saved each status change in `audit_log`.
- **R:** Trade-in went live for [client]. Ops team no longer tracked shipments by hand.

### Story 3: Bulk Upload, lakhs of SIMs
- **S:** Client wanted to upload [1–2] lakh SIMs from one CSV. The first version read the whole file into memory and saved one row at a time. It was slow and sometimes ran out of memory.
- **T:** Make it fast and safe.
- **A:**
  1. Read the file **line by line (streaming)** instead of all at once.
  2. Saved in **batches of 500/1000** using JDBC batch insert (`hibernate.jdbc.batch_size`).
  3. Validated each row (duplicate ICCID, wrong format) and gave an **error report** for bad rows instead of failing the whole file.
  4. Ran large uploads **in the background** and showed status (processing / done).
- **R:** Upload time went from **[X] min to [Y] min**, no memory crash, clear error report for the client.

### Story 4: Safe Delete (the delete problem)
- **S:** Our delete API did a **hard delete** (`DELETE FROM`). A client user deleted SIM records by mistake, and the data was gone. Also, other tables still pointed to those SIMs.
- **T:** Make delete safe and recoverable.
- **A:**
  1. Changed to **soft delete**: added `is_deleted` and `deleted_at`, `deleted_by` columns.
  2. Used Hibernate `@SQLDelete` + `@Where(clause = "is_deleted = false")` so normal queries hide deleted rows automatically.
  3. Blocked delete if the SIM is **active** or linked to an order (returns 409 Conflict).
  4. Saved every delete in `audit_log` (who, when, old value).
  5. Added a **restore** API for admins.
- **R:** No more data loss. When a client deleted something by mistake again, we restored it in minutes.

```java
@Entity
@SQLDelete(sql = "UPDATE sim_inventory SET is_deleted = true, deleted_at = NOW() WHERE id = ?")
@Where(clause = "is_deleted = false")
public class SimInventory { ... }
```

### Story 5: Slow API (performance optimization)
- **S:** The SIM list page for one big client took **[8] seconds**. Client complained.
- **T:** Find why and fix it.
- **A:**
  1. Checked **CloudWatch logs** and enabled the slow query log.
  2. Ran **`EXPLAIN`** on the query: it was a **full table scan** on lakhs of rows.
  3. Added a **composite index** on `(tenant_id, status, created_at)`.
  4. Added **pagination** (page of 50) instead of loading everything.
  5. Fixed an **N+1 problem**: used `JOIN FETCH` to get the warehouse in the same query.
  6. Returned a **DTO** with only needed fields, not the full entity.
- **R:** Response time went from **[8 sec] to [300 ms]**. We added "check EXPLAIN" to our PR checklist.

### Story 6: Double Job (cron job ran twice)
- **S:** The inventory sync job ran every [30 min]. Sometimes it took longer, so the next run started before the first finished. Also, we had [2] servers, and the job ran on both. This created duplicate records.
- **T:** Stop duplicate runs and keep data correct.
- **A:**
  1. Added **ShedLock** (a DB lock) so only one server runs the job at a time.
  2. Made the job **idempotent**: used unique keys + **upsert** (insert or update) and saved "last processed time", so running twice gives the same result.
- **R:** Zero duplicate records after the fix. Reconciliation reports became correct.

```java
@Scheduled(cron = "0 */30 * * * *")
@SchedulerLock(name = "inventorySync", lockAtMostFor = "PT25M")
public void syncInventory() { ... }
```

### Story 7: FedEx Down (third-party API failure)
- **S:** Sometimes the FedEx API was slow or returned 5xx errors. Label creation failed, and the user saw an error.
- **T:** Make shipping reliable even when FedEx has problems.
- **A:**
  1. Added **timeouts** on every FedEx call.
  2. **Retry with exponential backoff** (1s, 2s, 4s) only for 5xx/timeouts, not for 4xx (bad data).
  3. If still failing, saved the order as **`PENDING_SHIPMENT`** and a **cron job retried later**.
  4. **Cached the OAuth token** and refreshed it before expiry, which removed many extra calls.
  5. Logged every failure with order id for easy debugging.
- **R:** No order lost when FedEx was down. Failed label calls dropped by about **[X]%**.

### Story 8: Tenant Leak (data isolation bug)
- **S:** During testing, QA found that one report API showed records of **another client**. A developer forgot to add `tenant_id` in one custom query.
- **T:** Fix it and make sure it never happens again.
- **A:**
  1. Fixed the query immediately.
  2. Moved tenant filtering to **one central place** (Hibernate filter / base repository) where `tenant_id` comes from the logged-in user's token.
  3. Added a test that checks client A can never see client B's data.
  4. Added "tenant check" in the PR review checklist.
- **R:** Caught before production. No data leak since then.

### Story 9: Client Deadline (delivery under pressure)
- **S:** A client needed a new inventory report feature in [2 weeks] for their launch.
- **T:** Deliver on time without breaking things.
- **A:** Broke the work into small tasks in Jira, built the API first so QA could start early, had daily updates with the BA, raised small PRs for faster review, and tested on staging before release.
- **R:** Delivered **[1–2 days] before the deadline**. Client used it in their launch.

### Story 10: Production Bug at Night (live support)
- **S:** After a release, orders stopped moving to the "shipped" status in production.
- **T:** Find and fix it fast.
- **A:** Checked CloudWatch logs, found a `NullPointerException` because a new field was empty for old records. Did a quick fix with a null check + default value, tested on staging, and deployed. Then wrote a data fix for old records and added a unit test for this case.
- **R:** Fixed in **[X] hours**, no data lost. Lesson: always test with old data, not only new data.

---

## 4. Rnpsoft Internship: Ms. Kalinga AI

**What:** A voice-powered news assistant used by **1,000+ readers**.

**Easy explanation:**
> "It was a news app where users could listen to news by voice. I built the backend in **Spring Boot**: REST APIs to fetch news by category and latest news, and connected them to the frontend. The voice part used [text-to-speech API / tool you actually used]. I tested APIs in **Postman** and deployed on **Linux servers**."

**Flow:** User asks / clicks → frontend calls `GET /api/news?category=sports` → Spring Boot gets news from DB → returns JSON → voice reads it out.

**What I learned:** REST API design, Linux deployment, working in Agile sprints with Git.

**Story (STAR):**
- **S:** News API was slow when many readers opened the app in the morning.
- **A:** Added pagination and cached the latest news for a few minutes.
- **R:** Faster response during peak time.

> ⚠️ If they ask "which AI model?", say honestly what it really used. Don't over-claim.

---

## 5. CodeCampus (personal project, deep)

**One line:** "A coding test platform like HackerRank for colleges and companies."

**Tech:** Spring Boot, Spring Security + JWT, Spring Data JPA, MySQL, React, Docker, Judge0.

**Features:**
- 4 roles: Admin, Teacher, Student, Company (JWT + `@PreAuthorize`)
- 150+ problems, 50+ test cases each, timed contests, MCQ tests
- Code runs in **5 languages** inside **Docker sandbox (Judge0)**
- Proctoring: camera, tab-switch detection, auto-submit
- Plagiarism: Jaccard + n-gram + LCS score

**Flow (submit code):**
```
Student clicks Submit
 → POST /api/code/submit (with JWT)
 → Security checks token
 → CodeController → CodeExecutionService
 → Thread pool takes the task → sends to Judge0 (Docker)
 → Runs on all test cases, 10 sec timeout
 → Save Submission (score, status) → return result
```

**Story: CodeCampus Engine (multithreading)**
- **S:** In a live contest, [100+] students submit code at the same time. Running all at once would crash the server.
- **A:** Used a **bounded thread pool** (`ThreadPoolExecutor`: 4 core, 8 max, queue of 100). When full, `CallerRunsPolicy` slows down new requests instead of crashing. **Timeout** per submission. **Graceful shutdown** so running tasks finish. Untrusted code runs in **Docker (Judge0)**, so it can't harm the server.
- **R:** Handled contest load without crashing. Safe from bad code (infinite loops, file access).

**What I would improve:** move code runs to a **message queue** (Kafka/RabbitMQ) as a separate **microservice**, add Redis cache, more tests.

---

## 6. EasyShop (short)

> "A full stack e-commerce app with Next.js, Node.js, MongoDB and Redux: login with JWT, product search and filter, cart and orders. I containerized it with Docker and set up GitHub Actions for auto deploy with rolling updates, so there is no downtime during release."

Why MongoDB here: product data has flexible fields. (For money/orders in real life, SQL is safer because of transactions.)

---

## 7. AWS CI/CD Pipeline (short, strong for Infosys DevOps questions)

> "When I push code, **GitHub Actions** builds and runs tests. It logs into AWS using **OIDC**, so no AWS keys are stored in GitHub. It builds a **Docker image**, tags it with the **git commit ID**, pushes it to **ECR**, and deploys to **ECS Fargate** behind a **load balancer**. If the new version fails health checks, the **circuit breaker rolls back** automatically. All infrastructure is written in **Terraform**. Production release needs approval."

Memory: **Push → Test → Docker → ECR → ECS → Rollback if bad**.

If asked about **Jenkins** (Infosys JD): "Jenkins does the same job as GitHub Actions: a pipeline with stages like build, test, Docker build, deploy. In Jenkins we write it in a `Jenkinsfile`."

---

## 8. Common Follow-up Questions (short answers)

| Question | Answer |
|---|---|
| What was your role in the team? | Backend developer for inventory module; owned SIM + trade-in APIs, jobs, FedEx integration |
| Team size? | [X] developers, [X] QA, 1 BA, 1 lead |
| How do you get requirements? | From BA/Jira story → clarify in grooming → estimate → build in sprint |
| How do you test? | JUnit + Mockito for service, Postman for APIs, QA on staging |
| How is code reviewed? | PR on GitHub, at least 1 approval, fix comments, then merge |
| How do you deploy? | Merge to main → GitHub Actions builds/tests → deploy to staging → prod after approval |
| Monolith or microservices? | [Say what is real.] e.g. "Inventory runs as a separate Spring Boot service; it talks to other modules using REST APIs." |
| Hardest problem? | Story 3 (Bulk Upload) or Story 6 (Double Job) |
| A mistake you made? | Story 10: forgot old data case; now I always test with old data |
| How do you handle a production issue? | Check logs → reproduce on staging → quick fix → deploy → root cause fix + test |
| What did you learn at Telgoo5? | Writing code for real clients: performance, safe data, retries, monitoring |

---

## 9. Before the Interview: 5-Minute Revision

1. Say "Tell me about yourself" once out loud.
2. Say the **trade-in flow** (Request → Label → Transit → Received → Inspected → Credit).
3. Pick **3 stories**: Slow API, Double Job, Safe Delete.
4. Remember your **numbers** (fill the `[ ]` once and memorize).
5. CodeCampus: **thread pool + Docker sandbox**.

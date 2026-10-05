# My Resume vs the Windy Street JD

Interviewers ask questions **from your resume first**. Anything written there, you must be able to explain simply. This file goes through every resume point, shows how it matches the job, and gives likely questions with short answers.

---

## 1. Match Score: Where You Are Strong and Where to Prepare

| JD asks for | Your resume | Status |
|---|---|---|
| B.Tech CS/IT | B.Tech CSE, Sharda University | ✅ |
| 6 to 18 months real experience | Telgoo5 (Oct 2025 to now, about 12 months) + Rnpsoft internship (4 months) | ✅ Perfect fit |
| Strong in one language (Java/JS/Python) | Java primary, JS, Python, TypeScript | ✅ |
| React | React hooks, Redux, Next.js | ✅ |
| Backend REST APIs | Spring Boot, Express, production REST APIs | ✅ Strong |
| Databases + schema design | MySQL, MongoDB, Redis, query optimization | ✅ Strong |
| Git, PRs, code review | GitHub, PR reviews in Agile sprints | ✅ |
| Debugging, production issues | Production support across backend, DB, deployment | ✅ |
| Cloud (AWS preferred) | AWS (Amplify, Lambda, EC2, S3, CloudWatch, IAM), AWS Cloud Practitioner, cloud infra + cost optimization + live operations at Telgoo5 | ✅ Big plus |
| CI/CD | GitHub Actions, Terraform, ECS Fargate pipeline | ✅ Big plus |
| Automated testing | JUnit, Postman | 🟡 Prepare: be ready to write a unit test (`06_Testing_CICD_Cloud.md`) |
| Data pipelines / GL ingestion | Cron jobs, batch sync, data reconciliation | ✅ Very close, so connect them |
| Fintech / accounting domain | Not yet | 🔴 Prepare `07_Domain_FDD_Accounting.md` |
| openpyxl / python-docx | Python listed, but no Excel work | 🔴 Prepare `08_Data_Pipelines_Excel_AI.md`, and do the mini project below |
| AI/LLM APIs | Ms. Kalinga AI (voice news assistant) | 🟡 Be ready to explain what "AI" meant there |
| TypeScript | Listed | 🟡 Revise `01_JavaScript_TypeScript.md` section 12 |

**Summary:** Your backend + cloud + reconciliation-job experience is your strongest selling point. Your gaps are **accounting domain** and **Excel/Python automation**. Close those two gaps and you are a very strong candidate.

---

## 2. Your Best "Bridge" Points (connect your work to their product)

Say these naturally in answers. They show you are already halfway there.

| Your work | Their FDD platform |
|---|---|
| Multi-tenant data isolation (each MVNO sees only its data) | Each CPA firm / engagement must see only its own client's data |
| Data reconciliation cron jobs | Stage 1: multi-period reconciliations |
| Batch inventory sync with idempotency + overlap guards | GL ingestion that never duplicates rows when re-run |
| High-volume datasets, lakhs of requests | General ledgers with lakhs of rows |
| Third-party API integration with retry (FedEx) | QuickBooks / Xero / NetSuite / Sage Intacct API integration |
| Role-based access control | Staff / Manager / Partner roles, partner sign-off |
| Automated reporting jobs | Stage 3: report assembly |
| Plagiarism similarity scoring (Jaccard, n-gram, LCS) | Fuzzy matching of account names for account mapping! |
| Production debugging across layers | "Debug and resolve issues across front end, back end, and integrations" |

> 💡 **The plagiarism point is a hidden gem.** Mapping "Rent - Warehouse" to "Rent Expense" is a text-similarity problem, the same technique you used for plagiarism. Mention it when they ask about account mapping.

---

## 3. Telgoo5: Likely Questions and Answers

### "Explain your work at Telgoo5 simply."
> "Telgoo5 provides a telecom BSS/OSS platform to MVNOs, which are mobile companies that don't own towers. It's multi-tenant, so many MVNOs use the same system with their data kept separate. I work on inventory: SIM cards and device trade-ins. I build the REST APIs, design the MySQL tables, integrate FedEx for shipping, and write scheduled jobs for sync, reports and reconciliation."

### "What is multi-tenant? How did you isolate data?"
> "One application serves many customers (tenants). Each table has a `tenant_id` column, the tenant is identified from the logged-in user's token, and every query is filtered by `tenant_id`, applied centrally (e.g. in a base repository / Hibernate filter) so a developer can't forget it. Other options are a separate schema or a separate database per tenant: stronger isolation but more cost."
*(Say what your team actually used.)*

### "How did you optimize SQL queries?"
> "I used `EXPLAIN` to find full table scans, added indexes on columns used in WHERE and JOIN (like `tenant_id`, `status`, `created_at`), used composite indexes, avoided `SELECT *`, used pagination instead of loading everything, and fixed N+1 queries in JPA with `JOIN FETCH`."

### "Explain the FedEx integration. What if FedEx is down?"
> "We call FedEx's REST APIs to create shipments, get labels and track status. FedEx uses OAuth tokens, so we cache the token and refresh it before expiry. For failures: timeouts on every call, **retry with exponential backoff** only for temporary errors (5xx, timeouts), not for 4xx. If it still fails we mark the order as 'pending shipment', log it, and a scheduled job retries later, so the user's order is never lost."

### "Explain your cron jobs. What is idempotent? What is an overlap guard?"
> - **Cron expression:** `0 0 2 * * *` in Spring `@Scheduled(cron = ...)` = every day at 2 AM (Spring has 6 fields, starting with seconds; Linux crontab has 5).
> - **Idempotent:** running the job twice gives the same result as once. Done using unique keys, "upsert" (insert or update), and tracking the last processed record/time.
> - **Overlap guard:** if a run is still going, the next one must not start. Done with a lock: a DB flag/lock row, or a library like **ShedLock** (important when many servers run the same app, because `@Scheduled` alone runs on every server).

```java
@Scheduled(cron = "0 0 2 * * *")
@SchedulerLock(name = "inventorySync", lockAtMostFor = "PT1H")
public void syncInventory() { ... }
```

### "What did data reconciliation mean in your job?"
> "Comparing two sources that should match, for example our inventory records vs the warehouse/partner records, finding mismatches (missing, extra, different quantities), and reporting or fixing them. Accountants do the same thing with the general ledger vs the bank or trial balance, so I already understand the idea."

### "What is AWS Amplify and Lambda? Why use them?"
> "Amplify hosts and deploys frontend apps directly from Git, with build, hosting and CDN included. Lambda runs small functions without managing servers, and you pay only when they run. Good for event-driven tasks like processing a file when it's uploaded to S3."

### "Tell me about a production issue you debugged."
Prepare one real story (STAR). Example structure:
> "An API became slow for one tenant → checked CloudWatch logs and the slow query log → found a query without an index on a big table → added a composite index → response time went from X seconds to Y ms → added it to our checklist."

---

## 4. Rnpsoft Internship: Likely Questions

### "What was Ms. Kalinga AI?"
> "A voice-powered news assistant for 1,000+ readers. The backend in Spring Boot exposed REST APIs that served news content; the voice part used [text-to-speech / speech API, say what you actually used]. I designed the APIs, tested them in Postman, and deployed on Linux servers."

⚠️ **Be careful:** if they ask "which AI model did you use?", answer honestly about what it really was. Don't over-claim. If it used an AI/speech API, this connects to the JD's "AI/LLM APIs" bonus.

---

## 5. Core Java and Multithreading (you wrote a lot here, so expect deep questions!)

Your resume lists threads, thread pools, locks, volatile, atomic, CompletableFuture, JVM memory. **Anything written there can be asked.**

| Question | Simple answer |
|---|---|
| Thread lifecycle? | NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED |
| Runnable vs Callable? | Runnable returns nothing; Callable returns a value and can throw exceptions (used with `Future`) |
| Why a thread pool? | Creating threads is expensive; a pool reuses a fixed number of threads and queues extra tasks |
| How do you size a pool? | CPU-heavy work: about the number of CPU cores. I/O-heavy work (waiting on network/DB): more threads, roughly cores × (1 + wait time / compute time) |
| `synchronized` vs `ReentrantLock`? | Both give one-thread-at-a-time access. Lock adds tryLock, timeouts, fairness |
| `volatile`? | Every thread sees the latest value (visibility), but it's **not** atomic for `count++` |
| Atomic classes? | `AtomicInteger.incrementAndGet()`: thread-safe without locks (uses CAS) |
| Race condition? | Two threads change shared data at once → wrong result. Fix: locks, atomic, immutable data |
| Deadlock? | Two threads each wait for a lock the other holds. Avoid: always take locks in the same order, use tryLock with timeout |
| CompletableFuture? | Run async tasks and chain them: `supplyAsync(...).thenApply(...).exceptionally(...)` |
| JVM memory? | **Heap** (objects, shared, cleaned by GC: young + old generation), **Stack** (per thread, method calls + local variables), **Metaspace** (class info) |
| Garbage collection? | JVM frees objects nobody references. Modern default is G1 GC |
| HashMap internals? | Array of buckets; key's hash picks the bucket; collisions become a linked list, then a tree after 8 items; resizes at 75% full |
| HashMap vs ConcurrentHashMap? | HashMap is not thread-safe; ConcurrentHashMap is safe and fast (locks small parts) |

### CodeCampus thread pool: explain like this
```java
ExecutorService pool = new ThreadPoolExecutor(
    4, 8,                                   // core and max threads
    60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),          // bounded queue: max 100 waiting submissions
    new ThreadPoolExecutor.CallerRunsPolicy()  // when full, slow down the caller instead of crashing
);

Future<Result> f = pool.submit(() -> judge0.run(code));
Result r = f.get(10, TimeUnit.SECONDS);     // per-submission timeout
```
> "The queue is **bounded** so that during a contest, 1,000 submissions at once don't use up all memory. Each submission has a **timeout**. On shutdown I call `shutdown()` then `awaitTermination()` so running submissions finish (**graceful shutdown**). User code runs inside **Judge0 in Docker**, so it's isolated and can't harm the server."

**Link to FDD:** "The same idea works for running 20 analysis modules in parallel, or processing big GL files in chunks."

---

## 6. CodeCampus: Extra Questions From Your Resume

| Question | Answer |
|---|---|
| 4 roles: how? | Spring Security + JWT; role stored in the token; `@PreAuthorize("hasRole('TEACHER')")` on endpoints |
| What is Judge0? | Open-source code execution engine; runs code in sandboxed Docker containers with time and memory limits |
| How did tests work with 50+ test cases? | Run code for each input, compare output with expected output, give a pass/fail per case |
| Proctoring? | Browser camera access (`getUserMedia`), and tab-switch detection with the `visibilitychange` event; events are sent to the backend and logged |
| Plagiarism: Jaccard? | Similarity = (common tokens) ÷ (all unique tokens). 1 = same, 0 = totally different |
| n-gram? | Break code into chunks of n tokens and compare those chunks, so it catches reordered copying |
| LCS? | Longest Common Subsequence: the longest sequence of tokens in the same order in both codes |
| Why combine 3? | Each catches different tricks (renaming variables, reordering, adding junk lines). Combined score = fewer false results |

---

## 7. EasyShop and AWS Pipeline: Likely Questions

### EasyShop (Next.js + MongoDB + Redux)
| Question | Answer |
|---|---|
| SSR vs SSG? | SSR builds the page on every request (fresh data). SSG builds at build time (fastest, for static pages like product listings) |
| Why Redux? | One central store for shared state like cart and user; predictable updates with actions and reducers. For small apps, Context or Zustand is enough |
| MongoDB vs MySQL, which for money? | MySQL/Postgres for financial data (ACID, joins, exact decimals). MongoDB is good for flexible product catalogs |
| Rolling update / zero downtime? | New version starts gradually while old one still serves; traffic moves only to healthy new instances |

### AWS CI/CD pipeline project (this is very impressive, so explain it confidently)
> "On every PR, GitHub Actions builds and tests. On merge, it logs into AWS with **OIDC**, so no long-lived AWS keys are stored in GitHub. It builds a Docker image, tags it with the **git commit SHA** (so every version is traceable), pushes to **ECR**, and deploys to **ECS Fargate** behind an **Application Load Balancer**. A **circuit breaker** automatically rolls back if the new version fails health checks. All infrastructure is written in **Terraform**, so it's repeatable. Releases to production need an approval step."

| Question | Answer |
|---|---|
| What is OIDC here? | GitHub proves its identity to AWS and gets short-lived credentials. Safer than storing access keys |
| Fargate vs EC2? | Fargate = run containers without managing servers. EC2 = you manage the VMs |
| What is Terraform? | Infrastructure as Code: write cloud resources in files (`.tf`), `terraform plan` shows changes, `terraform apply` creates them |
| Why tag with git SHA? | Exactly know which code is running; easy rollback to any version |

---

## 8. Honest Gaps: How to Answer

**"Do you have accounting/finance experience?"**
> "Not professionally yet, but my reconciliation and reporting jobs at Telgoo5 are similar in spirit. To prepare, I learned the basics: general ledger, chart of accounts, trial balance, debit equals credit, EBITDA and quality of earnings. I understand that your platform imports GLs, maps accounts and runs consistency checks, and I'm keen to learn the domain properly."

**"Have you used openpyxl / python-docx?"**
> "I've used Python, and I've practised openpyxl to read a general ledger from Excel and write a trial balance, and python-docx to generate a simple report. I can pick them up quickly." *(Only say this after you actually do the mini project below!)*

**"Your skills list is very long. What are you strongest in?"**
> "Java and Spring Boot backend with MySQL. That's what I use daily at work. Next is React. The others I've used in projects and can work with, but backend is my core."

⚠️ **Rule:** for every skill on your resume, be ready for at least 2 basic questions. If you can't answer basics about something (e.g. PHP, Kubernetes, stored procedures), consider removing it from your resume before applying.

---

## 9. Mini Project to Close the Gaps (2 to 3 days, highly recommended)

**"GL Analyzer"**: put it on GitHub and add it to your resume as your 4th project.

- **Backend:** Spring Boot (your strength) **or** FastAPI (Python, closer to their stack)
- **Steps:**
  1. Upload a GL CSV/Excel (create a sample with ~200 rows: date, account, description, debit, credit).
  2. Parse it with an **adapter** (try 2 formats: "QuickBooks-style" and "Xero-style" columns).
  3. Save to MySQL/Postgres with `DECIMAL(18,2)`.
  4. **Account mapping** with keyword rules + text similarity (reuse your Jaccard code!).
  5. Show **trial balance** + **monthly P&L** in React.
  6. **Consistency checks:** debit = credit, no unmapped accounts, no duplicates.
  7. **Export** the report to Excel with **openpyxl** and to Word with **python-docx**.
  8. Optional: "Suggest mapping" button using the Claude/OpenAI API.

**Resume line:**
> "GL Analyzer: Built a general ledger ingestion and analysis tool with source-specific parsers, rule- and similarity-based account mapping, trial balance / P&L views, automated consistency checks, and Excel/Word report export (openpyxl, python-docx)."

This one project shows them **exactly the work in their JD**. It can be the reason you get shortlisted.

---

## 10. Resume Tweaks for This Application

1. **Professional summary:** add one line, e.g. *"Interested in data-intensive and financial applications; experienced in data reconciliation and batch processing."*
1. **Add your cloud work to the Telgoo5 section.** It isn't on your resume right now, so the interviewer won't know unless you say it. Example bullet (put in your real number):
   > *"Managed AWS cloud infrastructure and live production operations (deployments, CloudWatch monitoring, incident response); drove cost optimization through right-sizing, scheduling non-prod resources, and log/storage lifecycle policies, reducing monthly spend by ~[X]%."*
2. **Move "Python" and "TypeScript" earlier** in the languages list (JD mentions both).
3. In the Telgoo5 reconciliation bullet, keep the words **"data reconciliation"**, **"batch processing"** and **"high-volume datasets"**. These are keywords for this JD.
4. Add the **GL Analyzer** project once done.
5. Add **"Automated testing (JUnit, Mockito)"** if you've used Mockito, because the JD asks for unit/integration testing.
6. Keep it to **1 page** (it already is, good).

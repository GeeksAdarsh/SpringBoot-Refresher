# CodeCampus — How It All Works

This file explains **exactly how this project works** — what runs where, and how a button click turns into a database change and back.

> 📘 Relearning Spring Boot itself (annotations, layers, DI, JPA, Security)? That's now its own file: **`SPRINGBOOT_REFRESHER.md`**.
> 🐛 Something broken? That's now its own file too: **`DEBUGGING.md`**.

Read this one top to bottom once, then use it as a reference — jump to "Button → API Trace Library" whenever you forget how a specific feature works.

---

## 1. The Big Picture — 3 Things Running at Once

When you run this project locally (per `SETUP.md`), three separate processes talk to each other:

```
┌─────────────────────┐        ┌──────────────────────┐        ┌──────────────────┐
│   BROWSER            │        │   BACKEND             │        │   DATABASE        │
│   React app          │  HTTP  │   Spring Boot         │  SQL   │   MySQL / H2       │
│   localhost:5173     │ ─────► │   localhost:8080      │ ─────► │   localhost:3306   │
│   (Vite dev server)  │ ◄───── │   (embedded Tomcat)   │ ◄───── │                    │
└─────────────────────┘  JSON   └──────────────────────┘  rows  └──────────────────┘
```

- **Frontend** (`client/`): React 19 + Vite. This is just static JS/HTML/CSS served to your browser. It has **no direct database access** — it can only talk over HTTP.
- **Backend** (`server/`): Spring Boot. A Java program that exposes REST endpoints (`/api/...`), enforces security/business rules, and is the *only* thing allowed to touch the database.
- **Database**: MySQL in production, or the bundled H2 file (`server/data/codecampus.mv.db`) for zero-setup local dev.

**Why the frontend can call `/api/...` without specifying `localhost:8080`:** look at `client/vite.config.js`:

```js
server: {
  port: 5173,
  proxy: {
    '/api': { target: 'http://localhost:8080', changeOrigin: true },
  },
},
```

Vite's dev server silently forwards any request starting with `/api` to the Spring Boot server on port 8080. So in the browser it *looks* like the frontend has its own API, but it's really just a proxy — this avoids CORS headaches during development.

---

## 2. The Full Request Lifecycle (generic template)

Every single button click in this app follows this exact 10-step path:

```
 1. User clicks a button in a React page (e.g. onClick={handleLogin})
 2. React calls a function from services/api.js (e.g. authAPI.login(form))
 3. Axios sends an HTTP request to /api/... with the JWT in the header (if logged in)
 4. Vite proxy forwards it to http://localhost:8080 (dev) — or nginx does this in production
 5. Spring Security's filter chain intercepts it FIRST:
      a. JwtAuthenticationFilter validates the token, sets the authenticated user
      b. @PreAuthorize / URL rules in SecurityConfig check the role is allowed
 6. Spring routes the URL + HTTP verb to the matching @RestController method
 7. The Controller calls a @Service method, passing DTOs / the authenticated User
 8. The Service applies business logic, calling one or more Repository methods
 9. The Repository (Spring Data JPA) runs the actual SQL against MySQL/H2 via Hibernate
10. The result flows back UP: Repository → Service → Controller → JSON response
      → Axios resolves the promise → React updates state → UI re-renders
```

Everything below is this same loop, just with different endpoints and business logic.

---

## 3. Button → API Trace Library

Concrete, file-by-file traces for the most important actions in the app.

### 3.1 🔐 Login button (`Login.jsx`)

| Step | File | What happens |
|---|---|---|
| Click | `client/src/pages/Login.jsx` | `handleSubmit()` calls `login(form)` from `AuthContext` |
| Context | `client/src/context/AuthContext.jsx` | Calls `authAPI.login(data)` |
| HTTP call | `client/src/services/api.js` | `POST /api/auth/login` with `{ username, password }` |
| Security | `SecurityConfig.java` | `/api/auth/**` is `permitAll()` — no token needed to reach this endpoint |
| Controller | `AuthController.java` → `login()` | Delegates straight to `AuthService.login(request)` |
| Service | `AuthService.java` | Uses Spring's `AuthenticationManager` to verify the password (BCrypt compare) against the `UserRepository` record, then calls `JwtService.generateToken(user)` |
| Response | `AuthResponse` DTO | `{ token, id, username, fullName, role, ... }` sent back as JSON |
| Frontend | `AuthContext.jsx` | Saves `token` + `user` to `localStorage`, sets React state |
| UI | `Login.jsx` | Shows a success toast, then `navigate()`s to the dashboard matching `userData.role` (Admin/Teacher/Company/Student) |

### 3.2 📝 Register button (`Register.jsx`)

`POST /api/auth/register` (public) → `AuthController.register()` → `AuthService.register()`, which:
- Validates the email domain if `college.email.domain` is configured
- Hashes the password with `PasswordEncoder` (BCrypt)
- Looks up the selected `Batch`/`Section` via their repositories
- Saves a new `User` entity with `role = STUDENT/TEACHER/COMPANY`
- Immediately logs them in by generating and returning a JWT (same `AuthResponse` shape as login)

### 3.3 ▶️ "Run" button on the code editor (`ProblemDetail.jsx` / `Compiler.jsx`)

This is a **Run**, not a **Submit** — it only tests against the *sample* test cases and never saves a graded submission.

| Step | File | Detail |
|---|---|---|
| Click | `ProblemDetail.jsx` → `handleRun()` | Loops over each sample test case, calling `codeAPI.execute(...)` once per case |
| HTTP | `api.js` | `POST /api/code/execute` with `{ code, language, input }` |
| Security | requires `authenticated()` (any logged-in role) |
| Controller | `CodeController.java` → `executeCode()` | Calls `CodeExecutionService.executeCode(request, user)` |
| Service | `CodeExecutionService.java` | 1. Acquires a `Semaphore` permit (caps concurrent executions so an exam spike doesn't overload the server) <br> 2. Creates a temp directory <br> 3. Writes the code to a file (`Main.java`/`Main.py`/etc.) <br> 4. If Judge0 is reachable, delegates execution there; otherwise compiles+runs locally via `ProcessBuilder` (`gcc`, `javac`, `python`, `node`...) <br> 5. Captures stdout/stderr with a 10-second timeout <br> 6. Deletes the temp directory |
| Response | `CodeExecutionResponse` DTO | `{ output, errorOutput, executionTimeMs }` — no DB write happens for a Run |
| UI | `ProblemDetail.jsx` | Compares `response.data.output` against the expected output client-side and shows PASSED/FAILED per case |

### 3.4 ✅ "Submit" button on the code editor (`ProblemDetail.jsx`)

This is the **graded** action — it runs against ALL test cases (including hidden ones) and permanently records a `Submission`.

`POST /api/code/submit` → `CodeController` → `CodeExecutionService.executeCode()` (same sandbox as Run, but against every `TestCase` for that question, including hidden ones) →
- Computes `testCasesPassed` / `totalTestCases` and a `score`
- Sets `status` (`ACCEPTED`, `WRONG_ANSWER`, `RUNTIME_ERROR`, `TIME_LIMIT_EXCEEDED`, etc.)
- Saves a new row via `SubmissionRepository.save(...)`
- If this run is part of a test (`testId` present), the score also feeds into that test's leaderboard next time it's computed
- `PatternBasedCodeReviewService` adds rule-based improvement hints to the response (e.g. "consider avoiding nested loops")
- Result flows back to `ProblemDetail.jsx`, which shows the verdict and switches to the "Result" tab

### 3.5 🏁 Finishing a test — "Submit Test" button (`TakeTest.jsx` / `ProblemDetail.jsx` in test mode)

`POST /api/tests/{id}/submit` → `TestController.submit()` → `CodingTestService`:
- Marks a `TestCompletion` record for this student+test (so they can't retake it)
- Aggregates all their `Submission`s and `McqAnswer`s for that test into `totalScore`
- Locks the student out of further submissions for that test (checked by `GlobalExceptionHandler`'s "already submitted" → HTTP 409 handling)
- Frontend navigates back to `/contests` and shows a success toast

### 3.6 🛠️ Teacher "Publish Test" button (`CreateTest.jsx` → `TeacherDashboard.jsx`)

| Step | Detail |
|---|---|
| Build | `CreateTest.jsx` collects: questions, MCQs, section restrictions, excluded students, proctoring flags, timing, into one `CodingTestDTO` |
| Save | `POST /api/tests` (create) or `PUT /api/tests/{id}` (edit) — role-gated to `TEACHER`/`ADMIN`/`COMPANY` via `@PreAuthorize` |
| Service | `CodingTestService.createTest()` / `updateTest()` saves the `CodingTest` entity, links `Question`s via the `test_questions` join table, saves `McqQuestion`s, and saves the section-restriction list |
| Publish toggle | `TeacherDashboard.jsx` → `handlePublishTest()` calls a dedicated endpoint that flips `isPublished = true`, which is what makes the test appear for students in `Contests.jsx` |
| Notify | `CodingTestService` calls `NotificationService` to create an in-app notification for every eligible student, which then shows up via `NotificationBell.jsx` polling `/api/notifications/unread-count` |

### 3.7 🎥 Automatic proctoring events (no button — triggered by browser events)

`ProctoringGuard.jsx` (wraps `TakeTest.jsx`/`ProblemDetail.jsx` in test mode) listens for browser events and **automatically** fires API calls:

| Browser event | What fires | Endpoint |
|---|---|---|
| `visibilitychange` (tab switch) | Increments a local counter, sends event | `POST /api/proctoring/log` with `eventType: TAB_SWITCH` |
| `fullscreenchange` (exits fullscreen) | Sends event, shows a warning modal | `POST /api/proctoring/log` with `eventType: FULLSCREEN_EXIT` |
| Timer interval | Captures a `<canvas>` snapshot from the webcam feed | `POST /api/proctoring/log` with `eventType: SNAPSHOT` + base64 image |

Backend: `ProctoringController.logEvent()` → `ProctoringService.logEvent()`:
- Saves a `ProctoringLog` row
- If it's a tab switch, increments `tabSwitchCount` and checks against `test.getMaxTabSwitches()` (default 7, set in `application.properties`)
- Returns `{ logged: true, flagged: bool, autoSubmit: bool }` — if `autoSubmit` is `true`, the frontend immediately calls the test-submit endpoint on the student's behalf (see 3.5), ending the test forcibly.

### 3.8 🔍 "Run Plagiarism Check" button (`CompanyDashboard.jsx` / `PlagiarismReport.jsx`)

`POST /api/plagiarism/check/{testId}` (Teacher/Admin/Company only) → `PlagiarismController.runCheck()` → `PlagiarismService.runPlagiarismCheck()`:
1. Fetches all `Submission`s for that test, grouped by question
2. For every pair of submissions to the same question, computes: `0.4 × Jaccard + 0.3 × N-gram + 0.3 × LCS` similarity (see `SYSTEM_DESIGN.md` §2.4 for the algorithm math)
3. Any pair scoring ≥ `plagiarism.similarity-threshold` (default 70, in `application.properties`) gets saved as a `PlagiarismReport` with `status = DETECTED`
4. Response is a list of flagged pairs, rendered by `PlagiarismReport.jsx` with usernames + similarity %

### 3.9 📋 MCQ answer submission (inside `TakeTest.jsx`)

Each time a student picks an option: `POST /api/mcq/answer` → `McqController` saves/updates an `McqAnswer` row (`user_id`, `mcq_question_id`, `selected_option`, `is_correct` computed server-side by comparing against `correct_option` — never trust the client with the answer key). Teachers fetching the same test's MCQs (`GET /api/mcq/tests/{id}/questions`) get the `correct_option` field stripped from the JSON unless they're staff.

### 3.10 🔑 Forgot / Reset Password

```
ForgotPassword.jsx → POST /api/auth/forgot-password → AuthService
   → generates a random UUID token, saves a PasswordResetToken (expires in 30 min)
   → EmailService sends a reset link: {app.frontend.url}/reset-password?token=...
   → (if SMTP isn't configured, the link is just logged to the console — dev mode)

ResetPassword.jsx (reads ?token= from URL) → POST /api/auth/reset-password
   → AuthService validates the token isn't expired/used, updates the User's password (re-hashed with BCrypt)
```

---

## 4. Configuration Reference (`server/src/main/resources/application.properties`)

This file is Spring Boot's central settings file. Every `${VAR:default}` means: *"read environment variable `VAR`; if it's not set, use `default`."* This is how the same code runs unmodified on your laptop (using the defaults) and in production (where you set real env vars).

| Section | Key properties | Purpose |
|---|---|---|
| Server | `server.port=${PORT:8080}` | Which port Tomcat listens on |
| Database | `spring.datasource.url/username/password` | MySQL connection string; `createDatabaseIfNotExist=true` means it even creates the DB itself |
| Connection pool | `hikari.maximum-pool-size=50` | How many simultaneous DB connections are allowed (HikariCP is Spring Boot's default pool) |
| JPA | `ddl-auto=update` | Auto-sync entity classes → DB schema (safe for dev; real production apps usually use migrations instead) |
| JWT | `jwt.secret`, `jwt.expiration=86400000` | Signing key + token lifetime in ms (24 hours) |
| CORS | `cors.allowed-origins` | Which frontend origins are allowed to call the API |
| Mail | `spring.mail.*` | SMTP settings for sending password-reset emails |
| Judge0 | `judge0.api.url` | If reachable, code execution is sandboxed via Docker; otherwise falls back to local compilers |
| Execution limits | `code.execution.timeout=10`, `max-concurrent=12` | Prevents runaway/malicious code from hanging the server, and throttles concurrent runs |
| Proctoring | `max-tab-switches=7`, `snapshot-interval-seconds=300` | Default exam integrity thresholds |
| Plagiarism | `similarity-threshold=70` | % similarity above which a pair is flagged |

---

## 5. Dependencies Reference (`server/pom.xml`)

Maven's `pom.xml` lists every library the backend needs. Spring Boot "starters" are bundles that pull in everything for one concern:

| Dependency | What it gives you |
|---|---|
| `spring-boot-starter-web` | Embedded Tomcat + Spring MVC (the `@RestController` machinery) |
| `spring-boot-starter-data-jpa` | Hibernate + Spring Data JPA (the `@Entity`/`JpaRepository` machinery) |
| `spring-boot-starter-security` | The whole authentication/authorization filter chain |
| `spring-boot-starter-validation` | Powers `@Valid`, `@NotBlank`, `@Email` on DTOs |
| `spring-boot-starter-mail` | `JavaMailSender` used by `EmailService` |
| `mysql-connector-j` | JDBC driver so Java can talk to MySQL |
| `h2` | Embedded file-based DB for zero-setup local dev |
| `jjwt-api` / `jjwt-impl` / `jjwt-jackson` | JWT creation/parsing library used by `JwtService` |

---

## 6. Where to Look Next

- **`SPRINGBOOT_REFRESHER.md`** — relearn the actual Spring Boot concepts (annotations, DI, JPA, Security filter chain) behind everything traced in this file.
- **`DEBUGGING.md`** — what to do when any of these flows breaks (status code meanings, common exceptions, symptom → file map).
- **`FOLDER_STRUCTURE.md`** — what every single file/class is for, in isolation.
- **`SYSTEM_DESIGN.md`** — architecture diagrams, ER diagram, RBAC matrix, plagiarism algorithm math.
- **`SETUP.md`** — exact commands to run this locally, demo login accounts, Judge0 Docker setup.
- **`DEPLOYMENT.md`** — how this gets deployed to AWS via Terraform.

If you get stuck relearning a specific Spring concept, the fastest way back in is to **pick one feature (e.g. Feedback — it's the smallest), and read its 3 files top to bottom**: `FeedbackController.java` → `FeedbackRepository.java` → `Feedback.java` (there's no separate `FeedbackService` — the controller talks to the repository directly, which is the simplest possible version of this pattern). Once that "clicks", the bigger services (`CodingTestService`, `CodeExecutionService`) are just the same pattern with more steps.

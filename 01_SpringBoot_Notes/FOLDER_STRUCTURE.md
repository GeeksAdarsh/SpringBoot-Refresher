# CodeCampus — Repository Folder Structure

## `backups/`
Database backup dumps
- `codecampus_full_backup_2026-03-26_222447.sql`
- `codecampus_full_backup_2026-03-26_224732.sql`
- `RESTORE_INSTRUCTIONS.md`

## `client/` — React Frontend
Root config: `Dockerfile`, `.dockerignore`, `.gitignore`, `eslint.config.js`, `index.html`, `nginx.conf`, `package.json`, `package-lock.json`, `postcss.config.js`, `README.md`, `tailwind.config.js`, `vite.config.js`

### `client/public/`
- `favicon.svg`
- `icons.svg`

### `client/src/`

**`App.jsx`**
Defines every route in the app (`react-router-dom`), wraps them in `AuthProvider`/`ThemeProvider`, and wires each protected page behind `ProtectedRoute` with the correct allowed roles. Also contains `SmartHome`, which redirects a logged-in user to their role's dashboard instead of the public landing page.

**`main.jsx`**
The Vite/React entry point — mounts `<App />` into the DOM's root element. Standard bootstrap file, rarely touched.

**`index.css`**
Global stylesheet: Tailwind directives, CSS theme variables for dark/light mode, and shared utility classes like `.glass-card`, `.btn-primary`, and `.animated-bg` used across every page.

### `client/src/assets/`
- `hero.png` — Landing page hero image
- `react.svg`, `vite.svg` — default Vite/React starter icons

### `client/src/components/`

**`Navbar.jsx`**
Top navigation bar shown on every page — shows role-aware links (Admin/Teacher/Student/Company dashboards), a dark/light theme toggle, the notification bell, and logout. Reads the current user from `AuthContext` to decide what to render.

**`NotificationBell.jsx`**
Dropdown bell icon showing unread notification count and a list of recent notifications (polls `/api/notifications`). Lets users click through to the relevant test/page and marks items as read.

**`ProctoringGuard.jsx`**
Wraps test-taking pages to enforce exam integrity: requests camera access, forces fullscreen, listens for tab-switch/visibility events, periodically captures snapshots, and triggers auto-submit when limits are exceeded. This is the client-side half of the proctoring feature (paired with `ProctoringController`/`ProctoringService` on the backend).

**`ProtectedRoute.jsx`**
Route guard component — redirects to `/login` if no user is authenticated, or to `/` if the user's role isn't in the allowed `roles` list for that route. Shows a loading spinner while auth state is still resolving.

### `client/src/context/`

**`AuthContext.jsx`**
React Context providing `user`, `login()`, `register()`, `logout()`, and `updateUser()` to the whole app. Persists the JWT + user object in `localStorage` and checks token expiry on load so stale sessions are cleared automatically.

**`ThemeContext.jsx`**
Small Context managing dark/light theme state, persisting the choice to `localStorage` and toggling a CSS class on `<html>` so Tailwind's dark-mode styles apply globally.

### `client/src/services/`

**`api.js`**
Single Axios instance used by the entire frontend — attaches the JWT `Authorization` header to every outgoing request, auto-logs-out the user on a `401` response, and exports grouped API helper objects (`authAPI`, `testAPI`, `questionAPI`, `mcqAPI`, `proctoringAPI`, `plagiarismAPI`, `companyAPI`, `batchAPI`, `profileAPI`, `notificationAPI`, `feedbackAPI`, `codeAPI`, `adminAPI`) so pages never hand-write fetch URLs.

### `client/src/pages/` (24 pages)

**`AdminDashboard.jsx`**
Admin's control center showing platform-wide stats, the full user list (grouped/searchable), and all tests. Lets the admin add teachers, enable/disable accounts, and review/resolve user-submitted feedback. It's the single place an Admin manages people and monitors the whole system.

**`BatchManagement.jsx`**
Manages the Batch → Section → Student hierarchy (e.g. "2026 B.Tech" batch with sections A/B). Supports creating batches/sections, adding students manually, bulk CSV import, and debarring/reinstating students. Used by Admin and Teacher roles to organize the student roster.

**`CandidateResults.jsx`**
Shows a company's hiring-test results as a ranked candidate list (pass/fail, scores) for a specific test. Lets a Company user review applicant performance and export results. Reached from the Company dashboard per test.

**`CompanyDashboard.jsx`**
Landing page for Company-role users, listing their hiring tests with quick stats and actions to view candidates, run plagiarism checks, or view proctoring reports. Acts as the hub for recruitment-oriented test management.

**`Compiler.jsx`**
A standalone "just run code" playground (no test/question context) using the Monaco editor. Lets any user pick a language, write code, provide stdin, and execute it via `/api/code/execute` to see output/errors. Useful for quick experimentation outside of problems/tests.

**`Contests.jsx`**
Lists all coding contests/tests visible to the logged-in student (with live countdown timers for upcoming/ongoing tests). Filters tests by section access and join windows, and links into `TestDetail`/`TakeTest`. This is the student's "browse assessments" page.

**`CreateQuestion.jsx`**
Form for Teachers/Admins to author or edit a coding problem — title, description, constraints, difficulty, tags, and multiple test cases (sample + hidden). Submits to the question API and reused for both create and edit flows via a route param.

**`CreateTest.jsx`**
The most complex authoring page: build a coding+MCQ test, attach questions, restrict visibility to specific batches/sections, exclude individual students, configure proctoring (camera/fullscreen/tab-switch limits), and set timing/marks. Used by Teachers, Admins, and Companies to publish assessments.

**`ForgotPassword.jsx`**
Simple form to request a password-reset email by entering an account email address. Always shows a generic "check your inbox" success message (to avoid leaking which emails are registered) and links back to Login.

**`Landing.jsx`**
Public marketing/home page shown to logged-out visitors, describing the platform ("Code. Compete. Conquer.") with CTAs to register or log in. First page most users see before authenticating.

**`LiveExamDashboard.jsx`**
Real-time monitoring view for an ongoing test — auto-refreshing leaderboard, active-student tracking, and a way for teachers/admins to reset a student's auto-submitted attempt so they can rejoin. Used during live proctored exams to watch progress as it happens.

**`Login.jsx`**
Username/password sign-in form that authenticates via `AuthContext` and redirects to the correct role-specific dashboard (Admin/Teacher/Company/Student) on success. The main entry point into the authenticated app.

**`PlagiarismReport.jsx`**
Displays plagiarism-detection results for a test's submissions — similarity scores, flagged pairs, and detailed comparisons. Lets Teachers/Admins/Companies trigger a fresh scan and review suspected cheating cases.

**`ProblemDetail.jsx`**
The core coding workspace: Monaco code editor, language picker (with "simple" vs "competitive programming" templates), run/submit against test cases, submission history, and hints. This is where students actually solve individual problems (largest page in the app, ~1400 lines).

**`Problems.jsx`**
Browsable, searchable, paginated catalog of all coding problems with category/difficulty/tag filters and solved-status tracking per student. Teachers/Admins get inline edit access. Acts as the entry point into `ProblemDetail`.

**`ProctoringReport.jsx`**
Summarizes proctoring data per student for a test — trust/integrity score, camera snapshots, tab-switch counts, and flagged events. Used by Teachers/Admins/Companies to audit whether a student's attempt was honest.

**`Profile.jsx`**
User's public/own profile page — bio, social links, university/department info, streak, and recent activity/study plans. Supports inline editing of profile fields and changing one's username (with live availability checking).

**`Register.jsx`**
Account-creation form supporting all roles (Student/Teacher/Company), dynamically showing batch/section pickers for students or company-specific fields for company sign-ups. Calls the register API then routes to the appropriate dashboard.

**`ResetPassword.jsx`**
Consumes a reset token from the URL (sent via email) and lets the user set a new password with confirmation and minimum-length validation. Final step of the forgot-password flow, redirects to Login on success.

**`StudentDashboard.jsx`**
Student's home page after login — quick stats, currently active tests, and recent/recommended problems to solve. The main hub students land on, linking out to Problems, Contests, and Profile.

**`StudyPlanDetail.jsx`**
Shows one curated study plan's questions grouped/collapsible by topic, tracking which problems the student has completed. Lets students follow a structured learning path (e.g. "DSA in 30 days") rather than browsing problems freely.

**`StudyPlans.jsx`**
Lists all available study plans (curated problem sequences) as cards, linking into `StudyPlanDetail` for each. Simple catalog/listing page.

**`TakeTest.jsx`**
The proctored test-taking flow for coding+MCQ assessments — pre-test countdown, question navigation, timed submission, and integration with `ProctoringGuard` for camera/fullscreen/tab monitoring. This is the actual "exam mode" experience students go through during a live test.

**`TeacherDashboard.jsx`**
Teacher's home page listing their created questions and tests, with search/filter/pagination and quick actions like publishing a test. Entry point for Teachers into `CreateQuestion`/`CreateTest` and result-viewing pages.

**`TestDetail.jsx`**
Central hub for a single test showing tabs for instructions, live leaderboard, and (for staff) per-student results/completion status; also shows a student's own results after submitting. Branches into "Take Test", proctoring reports, and plagiarism reports depending on role.

## `deploy/`
- `docker-compose.yml`

## `docs/`
- `CodeCampus_Presentation.pptx`
- `generate_deck.py`
- `HLD.md` (High-Level Design)
- `LLD.md` (Low-Level Design)

## `ppt_assets/`
- `devops_future.png`
- `hld_layers.png`
- `lld_data_model.png`

## `server/` — Spring Boot Backend
Root config: `.dockerignore`, `Dockerfile`, `mvnw.cmd`, `pom.xml`

### `server/.mvn/`
- `jvm.config`
- `wrapper/maven-wrapper.properties`

### `server/data/`
- `codecampus.mv.db` (H2 dev database file)

### `server/src/main/java/com/codecampus/`
- `CodeCampusApplication.java` (main entry point)

### `.../config/`

**`DataInitializer.java`**
Runs on application startup (`CommandLineRunner`) — seeds demo accounts (admin/teacher/student/company), triggers `ProblemSeeder` and `BatchSectionSeeder` if the DB is empty, and creates a few sample coding tests so the app isn't blank on first run.

**`SecurityConfig.java`**
Central Spring Security configuration: disables CSRF (stateless JWT API), configures permissive CORS, defines the public vs. role-gated URL rules (e.g. `/api/auth/**` public, `/api/admin/**` ADMIN-only), sets session policy to `STATELESS`, and wires the JWT filter + BCrypt password encoder.

### `.../controller/` (15 REST controllers)

**`AdminController.java`**
`/api/admin/**`, ADMIN-only. Dashboard stats, full user management (list/add teacher/enable-disable), and platform-wide test listing.

**`AuthController.java`**
`/api/auth/**`, public. Handles `register` and `login`, delegating to `AuthService` and returning friendly error messages on failure.

**`BatchSectionController.java`**
`/api/batch-section/**`. Manages the Batch→Section→Student hierarchy: list batches/sections (public, for registration dropdowns), CSV student import, debar/reinstate students, and CRUD for batches/sections (Admin/Teacher).

**`CodeController.java`**
`/api/code/**`. Executes submitted code against test cases via `CodeExecutionService` and exposes a student's past submission history for a question.

**`CompanyController.java`**
`/api/company/**`, COMPANY/ADMIN. Dashboard stats and hiring-test listing scoped to tests created by that company.

**`FeedbackController.java`**
`/api/feedback/**`. Lets users submit feedback/bug reports tied to a question, and lets Admins list/filter/resolve them.

**`GlobalExceptionHandler.java`**
`@RestControllerAdvice` that catches uncaught `RuntimeException`s app-wide and maps them to sensible HTTP status codes (409 for "already submitted", 404 for "not found", 403 for "not authorized") instead of a generic 500.

**`McqController.java`**
`/api/mcq/**`. Teachers save MCQ question sets for a test; students fetch questions (answers hidden) and submit answers; supports resetting answers for practice mode.

**`NotificationController.java`**
`/api/notifications/**`. Lists a user's notifications and unread count, backing the `NotificationBell` frontend component.

**`PlagiarismController.java`**
`/api/plagiarism/**`, TEACHER/ADMIN/COMPANY. Triggers a plagiarism scan for a test, fetches existing reports, and lets staff manually compare two code snippets.

**`ProctoringController.java`**
`/api/proctoring/**`. Receives proctoring events (tab switch, fullscreen exit, camera snapshot) from `ProctoringGuard` during a live test, and serves per-student/per-test proctoring reports to staff.

**`ProfileController.java`**
`/api/profile/**`. Fetches/updates a user's public profile (bio, socials, avatar), handles username changes with availability checks, and computes solved-question stats/streaks.

**`QuestionController.java`**
`/api/questions/**`. CRUD for the coding-problem catalog, paginated listing, and exposes language-specific starter code via `HarnessRegistry`.

**`StudyPlanController.java`**
`/api/studyplans/**`. Serves curated study plans (loaded from `studyplans.json`, cached in memory) enriched with each student's per-question completion status.

**`TestController.java`**
`/api/tests/**`. Full lifecycle for coding tests: list (filtered per-user), active/upcoming views, get by id, create/update (Teacher/Admin/Company), submit, and leaderboard.

### `.../dto/` (10 request/response objects)

- **`AuthRequest.java`** — login payload (username/email + password)
- **`AuthResponse.java`** — login/register response (JWT token + user summary)
- **`CodeExecutionRequest.java`** — code, language, and stdin sent to `/api/code/execute`
- **`CodeExecutionResponse.java`** — stdout/stderr, pass/fail per test case, execution time
- **`CodingTestDTO.java`** — full test payload used to create/update/display a `CodingTest` (questions, sections, proctoring flags, timing)
- **`ForgotPasswordRequest.java`** — email address for a password-reset request
- **`QuestionDTO.java`** — coding problem payload (description, constraints, test cases) decoupled from the JPA entity
- **`RegisterRequest.java`** — signup payload covering all roles (student/teacher/company fields)
- **`ResetPasswordRequest.java`** — reset token + new password
- **`TestCaseDTO.java`** — a single input/expected-output pair for a question

### `.../model/` (21 JPA entities/enums)

**`User.java`**
The central entity — implements Spring Security's `UserDetails` directly. Holds credentials, role, batch/section, profile fields (bio, socials, avatar), and debarment status. Every other entity ultimately links back to a `User`.

**`CodingTest.java`**
Represents an assessment (contest/exam): timing window, duration, proctoring settings (camera/fullscreen/tab-switch limits), section restrictions, and test type (college exam vs. company hiring test).

**`Question.java`**
A coding problem: description, constraints, difficulty, tags, sample I/O, and its associated `TestCase` list. Shared by the practice catalog (`Problems.jsx`) and tests.

**`Submission.java`**
One code-run/submit attempt — stores the submitted code, language, verdict (`SubmissionStatus`), test cases passed, execution time, and score. The core record used for grading, plagiarism, and leaderboards.

**Other entities/enums** (`Batch`, `Section`, `TestCase`, `McqQuestion`, `McqAnswer`, `ProctoringLog`, `PlagiarismReport`, `TestCompletion`, `TestQuestionWeight`, `Feedback`, `UserNotification`, `PasswordResetToken`) model the supporting data — organizational hierarchy, MCQ content/answers, proctoring event logs, plagiarism scan results, per-student test completion records, per-question mark weighting, feedback tickets, in-app notifications, and password-reset tokens, respectively. The plain enums (`Difficulty`, `Role`, `ProgrammingLanguage`, `SubmissionStatus`, `PlagiarismStatus`, `ProctoringEventType`, `NotificationType`, `TestType`) just constrain those fields to fixed value sets.

### `.../repository/` (13 Spring Data JPA repositories)
Thin interfaces extending `JpaRepository`, each adding a handful of custom finder methods (e.g. `findByUsername`, `findByCodingTestId`, `countTabSwitches`) used by the matching service/controller. One repository per major entity: `BatchRepository`, `CodingTestRepository`, `FeedbackRepository`, `McqAnswerRepository`, `McqQuestionRepository`, `PasswordResetTokenRepository`, `PlagiarismReportRepository`, `ProctoringLogRepository`, `QuestionRepository`, `SectionRepository`, `SubmissionRepository`, `TestCompletionRepository`, `TestQuestionWeightRepository`, `UserNotificationRepository`, `UserRepository`.

### `.../security/`

**`JwtService.java`**
Generates and validates JWTs — signs tokens with a Base64 secret key (HS256), embeds username/role/expiry claims, and provides helpers to extract claims and check expiry.

**`JwtAuthenticationFilter.java`**
A `OncePerRequestFilter` run on every request: reads the `Authorization: Bearer <token>` header, validates it via `JwtService`, loads the user, and populates Spring Security's context so `@PreAuthorize` role checks work downstream.

### `.../service/` (13 business logic classes)

**`AuthService.java`**
Implements registration (with email-domain validation) and login, issuing JWTs on success and generating password-reset tokens/emails.

**`CodeExecutionService.java`**
The code-execution engine — writes submitted code to a temp directory, compiles/runs it with the right local compiler (or delegates to Judge0 if configured), enforces a timeout, compares output against expected test cases, and cleans up afterward. Uses a `Semaphore` to cap concurrent executions so exam-deadline submission spikes don't overload the server.

**`CodingTestService.java`**
The largest service — full CRUD for tests, per-user visibility filtering (section restrictions, join windows, debarment), submission grading/scoring, leaderboard computation, and completion tracking.

**`PlagiarismService.java`**
Runs the Jaccard + N-gram + LCS weighted similarity algorithm across all submissions for a test's questions, flags pairs above the configured threshold, and persists `PlagiarismReport` records.

**`ProctoringService.java`**
Records proctoring events (tab switches, fullscreen exits) per student/test, tracks running tab-switch counts, flags suspicious events, and decides when a test should auto-submit due to violations.

**`QuestionService.java`**
CRUD and DTO-mapping for the question catalog; caches the full question list (`@Cacheable`) since it's read far more often than it changes, evicting the cache on any write.

**`HarnessRegistry.java`**
Powers "LeetCode-style" function-mode problems — stores per-language starter code stubs and driver programs that adapt a user's function to the classic stdin/stdout judging format, using a reusable "archetype engine" for common input/output shapes.

**`PatternBasedCodeReviewService.java`**
Generates deterministic, rule-based code-improvement hints (e.g. detecting nested loops) from the submitted code and verdict — a lightweight, no-AI substitute for real code review that's safe to show even during contests.

**`BatchSectionSeeder.java`** / **`ProblemSeeder.java`**
One-time data seeders run by `DataInitializer`: the former generates realistic demo batches/sections/students (Indian name pools) for local testing, the latter loads the 39+ bundled coding problems from `problems.json`/`striver_new_problems.json` into the database on first boot.

**`NotificationService.java`**
Creates and lists in-app notifications (e.g. "new test published") for users, tracking read/unread state — backs the `NotificationBell` component.

**`EmailService.java`**
Sends transactional emails (password reset links) via Spring's `JavaMailSender`, with a configurable "from" address.

**`UserDetailsServiceImpl.java`**
Bridges Spring Security to the app's `User` entity — looks up a user by email first, then username, for use during authentication.

### `server/src/main/resources/`
- `application.properties`
- `problems.json`
- `striver_new_problems.json`
- `studyplans.json`

## `terraform/` — AWS Infra
- `.gitignore`
- `main.tf`
- `outputs.tf`
- `terraform.tfvars.example`
- `userdata.sh.tftpl`
- `variables.tf`

## Root-level files
- `.gitignore`
- `codecampus_backup.sql`
- `CodeCampus_PPT.pptx`
- `CodeCampus_Presentation_2026.pptx`
- `create_ppt.py`
- `docker-compose.yml`
- `DEPLOYMENT.md`
- `SETUP.md`
- `SYSTEM_DESIGN.md`
- `existing_titles.txt`
- `generate_testcases.py`
- `generate_testcases2.py`
- `generate_testcases3.py`
- `generate_testcases4.py`
- `requirements-ppt.txt`

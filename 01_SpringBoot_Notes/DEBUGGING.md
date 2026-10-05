# Debugging Guide — CodeCampus

A practical, symptom-first guide for when something breaks. Organized so you can jump straight to your problem instead of reading top to bottom.

---

## 0. The General Debugging Mindset

Every bug in a full-stack app like this is a **detective trail** across up to 4 places. Always check them in this order — it's the fastest path to the root cause:

```
1. Browser Console (F12 → Console tab)     → JavaScript errors, React crashes
2. Browser Network tab (F12 → Network tab) → What request was sent? What did the server reply?
3. Backend terminal (where `mvn spring-boot:run` is running) → Java stack traces
4. Database (H2 console or MySQL client)   → Is the data actually what you expect?
```

**Rule of thumb:** if a button "does nothing" or shows a generic toast like "Failed to load data", **always open the Network tab first**. It tells you immediately whether the problem is:
- The request never left the browser (JS error — check Console)
- The request was sent but got a `4xx`/`5xx` back (backend rejected it — check that response body + backend terminal)
- The request succeeded (`200`) but the UI still looks wrong (frontend logic bug — check React state/props)

---

## 1. How to Read the Network Tab (your #1 debugging tool)

1. Open DevTools (`F12`) → **Network** tab → filter by `Fetch/XHR`
2. Click the button that's misbehaving
3. Find the request (e.g. `POST /api/tests`) and click it
4. Check three things:
   - **Status code** (top right) — see the table in §4 below for what each means here
   - **Response tab** — this is the exact JSON the backend sent back, including error messages from `GlobalExceptionHandler`
   - **Request tab** (Payload/Headers) — confirm the `Authorization: Bearer ...` header is present, and the request body has the fields you expect

This single tab answers 80% of "why isn't this working" questions in this app.

---

## 2. Backend Won't Start At All

Spring Boot fails **loudly** at startup if something's wrong — read the **last ~30 lines** of the terminal output, not the first. Scroll to the bottom; the real error is usually right before `APPLICATION FAILED TO START`.

| Error text you see | Cause | Fix |
|---|---|---|
| `Port 8080 was already in use` | Another process (often a previous run that didn't die) is holding the port | `Get-NetTCPConnection -LocalPort 8080 \| ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }` (see `SETUP.md`) |
| `Communications link failure` / `Access denied for user 'root'` | MySQL isn't running, or the password in `application.properties` doesn't match | Start MySQL, or just delete/comment the MySQL lines to fall back to H2 (see `SETUP.md` §"Switch to MySQL") |
| `NoSuchBeanDefinitionException: No qualifying bean of type '...'` | You added a new `@Service`/`@Repository` but forgot the annotation, or a constructor asks for a type Spring can't build | Check the class has `@Service`/`@Component`/`@Repository`, and that all of *its* constructor dependencies are also beans |
| `NoUniqueBeanDefinitionException` | Two beans implement the same interface and Spring doesn't know which to inject | Add `@Primary` to the preferred one, or `@Qualifier` at the injection site |
| `Circular reference` / `BeanCurrentlyInCreationException` | Bean A's constructor needs Bean B, and B's constructor needs A | Break the cycle — usually by moving shared logic to a third service, or injecting via a setter/lazy reference |
| `mvn : The term 'mvn' is not recognized` (PowerShell) | Maven isn't on PATH for this terminal session | Run the `$env:Path = "$env:USERPROFILE\.m2\maven\...\bin;$env:Path"` line from `SETUP.md` first |
| `Table 'codecampus.xxx' doesn't exist` right after startup | `ddl-auto` isn't set to `update`, or Hibernate couldn't create the table due to a bad `@Entity` mapping | Check `application.properties` has `spring.jpa.hibernate.ddl-auto=update`; check the entity for typos in `@Column`/`@JoinColumn` |

---

## 3. Frontend Won't Load / Shows Blank Page

| Symptom | Cause | Fix |
|---|---|---|
| Totally blank white page | A JS error crashed React before it could render — open Console tab | Read the red error + stack trace; it'll point to a specific `.jsx` file/line |
| "This site can't be reached" | Vite dev server isn't running, or wrong port | Run `npx vite --port 5173` in `client/`; check the terminal for the actual port it bound to |
| Page loads but every API call fails with a Network Error | Backend isn't running, or `vite.config.js` proxy target is wrong | Confirm `mvn spring-boot:run` is up on `:8080`; check `client/vite.config.js` proxy target |
| `vite : command not found` | Dependencies not installed | `npm install --legacy-peer-deps` in `client/` (see `SETUP.md` — the `--legacy-peer-deps` flag matters here due to React 19) |
| Infinite redirect loop to `/login` | Stale/expired JWT stuck in `localStorage`, `AuthContext` keeps clearing it and redirecting | Open DevTools → Application tab → Local Storage → delete `token` and `user` manually, then reload |

---

## 4. HTTP Status Code Reference (what each one means IN THIS APP)

| Status | What it means here | Where to look |
|---|---|---|
| **200 OK** | Success | If the UI still looks wrong, it's a frontend state bug, not a backend one |
| **400 Bad Request** | `@Valid` validation failed, or the controller manually returned `ResponseEntity.badRequest()` | Check the Response tab for the `{"error": "..."}` message; compare your request body against the DTO's validation annotations (`@NotBlank`, `@Email`, etc.) |
| **401 Unauthorized** | No JWT sent, or it's invalid/expired | Check the Request Headers for `Authorization: Bearer ...`. If missing, you're not logged in or `api.js`'s interceptor isn't attaching it. If present but still 401, the token expired (24h lifetime) — log in again |
| **403 Forbidden** | You ARE logged in, but your role isn't allowed for that endpoint (`@PreAuthorize` or `SecurityConfig` rule blocked you) | Check `SecurityConfig.java` and the controller's `@PreAuthorize` annotation for the required role; confirm your test account has that role |
| **404 Not Found** | Wrong URL path, OR a `.orElseThrow()` in a service threw because the ID doesn't exist | Check the exact path in `api.js` matches the `@RequestMapping`/`@GetMapping` on the controller |
| **409 Conflict** | `GlobalExceptionHandler` mapped a message containing "already submitted" to 409 | This is usually intentional — e.g. trying to submit a test twice |
| **500 Internal Server Error** | An unhandled exception occurred, caught by `GlobalExceptionHandler`'s fallback | **Go straight to the backend terminal** — the full Java stack trace with the real cause is printed there. The Response tab only shows a generic message |

---

## 5. Common Backend Exceptions & What They Mean

| Exception | Typical Cause | Fix |
|---|---|---|
| `LazyInitializationException: could not initialize proxy - no Session` | You accessed a `@ManyToOne(fetch = LAZY)` field (e.g. `submission.getUser().getFullName()`) *after* the transaction/session already closed — often while Jackson is serializing the response | Either fetch it eagerly inside the `@Transactional` service method, map it to a DTO field *before* returning, or add `@Transactional(readOnly = true)` around the read |
| `ConstraintViolationException` / `DataIntegrityViolationException` | Tried to save a row that violates a DB constraint — e.g. duplicate `username`/`email` (both `@Column(unique = true)` on `User`) | Check for an existing row with that value first, or catch the exception and return a friendly 400/409 |
| `NullPointerException` on `.orElseThrow()` chains | An ID that doesn't exist was passed in (e.g. a stale test ID from an old browser tab) | Add a clearer error message to `.orElseThrow(() -> new RuntimeException("Test not found"))` so `GlobalExceptionHandler` reports 404 instead of a raw NPE → 500 |
| `MethodArgumentNotValidException` | A `@Valid @RequestBody` DTO failed one of its Bean Validation annotations | Spring auto-converts this to 400 already — check the response body for exactly which field failed |
| `JwtException` / `ExpiredJwtException` (inside `JwtAuthenticationFilter`) | Token is malformed or past its 24h expiry | Frontend should catch the resulting 401 and force a re-login (already handled by the Axios interceptor in `api.js`) |
| `Semaphore` timeout / "Server busy" message from `CodeExecutionService` | Too many students hit Run/Submit at the same second (default cap: 12 concurrent, `code.execution.max-concurrent`) | This is a deliberate safety valve during exam spikes — raise `EXEC_MAX_CONCURRENT` env var if your machine can handle more, or just wait a moment and retry |

---

## 6. Debugging the Code Execution Sandbox Specifically

This is the most "systems-y" part of the app (`CodeExecutionService.java`) and has its own failure modes:

| Symptom | Cause | Fix |
|---|---|---|
| Every submission fails with a generic error, for every language | The compiler isn't installed/on PATH (`gcc`, `javac`, `python`, `node`) | Verify each compiler works from the same terminal Spring Boot runs in: `javac -version`, `python --version`, `node -v`, `gcc --version` |
| Works for Java but not Python (or vice versa) | Only that specific compiler is missing/misconfigured | Install the missing one; Windows may need `python` vs `python3` — check `CodeExecutionService`'s OS-specific command selection |
| Code execution takes exactly 10s then fails every time | Hit `code.execution.timeout=10` — likely an infinite loop in the submitted code, OR the process hung waiting for stdin that was never provided | Check the test case actually provides the expected input; for genuinely long-running code, raise the timeout in `application.properties` |
| "Judge0 not reachable" fallback message in logs | Docker isn't running, or `docker compose up -d` was never run | This is **not an error** — the app is designed to gracefully fall back to local compilers. Only worry about this if you specifically wanted Judge0's stronger sandboxing |
| Leftover `codecampus_*` temp folders piling up in your OS temp directory | An execution crashed before reaching the `cleanup()` step | Safe to manually delete; if it happens constantly, add logging around the `finally`/cleanup block to see why it's being skipped |

---

## 7. Debugging Proctoring / Camera Issues

| Symptom | Cause | Fix |
|---|---|---|
| Camera preview never appears | Browser denied camera permission, or test isn't `cameraRequired: true` | Check the browser's site permissions (padlock icon in the address bar); confirm the test's proctoring settings in `CreateTest.jsx` |
| Fullscreen keeps kicking the student out immediately | Some browsers block `requestFullscreen()` unless triggered directly by a user gesture (click) | Ensure the fullscreen request happens synchronously inside the button's `onClick`, not after an `await` |
| Tab-switch count increments even when the student didn't switch tabs | Alt-tabbing to a different app, or an OS notification stealing focus, also fires `visibilitychange` | This is a known limitation of browser-based proctoring — check `ProctoringGuard.jsx`'s event listener for whether debouncing/grace-period logic exists |
| Auto-submit fires unexpectedly | `maxTabSwitches` threshold reached, or `ProctoringController.logEvent()` returned `autoSubmit: true` | Check `ProctoringLog` rows for that student/test in the DB to see the exact event history that triggered it |

---

## 8. Debugging the Database Directly

**If using H2 (default, zero-setup):**
1. Start the backend
2. Go to `http://localhost:8080/h2-console`
3. JDBC URL: `jdbc:h2:mem:codecampus`, username `sa`, no password
4. Run plain SQL, e.g. `SELECT * FROM users;` or `SELECT * FROM submissions ORDER BY submitted_at DESC LIMIT 20;`

**If using MySQL:**
```powershell
mysql -u root -p codecampus
SHOW TABLES;
SELECT * FROM coding_tests WHERE is_published = true;
```

**Useful sanity-check queries when a feature "isn't working":**
```sql
-- Did the submission actually save?
SELECT id, status, score, submitted_at FROM submissions WHERE user_id = <id> ORDER BY submitted_at DESC;

-- Is the test actually visible to this student's section?
SELECT * FROM test_sections WHERE test_id = <id>;

-- Is the user debarred (which hides all tests)?
SELECT username, is_debarred FROM users WHERE username = '<username>';
```

---

## 9. Adding Your Own Debug Logging

When the symptom tables above aren't enough, add temporary logging:

**Backend (Java):** every service already has an SLF4J logger pattern you can copy:
```java
private static final Logger log = LoggerFactory.getLogger(YourClass.class);
// ...
log.info("Submitting test {} for user {}", testId, user.getUsername());
```
Output appears directly in the terminal running `mvn spring-boot:run`. Remove or lower to `log.debug(...)` before committing.

**Frontend (React):** just `console.log(...)` inside the handler — it shows in the browser's Console tab:
```js
const handleSubmit = async () => {
  console.log('Submitting with payload:', payload);
  const res = await testAPI.submitTest(testId);
  console.log('Response:', res.data);
};
```

**Quick temporary breakpoint alternative** if you don't want to attach a real debugger: throw a deliberate exception with the value you want to inspect —
```java
throw new RuntimeException("DEBUG user=" + user + " test=" + test);
```
`GlobalExceptionHandler` will surface that message straight to the Network tab's Response body, no IDE debugger setup required. (Remove before committing, obviously.)

---

## 10. Quick Symptom → File Map

| "It's broken when I..." | Start reading here |
|---|---|
| ...try to log in | `AuthController.java` → `AuthService.java` → `JwtService.java` |
| ...click Run/Submit on a problem | `CodeExecutionService.java` (check backend terminal for compiler errors) |
| ...create/publish a test | `CreateTest.jsx` → `TestController.java` → `CodingTestService.java` |
| ...take a proctored test | `ProctoringGuard.jsx` → `ProctoringController.java` → `ProctoringService.java` |
| ...see a test that shouldn't be visible (or don't see one that should) | `CodingTestService.getAllTestsForUser()` — section/join-window/debarment filtering logic |
| ...run a plagiarism check | `PlagiarismService.java` (check the actual similarity math against `SYSTEM_DESIGN.md` §2.4) |
| ...get logged out randomly | `JwtService.java` (expiry) + `api.js`'s 401 interceptor |
| ...see stale data after an edit | Check `QuestionService`'s `@Cacheable("allQuestions")` — the cache might not have been evicted; confirm `@CacheEvict` is on the corresponding write method |

---

## Where to Go Next

- **`SPRINGBOOT_REFRESHER.md`** — the concepts behind *why* these errors happen (beans, DI, JPA, filters)
- **`PROJECT_FLOW.md`** — the happy-path flow for each feature, useful to compare against when something deviates
- **`FOLDER_STRUCTURE.md`** — find any file's purpose fast

# Backend and REST APIs (Easy Notes)

The JD says: *"Develop and support backend APIs and services"* and *"back-end basics (Node.js/Express, FastAPI, or equivalent, REST APIs)"*.

You know **Spring Boot** (Java), which counts as "equivalent". Also learn a little **Node/Express** and **FastAPI** so you can say "I can pick them up fast".

> Deep Spring Boot notes are in `../01_SpringBoot_Notes/SPRINGBOOT_REFRESHER.md`.

---

## 1. What is a REST API?

A way for the frontend (or another app) to talk to the backend using **HTTP** and **URLs**.
- Each **resource** (a thing) has a URL: `/api/engagements`, `/api/accounts/5`
- Each **action** uses an HTTP method.
- Data is sent as **JSON**.
- **Stateless:** each request carries everything needed (like the JWT token). The server does not remember the previous request.

---

## 2. HTTP Methods

| Method | Meaning | Example | Idempotent? |
|---|---|---|---|
| GET | Read | `GET /api/accounts` | Yes |
| POST | Create | `POST /api/accounts` | No |
| PUT | Replace fully | `PUT /api/accounts/5` | Yes |
| PATCH | Update part | `PATCH /api/accounts/5` | Usually no |
| DELETE | Remove | `DELETE /api/accounts/5` | Yes |

**Idempotent** = calling it 1 time or 10 times gives the same final result.

---

## 3. Status Codes (must know)

| Code | Meaning | When |
|---|---|---|
| 200 | OK | Success |
| 201 | Created | After POST |
| 204 | No Content | After DELETE |
| 400 | Bad Request | Wrong input / validation failed |
| 401 | Unauthorized | Not logged in / bad token |
| 403 | Forbidden | Logged in, but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate / version clash |
| 422 | Unprocessable | Data format OK but logic wrong |
| 429 | Too Many Requests | Rate limit hit |
| 500 | Server Error | Bug on the server |
| 502/503 | Bad Gateway / Unavailable | Server down or overloaded |

**401 vs 403:** 401 = "Who are you?" 403 = "I know you, but you can't do this."

---

## 4. Good API Design (talk like this in the interview)

- Use **nouns**, not verbs: `/api/engagements` ✅, `/api/getEngagements` ❌
- Nesting: `/api/engagements/12/trial-balances`
- **Pagination:** `GET /api/ledger?page=0&size=50`
- **Filter / sort:** `?account=Rent&sort=date,desc`
- **Versioning:** `/api/v1/...`
- **Same error format every time:**
```json
{ "status": 400, "error": "Validation failed", "message": "amount must be a number", "timestamp": "..." }
```
- **Validate input** on the server (never trust the frontend).
- Use **DTOs** (don't send DB entities directly).

---

## 5. Spring Boot CRUD (your main stack)

```java
@RestController
@RequestMapping("/api/accounts")
@RequiredArgsConstructor
public class AccountController {
    private final AccountService service;

    @GetMapping
    public Page<AccountDto> list(Pageable pageable) { return service.list(pageable); }

    @GetMapping("/{id}")
    public AccountDto get(@PathVariable Long id) { return service.get(id); }

    @PostMapping
    public ResponseEntity<AccountDto> create(@Valid @RequestBody CreateAccountRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.create(req));
    }

    @PutMapping("/{id}")
    public AccountDto update(@PathVariable Long id, @Valid @RequestBody CreateAccountRequest req) {
        return service.update(id, req);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Layers:** Controller (HTTP) → Service (business logic) → Repository (database) → Entity (table).

### Global error handling
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> notFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorResponse(404, ex.getMessage()));
    }
}
```

### Transactions
`@Transactional` = all DB steps succeed together or all are rolled back.
**Example:** saving a journal entry with debit and credit lines. If one line fails, nothing should be saved. Very important for accounting data.

---

## 6. Node.js + Express (learn the basics)

```js
const express = require("express");
const app = express();
app.use(express.json());              // read JSON body

app.get("/api/accounts", async (req, res) => {
  const accounts = await db.query("SELECT * FROM accounts");
  res.json(accounts);
});

app.post("/api/accounts", async (req, res) => {
  if (!req.body.name) return res.status(400).json({ message: "name required" });
  const acc = await db.insert(req.body);
  res.status(201).json(acc);
});

// error middleware (4 args)
app.use((err, req, res, next) => res.status(500).json({ message: err.message }));

app.listen(3000);
```

- **Middleware** = a function that runs between the request and the response (logging, auth, parsing). Same idea as Spring's filters.
- **Node is single-threaded + non-blocking**: good for many I/O calls, bad for heavy CPU work.

---

## 7. FastAPI (Python): just know it exists

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Account(BaseModel):
    name: str
    amount: float

@app.get("/api/accounts/{id}")
def get_account(id: int):
    return {"id": id}

@app.post("/api/accounts", status_code=201)
def create(acc: Account):
    return acc
```
FastAPI validates input automatically using **Pydantic** and builds API docs at `/docs`.

---

## 8. Authentication and Security

### JWT (you used this in CodeCampus)
1. User logs in with username + password.
2. Server checks the password (stored as a **BCrypt hash**, never plain text).
3. Server returns a **JWT token** (header.payload.signature).
4. Frontend sends it in every request: `Authorization: Bearer <token>`.
5. Server checks the signature → knows who the user is.

### Session vs JWT
| Session | JWT |
|---|---|
| Server stores the login | Token holds the info, server stores nothing |
| Easy to log out | Hard to cancel before expiry (use short expiry + refresh token) |

### Roles (RBAC)
The FDD tool has roles like **Staff, Manager, Partner**. Use `@PreAuthorize("hasRole('PARTNER')")` so only partners can sign off.

### Common security risks (OWASP basics)
- **SQL Injection:** use parameterized queries / JPA, never join strings into SQL.
- **XSS:** don't put raw HTML from users into the page (avoid `dangerouslySetInnerHTML`).
- **CSRF:** matters with cookies; JWT in headers mostly avoids it.
- Never put secrets in Git. Use environment variables / a secrets manager.

---

## 9. Audit Trail (in the JD!)

The JD mentions *"an audit-trail layer that makes the partner's review faster"*.

**Audit trail = a record of who did what, when, and what changed.**

Simple design:
```
audit_log
- id
- entity_type   (e.g. "AccountMapping")
- entity_id
- action        (CREATE / UPDATE / DELETE / APPROVE)
- old_value     (JSON)
- new_value     (JSON)
- changed_by    (user id)
- changed_at    (timestamp)
```
How to build it in Spring Boot: **Spring Data JPA Auditing** (`@CreatedBy`, `@LastModifiedDate`), **Hibernate Envers** (keeps every version automatically), or write to `audit_log` inside the service.

**Rule:** Audit logs are **never edited or deleted** (append-only).

---

## 10. Multi-user Coordination (in the JD!)

Two accountants edit the same mapping at the same time. What happens?

- **Optimistic locking:** add a `version` column. On save, if the version changed, reject with **409 Conflict**. In JPA just add `@Version private Long version;`
- **Pessimistic locking:** lock the row while editing (slower, rarely needed).
- **Real-time updates:** WebSockets / Server-Sent Events to show "Rahul is editing this".

---

## 11. Background Jobs and Big Files

Uploading a 500,000-row general ledger can take a long time. Don't make the user wait on one HTTP request.

1. `POST /api/gl/upload` → save file, create a **job**, return `202 Accepted` + `jobId`.
2. A **background worker** (Spring `@Async`, a queue like RabbitMQ/SQS, or Celery in Python) processes it.
3. Frontend **polls** `GET /api/jobs/{jobId}` (or uses WebSocket) to show progress.

---

## 12. Caching, Rate Limiting, Logging

- **Cache** repeated reads (Redis, Spring `@Cacheable`).
- **Rate limit** to protect APIs (429).
- **Logging:** use log levels (`INFO`, `WARN`, `ERROR`), add a request id, never log passwords or financial secrets.

---

## 13. Quick Q&A

- **Monolith vs Microservices?** Monolith = one app (simple, good for small teams like Windy Street at start). Microservices = many small apps (scale separately, but more complex).
- **What is an idempotency key?** A unique id sent with POST so retries don't create duplicates.
- **REST vs GraphQL?** REST = many URLs, fixed responses. GraphQL = one URL, client asks for exact fields.
- **What happens when you type a URL?** DNS → find IP → TCP/TLS connection → HTTP request → server processes → response → browser renders.
- **Dependency Injection?** Spring creates objects and gives them to you, so classes are loosely coupled and easy to test.

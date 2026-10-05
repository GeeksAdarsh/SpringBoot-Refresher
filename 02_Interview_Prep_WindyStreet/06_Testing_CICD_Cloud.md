# Testing, CI/CD and Cloud (Easy Notes)

These are in the **"Good to Have"** part of the JD. Knowing the basics gives you extra points.

The JD also says: *"maintaining and improving the consistency check library and regression test suite"*. So testing matters here.

---

## 1. Types of Tests

| Type | What it tests | Tools |
|---|---|---|
| **Unit** | One function/class alone | JUnit + Mockito (Java), Jest/Vitest (JS), pytest (Python) |
| **Integration** | Several parts together (API + DB) | Spring Boot Test, Testcontainers, Supertest |
| **End-to-End (E2E)** | Full app like a real user | Cypress, Playwright |
| **Regression** | Old features still work after new changes | Your full test suite run again |
| **Component** | One React component | React Testing Library |

**Test pyramid:** many unit tests (fast) → fewer integration tests → very few E2E tests (slow).

---

## 2. Why Testing Matters for a Finance App

- A wrong number in a due diligence report can **cost a deal millions**.
- The engines are **deterministic**: the same input must always give the same output. That makes them very easy to test.
- **Regression suite** = known input GL file → expected output report. If output changes, a test fails.

---

## 3. Unit Test Examples

### Java (JUnit 5 + Mockito)
```java
@ExtendWith(MockitoExtension.class)
class TrialBalanceServiceTest {
    @Mock GlEntryRepository repo;
    @InjectMocks TrialBalanceService service;

    @Test
    void shouldBeBalancedWhenDebitEqualsCredit() {
        when(repo.findByEngagementId(1L)).thenReturn(List.of(
            new GlEntry(new BigDecimal("100.00"), BigDecimal.ZERO),
            new GlEntry(BigDecimal.ZERO, new BigDecimal("100.00"))
        ));
        assertTrue(service.isBalanced(1L));
    }
}
```

### JavaScript (Jest / Vitest)
```js
import { total } from "./ledger";

test("adds all amounts", () => {
  expect(total([{ amount: 100 }, { amount: 50 }])).toBe(150);
});

test("empty list gives 0", () => {
  expect(total([])).toBe(0);
});
```

### React Testing Library
```jsx
test("shows loading then rows", async () => {
  render(<LedgerTable />);
  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  expect(await screen.findByText("Rent")).toBeInTheDocument();
});
```

### Python (pytest)
```python
def test_total():
    assert total([100, 50]) == 150
```

### What to test (edge cases)
Empty input, null, zero, negative numbers, very large numbers, wrong date formats, duplicate rows.

---

## 4. Words to Know

- **Mock:** a fake object that returns what you tell it (e.g. fake DB).
- **TDD (Test-Driven Development):** write the test first → see it fail → write code → see it pass → clean up.
- **Code coverage:** % of code run by tests. 70 to 80% is good; 100% isn't the goal.
- **AAA pattern:** **A**rrange (set up) → **A**ct (run) → **A**ssert (check).

---

## 5. CI/CD

- **CI (Continuous Integration):** every push/PR automatically runs **build + lint + tests**.
- **CD (Continuous Delivery/Deployment):** after tests pass, the app is automatically deployed to staging/production.

### Example GitHub Actions file
```yaml
name: CI
on: [pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin' }
      - run: ./mvnw test
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: cd client && npm ci && npm run lint && npm test
```

**Tools:** GitHub Actions, GitLab CI, Jenkins, AWS CodePipeline, Azure DevOps.

**Environments:** Dev → Staging (QA tests here) → Production.

---

## 6. Docker (basic)

- **Docker** packages the app + everything it needs into a **container**, so it runs the same everywhere. No more "works on my machine".
- **Image** = the recipe/snapshot. **Container** = a running copy of the image.

```dockerfile
FROM eclipse-temurin:21-jre
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- **docker-compose** runs many containers together (app + Postgres + Redis).
- **Kubernetes** manages lots of containers at scale (just know the name).

---

## 7. Cloud: AWS Basics (AWS is preferred in the JD)

| Service | What it is | FDD use |
|---|---|---|
| **EC2** | Virtual server | Run the backend |
| **S3** | File storage | Store uploaded GL Excel/CSV files and final reports |
| **RDS** | Managed SQL database | PostgreSQL/MySQL for all data |
| **Lambda** | Run code without a server | Process a file when it's uploaded to S3 |
| **SQS** | Message queue | Queue big GL import jobs |
| **IAM** | Users and permissions | Who can access what |
| **CloudWatch** | Logs and monitoring | See errors and alerts |
| **ECS / EKS** | Run containers | Deploy Docker images |
| **CloudFront** | CDN | Serve the React app fast |
| **Secrets Manager** | Store passwords/API keys safely | DB password, AI API key |
| **VPC** | Private network | Keep the DB away from the internet |

**Simple architecture you can draw:**
```
User → CloudFront (React app from S3)
     → Load Balancer → Backend on ECS/EC2 → RDS PostgreSQL
                                          → S3 (files)
                                          → SQS → Worker (GL import)
```

### Azure / GCP names (in case)
- Azure: VM, Blob Storage, Azure SQL, Functions, AKS.
- GCP: Compute Engine, Cloud Storage, Cloud SQL, Cloud Functions, GKE.

---

## 8. Security for Client Financial Data

- **Encrypt** data in transit (HTTPS/TLS) and at rest (encrypted DB/S3).
- **Least privilege:** give each user/service only the access it needs.
- Separate data per client (multi-tenant isolation): one CPA firm must never see another's data.
- Keep audit logs.
- Compliance words: **SOC 2** (US standard for security of service companies), GDPR.

---

## 9. Quick Q&A

- **What is a load balancer?** Spreads traffic over many servers.
- **Horizontal vs vertical scaling?** Horizontal = more machines. Vertical = bigger machine.
- **What is a CDN?** Servers around the world that cache static files close to users.
- **Blue-green deployment?** Two environments; switch traffic to the new one, roll back fast if needed.
- **What is a health check?** An endpoint like `/actuator/health` that says the app is alive.

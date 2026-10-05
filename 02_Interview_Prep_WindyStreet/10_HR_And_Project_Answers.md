# HR, Behavioral and Project Answers (Easy Notes)

This round decides a lot. Most people lose marks here by giving long, unclear answers.
**Rule:** short, clear, honest, with one real example.

> ✏️ These answers use your resume. Change anything in `[brackets]` (like team size) to your real details. Practise saying each answer **out loud**.
> Every resume point with likely questions is in `13_My_Resume_Vs_JD.md`.

---

## 1. "Tell me about yourself" (60 to 90 seconds)

**Formula:** Present → Past → Project → Why this job.

> "Hi, I'm Adarsh Kumar Singh. I did my B.Tech in Computer Science from Sharda University, and I'm a Java full stack developer with about **one year of professional experience**.
>
> Right now I work at **Telgoo5** as a Full Stack Developer, on a **multi-tenant telecom SaaS platform**. I build **inventory management REST APIs** in Spring Boot and MySQL for SIM card and device trade-in modules, which handle lakhs of production requests. I've integrated the **FedEx API** for shipping, and I built **scheduled background jobs** for inventory sync and **data reconciliation**. I make sure those jobs are idempotent so they never run twice. Along with development, I also work on our **AWS cloud infrastructure**: I've worked on **cost optimization** to bring down our cloud bill, and I handle **live production operations** like deployments, monitoring, incident fixes and job checks. I deploy with **GitHub Actions** and work in Agile sprints with PR-based code reviews.
>
> Before that I interned at **Rnpsoft** as a Java developer, where I built a voice-based news assistant using Spring Boot.
>
> On the side I built **CodeCampus**, a coding and assessment platform in React and Spring Boot with JWT roles, a multithreaded code-execution engine, proctoring and plagiarism detection.
>
> I'm interested in Windy Street because a lot of my current work, like reconciliation jobs, multi-tenant data isolation, role-based access and processing large datasets, maps directly onto a financial due diligence platform. And I want to grow deeper into backend and data engineering in a domain where correctness really matters."

### Short version (30 seconds, if they say "briefly")
> "I'm Adarsh, a Java full stack developer with about a year of experience at Telgoo5 on a multi-tenant telecom SaaS platform. I build Spring Boot APIs with MySQL and React modules, integrate third-party APIs like FedEx, and write reconciliation and sync jobs. I also work on AWS infrastructure, cost optimization and live production operations. I'm looking to grow into backend and data engineering on a product like your FDD platform."

### Be ready for the follow-up on cloud and cost work
Once you say "cost optimization" and "live operations", they **will** ask: *"How did you reduce cost?"* and *"What do you do in live operations?"* Answers are in `14_Question_Bank.md`, section F5. Prepare **one real example with a number**, e.g.:
> "I found that our staging servers ran 24/7. I scheduled them to stop at night and on weekends, and right-sized a few over-provisioned instances after checking CloudWatch. That cut that part of the bill by about [X]%."

---

## 2. Explaining Your Project (CodeCampus): VERY important

Use your notes in `../01_SpringBoot_Notes/PROJECT_FLOW.md`. Explain in this order:

1. **What it is (1 line):** "A full stack platform where students write and run code, take timed tests, and teachers review results."
2. **Tech stack:** React 19 + Vite, Axios, Spring Boot, Spring Security + JWT, Spring Data JPA/Hibernate, MySQL (H2 for local).
3. **Architecture:** Browser (React) → REST API (Spring Boot) → Database. Controller → Service → Repository layers.
4. **One full flow** (e.g. login):
   - User clicks Login → `AuthContext` calls `POST /api/auth/login`
   - Spring Security allows `/api/auth/**` → `AuthController` → `AuthService` checks the BCrypt password → creates a JWT → returns it
   - Frontend stores the token and sends `Authorization: Bearer <token>` on every request
   - `JwtAuthenticationFilter` checks the token on each call; role checks via `@PreAuthorize`
5. **A challenge you solved:** e.g. securing code execution, handling timed submissions, proctoring, plagiarism comparison, CORS with the Vite proxy.
6. **What you'd improve:** tests, Docker deployment, caching, better monitoring.

### Link it to their product (big plus!)
> "Some ideas carry over directly to your FDD platform: role-based access (student/teacher ↔ staff/manager/partner), keeping a record of every submission (like an audit trail), and processing user input safely on the backend."

### Project questions they might ask
- Why Spring Boot? (Fast setup, auto-config, strong security, big ecosystem.)
- How did you secure the APIs? (JWT + Spring Security + roles + BCrypt.)
- How is the DB designed? (Tell 3 to 4 main tables and their relations.)
- What was the hardest bug? (Use the STAR story below.)
- How would you scale it? (Stateless backend behind a load balancer, DB indexes, caching, queue for code execution.)

---

## 3. STAR Method for Behavioral Questions

**S**ituation → **T**ask → **A**ction → **R**esult. Keep it 1 to 2 minutes.

### Prepare these 5 stories (based on your resume; adjust details to what really happened)

**1. A hard production problem: duplicate cron job runs (Telgoo5)**
> S: "Our inventory sync job ran on a schedule, but sometimes it took longer than the interval, so a second run started before the first one finished."
> T: "I had to stop duplicate runs and make sure the data stayed correct."
> A: "I added an overlap guard (a lock/flag so only one run happens at a time) and made the job idempotent, so running it twice gives the same result, using unique keys and checking what was already processed."
> R: "No more duplicate records, and the reconciliation reports became reliable."
> *Why this story is perfect for Windy Street: their GL imports and reconciliations need exactly this.*

**2. A feature you owned end to end: FedEx integration (Telgoo5)**
> "I got the requirement to automate shipping. I read the FedEx API docs, designed the flow (create shipment → label → tracking → delivery status), built the service in Spring Boot with retry and error handling for timeouts and failures, tested it in Postman, raised a PR, fixed review comments, and it went to production through GitHub Actions. It removed manual label creation and tracking for the ops team."

**3. Learning something new fast: Terraform / AWS pipeline or Judge0**
> "For my AWS pipeline project I had never used Terraform or OIDC. I read the docs, started with a small setup, then built the full pipeline: GitHub Actions authenticates with OIDC (no stored keys), builds a Docker image, pushes to ECR, deploys to ECS Fargate, and rolls back automatically if the deployment fails. I also earned the AWS Cloud Practitioner certificate that year."

**4. Disagreement / feedback in code review**
> "A senior suggested a different approach in my PR. I asked why, understood it handled edge cases better, updated my code, and learned the pattern for later."

**5. Tight deadline / mistake you made**
> "I once pushed a change that broke [X] in staging. I told the team quickly, reverted it, found the cause, and added a test. Since then I always run the full test suite before merging."

---

## 4. Common HR Questions + Simple Answers

**Why Windy Street?**
> "You're building a real product, not just a service, for a domain where accuracy is critical. I like the small team with direct access to senior engineers, and the chance to grow from full stack into backend and data work."

**Why should we hire you?**
> "I already have about a year of production experience on a multi-tenant SaaS platform: REST APIs, MySQL schema design, third-party API integration, scheduled reconciliation jobs, CI/CD on AWS, cloud cost optimization, and live production operations. That's very close to what your FDD platform needs: ingesting data, keeping each client's data separate, reconciling numbers, and shipping reliably. I'm comfortable on React too, I care about correctness, and I've started learning accounting basics like general ledgers and account mapping to understand your product better."

**What do you know about our product?**
> "You're building a platform for financial due diligence used by a US CPA firm when PE firms buy companies. It imports general ledgers, maps accounts, runs around 20 analysis modules like revenue and payroll, checks consistency, and assembles the report, with AI help and an audit trail so partners can review faster."
*(This answer alone puts you ahead of most candidates.)*

**Are you okay moving from frontend to backend/data later?**
> "Yes, definitely. That's actually one reason I applied. I want to grow on the backend and data side, and I'm already practising Python, pandas and openpyxl."

**Strengths?**
> "Debugging and breaking down unfamiliar code. For example, I documented the full folder structure and request flow of my project to understand it deeply."

**Weakness?** (real + fixing it)
> "I used to spend too long trying to solve problems alone. Now I time-box: after about an hour, I ask a senior and explain what I've tried."

**Where do you see yourself in 3 years?**
> "As a strong backend/data engineer who owns important modules of the product and helps mentor new joiners."

**Hybrid / Gurugram okay?** "Yes."

**Notice period / salary?** Be honest. For salary: *"I'm looking for a fair market range for this role. I'm flexible. My expectation is around [X], but learning and growth matter most."*

**Why leave Telgoo5 after one year?** (stay positive, never complain)
> "I've learned a lot at Telgoo5 about production systems. Now I want to work on a data-heavy product in the finance domain, in a small team where I can take more ownership and grow into backend and data engineering. Windy Street's FDD platform is exactly that."

---

## 5. Questions YOU Ask at the End (always ask 2)

1. "What does a typical week look like for this role?"
2. "What's the backend stack, and how do the data pipelines work today?"
3. "How do you handle mentoring and onboarding for new engineers?"
4. "What would success look like in the first 3 to 6 months?"
5. "What's the biggest technical challenge the team is solving right now?"

---

## 6. Communication Tips (JD: "good communication skills")

- **Think out loud** in technical questions: "First I'd check..., then...".
- If you don't know: *"I haven't used that directly, but here's how I'd approach it..."* Never make things up.
- Ask clarifying questions before coding.
- Speak slowly, short sentences, smile.
- Have a clean background, good mic, and stable internet for online rounds.

---

## 7. Resume Tips for This JD (to get shortlisted)

- Put these **keywords** (from the JD) in your resume if you've really used them: React, TypeScript, JavaScript, REST APIs, Spring Boot/Java, SQL, PostgreSQL/MySQL, Git, unit testing, CI/CD, AWS, Python.
- Write bullets as **Action + What + Result (number)**:
  - ✅ "Built 12 REST APIs in Spring Boot for the test module, cutting page load time by 40% with pagination."
  - ❌ "Worked on backend."
- Mention **team work:** "Worked in a 6-member agile team; participated in code reviews and sprint planning."
- Add a line about **data/finance interest** if true: "Built a data import feature for CSV/Excel files."
- Keep it 1 page. Add GitHub + LinkedIn links. Pin CodeCampus + this notes repo on GitHub.
- **Small project idea (2 to 3 days) that will impress them:** a mini "GL Analyzer". Upload a CSV GL → map accounts → show trial balance + monthly P&L + "balanced?" check → export to Excel. Put it on GitHub and in your resume.

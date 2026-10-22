# Infosys: Java, Spring Boot Developer (Start Here)

**Job:** Java, Spring Boot Developer, Infosys, Bangalore (Engineering Services)
**Job ID:** INFSYS-EXTERNAL-254643
**Experience:** 1 to 3 years (you have about 12 months at Telgoo5 + 4-month internship, so you fit)

---

## 1. What This Job Wants (easy words)

This is a **pure Java backend** job. No React, no accounting.

Most important (they will ask a lot):
1. **Core Java** (Java 8 / 11 / 17): OOP, Collections, Streams, Exceptions, Multithreading
2. **Spring Boot, Spring MVC, Spring Data JPA, Hibernate**
3. **Microservices**
4. **REST APIs**
5. **SQL** (MySQL / Oracle / PostgreSQL)

Also asked:
- Unit + integration tests (JUnit, Mockito)
- Git, Maven, Jenkins
- Docker, Kubernetes (preferred)
- Cloud: AWS / Azure / GCP (bonus)
- NoSQL (MongoDB)
- Agile / Scrum, debugging, production support

---

## 2. Do We Already Have Notes? (JD vs existing files)

| JD topic | Already covered? | Where to read |
|---|---|---|
| Spring Boot basics, annotations, layers, DI | ✅ Good | `01_SpringBoot_Notes/00_PART1_SHORT_NOTES.md`, `SPRINGBOOT_REFRESHER.md` |
| Spring Security + JWT | ✅ Good | `01_SpringBoot_Notes/00_PART1_SHORT_NOTES.md` §5 |
| REST APIs, status codes | ✅ Good | `02_.../03_Backend_REST_APIs.md`, `14_Question_Bank.md` Part B |
| SQL, JOINs, indexes, query tuning | ✅ Good | `02_.../04_Databases_SQL.md` |
| Hibernate / JPA basics (lazy, N+1, `@Transactional`) | 🟡 Basic | `14_Question_Bank.md` C8 to C11 |
| Git, code review, Agile/Scrum | ✅ Good | `02_.../05_Git_And_Team_Workflow.md` |
| Debugging, production issues | ✅ Good | `01_SpringBoot_Notes/DEBUGGING.md` |
| Multithreading, JVM memory, HashMap internals | ✅ Good | `02_.../13_My_Resume_Vs_JD.md` §5 |
| JUnit + Mockito | 🟡 Basic | `02_.../06_Testing_CICD_Cloud.md` |
| Docker, AWS, CI/CD | 🟡 Basic (your AWS pipeline is strong) | `02_.../06_Testing_CICD_Cloud.md`, `13_My_Resume_Vs_JD.md` §7 |
| Your project answers (Telgoo5, CodeCampus) | ✅ Good | `02_.../13_My_Resume_Vs_JD.md`, `10_HR_And_Project_Answers.md` |
| **Core Java deep** (OOP, String, Collections, equals/hashCode, exceptions) | 🔴 Only 1-line answers | **Need new notes** |
| **Java 8 / 11 / 17 features** (Streams, Optional, lambdas, records) | 🔴 Only 1 short answer | **Need new notes** |
| **Stream coding programs** (Infosys asks these a lot) | 🔴 Missing | **Need new notes** |
| **Microservices** (API Gateway, service discovery, circuit breaker, Feign, config server) | 🔴 Only 1 line | **Need new notes** |
| **Spring MVC internals** (DispatcherServlet, request flow) | 🔴 Missing | **Need new notes** |
| **Spring Data JPA deep** (relationships, JPQL, `@Query`, pagination, entity states) | 🔴 Missing | **Need new notes** |
| **Maven, Jenkins** | 🔴 Missing | **Need new notes** |
| **Kubernetes basics** | 🔴 Missing | **Need new notes** |
| **MongoDB / NoSQL basics** | 🔴 Missing | **Need new notes** |
| Java coding round (strings, arrays, simple DSA) | 🟡 Some | `02_.../11_Coding_Round.md` |

**Short answer:** Spring Boot, REST, SQL, Git and project stories are ready. **Core Java and Microservices are the big gaps**, and they are the main part of this job.

**Skip for Infosys:** React, JavaScript, Python, accounting/FDD, Excel files (`01`, `02`, `07`, `08`, `09` in the Windy Street folder).

---

## 3. Planned Files for This Folder

| File | What it will have |
|---|---|
| `01_My_Project_Stories.md` ✅ done | Deep project explanation (Telgoo5, Rnpsoft, CodeCampus, AWS) + 10 easy STAR stories |
| `02_Core_Java.md` | OOP, String, Collections, equals/hashCode, exceptions, immutability |
| `03_Java8_11_17_And_Stream_Programs.md` | Lambdas, Streams, Optional, records + 20 stream coding questions |
| `04_Spring_MVC_JPA_Hibernate_Deep.md` | DispatcherServlet, JPA relationships, JPQL, pagination, Hibernate cache |
| `05_Microservices.md` | Gateway, Eureka, Feign, circuit breaker, config server, saga, Kafka basics |
| `06_Maven_Jenkins_Docker_K8s.md` | Build, pipeline, container, pods/deployments in easy words |
| `07_Question_Bank_Infosys.md` | Infosys-style questions + your resume-based answers |

---

## 4. Your Strong Points for This Job (say these)

- **Daily Spring Boot + MySQL work at Telgoo5:** production REST APIs, schema design, query tuning.
- **Production support:** debugging across backend, DB and deployment, which matches "production issue resolution" in the JD.
- **Scheduled jobs:** idempotent cron jobs with overlap locks (ShedLock).
- **Third-party integration:** FedEx APIs with retry and backoff.
- **Multithreading:** thread pool + Judge0 sandbox in CodeCampus.
- **DevOps:** GitHub Actions + Docker + ECS + Terraform pipeline, which covers the Docker, CI/CD and AWS parts of the JD.

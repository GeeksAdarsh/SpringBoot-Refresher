# Spring Boot Refresher

Personal notes for relearning **Spring Boot** and full-stack project architecture — written using a real full-stack project ([CodeCampus](https://github.com/GeeksAdarsh/Code_Campus-Compiler)) as the worked example instead of toy tutorials.

## Contents

| File | What's inside |
|---|---|
| [`SPRINGBOOT_REFRESHER.md`](./SPRINGBOOT_REFRESHER.md) | Core Spring Boot concepts — annotations glossary, layered architecture (`controller → service → repository → model`), Dependency Injection, JPA/Hibernate, Spring Security + JWT, DTOs vs Entities, plus a quick-recall cheat sheet |
| [`PROJECT_FLOW.md`](./PROJECT_FLOW.md) | How a real full-stack app works end-to-end — the request lifecycle, and concrete "button click → API call → DB → response" traces for real features (login, code execution, test submission, proctoring, plagiarism checks) |
| [`DEBUGGING.md`](./DEBUGGING.md) | Practical, symptom-first troubleshooting guide — HTTP status code meanings, common Spring/Java exceptions, backend startup errors, frontend issues, and a quick symptom → file map |
| [`FOLDER_STRUCTURE.md`](./FOLDER_STRUCTURE.md) | File-by-file breakdown of a full-stack React + Spring Boot repo's folder structure, with a short explanation of what each file/class is for |

## How to use this

1. Start with `SPRINGBOOT_REFRESHER.md` to rebuild the core mental model.
2. Read `PROJECT_FLOW.md` to see those concepts fire in sequence for real features.
3. Keep `DEBUGGING.md` open as a reference whenever something breaks.
4. Use `FOLDER_STRUCTURE.md` to quickly look up what any individual file does.

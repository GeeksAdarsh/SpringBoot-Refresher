# Git, Team Workflow and Debugging (Easy Notes)

The JD says: *"Comfortable with Git and standard workflows (branching, PRs, code review)"*, *"participate in code reviews, sprint planning"*, and *"strong problem-solving and debugging ability; can work through an unfamiliar codebase"*.

Because they want 6 to 18 months of experience, they will check that you have **really worked in a team**.

---

## 1. Git Basics

| Command | What it does |
|---|---|
| `git clone <url>` | Copy a repo to your computer |
| `git status` | See what changed |
| `git add .` | Stage changes |
| `git commit -m "msg"` | Save a snapshot |
| `git push` | Send commits to remote (GitHub) |
| `git pull` | Get latest changes (fetch + merge) |
| `git fetch` | Download changes but don't merge |
| `git branch feature/x` | Make a branch |
| `git checkout -b feature/x` / `git switch -c feature/x` | Make and move to a branch |
| `git merge main` | Bring main's changes into your branch |
| `git rebase main` | Put your commits on top of latest main (clean history) |
| `git log --oneline` | Short history |
| `git diff` | See line changes |
| `git stash` / `git stash pop` | Save unfinished work aside / bring it back |
| `git reset --soft HEAD~1` | Undo last commit, keep changes |
| `git revert <hash>` | Make a new commit that undoes an old one (safe for shared branches) |
| `git cherry-pick <hash>` | Copy one commit to the current branch |

### merge vs rebase
- **Merge:** keeps full history, adds a merge commit. Safe.
- **Rebase:** rewrites your commits on top of main; history is a straight line. **Never rebase a shared branch.**

### reset vs revert
- **reset:** moves the branch back (rewrites history; local only).
- **revert:** adds a new "undo" commit (safe on main).

### Merge conflict: how to fix
1. `git pull` (or merge main) → Git shows conflict markers `<<<<<<< ======= >>>>>>>`.
2. Open the file, keep the correct code, remove the markers.
3. `git add file` → `git commit` (or `git rebase --continue`).
4. Run tests again.

---

## 2. Daily Team Workflow (explain this in the interview)

```
1. Pick a ticket from Jira (e.g. "FDD-123: Add filter to GL table")
2. git checkout main && git pull
3. git checkout -b feature/FDD-123-gl-filter
4. Write code + tests, commit in small steps
5. git push -u origin feature/FDD-123-gl-filter
6. Open a Pull Request (PR) with a clear description + screenshots
7. CI runs tests/lint automatically
8. Reviewer comments → I fix → push again
9. Approved → Squash and merge to main
10. Deployed to staging → test → production
```

### Good commit messages
- ✅ `feat: add month filter to GL table`
- ✅ `fix: handle empty amount in GL import`
- ❌ `changes`, `final`, `final2`

(This style is called **Conventional Commits**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.)

### Branching strategies
- **GitHub Flow:** main + short feature branches (simple, common in small teams).
- **Git Flow:** main, develop, feature, release, hotfix branches.
- **Trunk-based:** very short branches, merge many times a day, feature flags.

---

## 3. Code Review

### When you review someone's code, check:
- Does it do what the ticket says?
- Any bugs or missed edge cases (empty list, null, negative numbers)?
- Is it readable? Good names?
- Tests added?
- Security (no secrets, input validated)?
- Performance (no query inside a loop)?

### How to give feedback
Be kind and specific: *"Could we move this into a helper? It's repeated in 3 places."* Not: *"This is bad."*

### When you get feedback
Don't take it personally. Ask questions if unclear. Fix it, then reply "Done".

---

## 4. Agile / Scrum

| Word | Meaning |
|---|---|
| Sprint | 1 to 2 week work cycle |
| Sprint planning | Team picks tickets for the sprint |
| Daily standup | 10-15 min: What did I do yesterday? What will I do today? Any blockers? |
| Sprint review/demo | Show finished work |
| Retrospective | What went well / what to improve |
| Story points | Rough size of a task |
| Backlog | List of all pending work |
| Definition of Done | Code + tests + review + deployed |

---

## 5. Debugging: How to Explain Your Approach

Interviewers love a clear **step-by-step** answer:

1. **Reproduce** the bug. Get exact steps, input, user, time.
2. **Read the error** message and stack trace carefully. Find the first line from *your* code.
3. **Find the layer:** Is it frontend, API, or database?
   - Browser **DevTools → Network tab**: what request was sent? What status code came back?
   - **Console tab:** any JS errors?
   - **Backend logs:** any exception?
   - **DB:** run the query directly. Is the data right?
4. **Make a guess (hypothesis)** and test it: breakpoints, logs, small test.
5. **Fix** the root cause, not just the symptom.
6. **Add a test** so it doesn't come back.
7. **Write it down** (PR description / docs).

> **Easy memory:** *Reproduce → Read → Locate → Test guess → Fix → Prevent.*

### Common bug examples (good to use in answers)
- `Cannot read properties of undefined`: data not loaded yet → add loading state / `?.`
- CORS error → backend not allowing the frontend origin
- 401 after some time → JWT expired → refresh token
- Totals off by 0.01 → floating point → use decimals
- Works locally, fails in prod → different env variables / config / data

---

## 6. Working in an Unfamiliar Codebase (JD mentions this)

Say this:
1. Read the README and setup docs, and run it locally.
2. Look at the folder structure and find the entry point (`main`, `App.jsx`, routes).
3. Pick one feature and **trace it end to end** (UI → API → DB).
4. Read tests. They show how the code is supposed to work.
5. Use the debugger and "Find usages" in the IDE.
6. Make a small change first (a small bug fix).
7. Ask seniors focused questions after trying for some time.

> You already did this! Your `../01_SpringBoot_Notes/FOLDER_STRUCTURE.md` and `PROJECT_FLOW.md` are exactly this method. **Tell the interviewer about it.**

---

## 7. Escalation (JD: "escalating complex problems appropriately")

- Try on your own first for a fixed time (e.g. 30 to 60 minutes).
- Then ask, and share: **what the problem is, what you tried, what you think the cause is.**
- Escalate immediately if: production is down, data might be wrong/lost, or there's a security issue.

---

## 8. Documentation (JD: "write basic technical documentation")

- README: how to set up and run.
- API docs: Swagger / OpenAPI (Spring: `springdoc-openapi`).
- PR descriptions: what, why, how to test.
- Comments only for **why** something is done, not **what**.

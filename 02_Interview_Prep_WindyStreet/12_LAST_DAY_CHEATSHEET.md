# LAST DAY CHEAT SHEET

Read this 1 hour before the interview. Everything important is on one page.

---

## The Company in 3 Lines
- Windy Street = an accounting outsourcing firm for US clients, led by ex-Big-4 people.
- Building an **FDD (Financial Due Diligence) platform** for a US CPA firm, used when **PE firms buy companies**.
- 3 stages: **Data construction** (import GL, map accounts, reconcile) → **Analysis** (rent, payroll, revenue, healthcare waterfall) → **Consistency checks + report assembly**.

## Your Pitch in 3 Lines
- Java full stack dev, **1 year at Telgoo5** (multi-tenant SaaS: inventory APIs, MySQL, FedEx integration, reconciliation cron jobs, AWS + GitHub Actions, **cloud infra, cost optimization, live production operations**) + Rnpsoft internship.
- Built **CodeCampus** (React, Spring Boot, JWT roles, multithreaded Judge0 execution, proctoring, plagiarism scoring).
- Want to grow into **backend and data**; already learning Python, pandas, openpyxl, and accounting basics.

---

## 15 Accounting Words
General Ledger · Chart of Accounts · Debit/Credit (always equal) · Journal Entry · Trial Balance · P&L · Balance Sheet (Assets = Liabilities + Equity) · Revenue · COGS · EBITDA · Adjusted EBITDA · Quality of Earnings · Net Working Capital · Reconciliation · TTM/LTM

## Tech One-Liners
| Topic | Remember |
|---|---|
| `let/const` | const default; `===` always |
| Closure | Inner function remembers outer variables |
| Event loop | Sync → microtasks (Promise) → macrotasks (setTimeout) |
| async/await | Cleaner Promises; wrap in try/catch |
| TypeScript | Types catch bugs early; avoid `any` |
| Money | Never float. Use cents/`BigDecimal`/`DECIMAL(18,2)`/`Decimal` |
| React state vs props | State = own memory; props = from parent, read-only |
| useEffect `[]` | Runs once; return a cleanup function |
| Keys | Use unique id, not index |
| Big tables | Pagination / virtualization, `useMemo` for totals |
| REST | Nouns in URL, HTTP verbs, status codes, JSON, stateless |
| 401 vs 403 | Not logged in vs no permission |
| Layers | Controller → Service → Repository → DB |
| `@Transactional` | All or nothing |
| JWT | Login → token → `Authorization: Bearer` → filter validates |
| Audit trail | Who, what, when, old → new; append-only |
| Multi-user edits | Optimistic locking `@Version` → 409 Conflict |
| Big upload | 202 Accepted + background job + progress polling |
| JOINs | INNER = match only, LEFT = all left + match |
| WHERE vs HAVING | Before vs after GROUP BY |
| Index | Faster reads, slower writes |
| ACID | Atomic, Consistent, Isolated, Durable |
| N+1 | Fix with JOIN FETCH |
| Git | branch → commit → push → PR → review → merge |
| merge vs rebase | Merge keeps history; rebase = straight line, not on shared branches |
| Tests | Unit (many) > Integration > E2E (few) |
| CI/CD | Auto test on PR; auto deploy after pass |
| AWS | EC2, S3, RDS, Lambda, SQS, IAM, CloudWatch |
| ETL | Extract → Transform → Load |
| Adapter pattern | One parser per accounting system → one standard format |
| openpyxl | `load_workbook`, `ws.iter_rows(values_only=True)`, `ws.append`, `wb.save` |
| python-docx | `Document()`, `add_heading`, `add_table`, `save` |
| AI in finance | AI suggests, human approves, deterministic code computes; temperature 0; validate output |

## Debugging in 6 Words
**Reproduce → Read → Locate → Test → Fix → Prevent**

## Must-Say Lines (use naturally)
1. "I always handle loading, error and empty states."
2. "For money I avoid floating point and use decimal types."
3. "I'd add a consistency check / test so this bug can't come back."
4. "AI can suggest mappings, but final numbers should come from deterministic code with human review and an audit trail."
5. "I'm genuinely excited to grow into backend and data engineering."

## Before You Join the Call
- [ ] Resume open, with numbers ready for each bullet
- [ ] CodeCampus flow ready (login → JWT → controller → service → DB)
- [ ] 5 STAR stories ready
- [ ] 1 cost optimization example with a real number
- [ ] Scenario formula: check → find cause → fix → prevent (+ inform the team)
- [ ] 2 questions to ask them
- [ ] Water, charger, quiet room, camera on, smile 🙂

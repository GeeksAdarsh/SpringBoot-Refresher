# Databases and SQL (Easy Notes)

The JD says: *"Basic understanding of databases (SQL/NoSQL queries and simple schema design)"* and good to have *"relational databases in a production setting"*.

Accounting data is **tables of numbers**, so SQL is very important for this job.

---

## 1. SQL vs NoSQL

| SQL (Relational) | NoSQL |
|---|---|
| Tables, rows, columns | Documents (JSON), key-value, graph |
| Fixed schema | Flexible schema |
| Strong **ACID** transactions | Often eventually consistent |
| MySQL, PostgreSQL, SQL Server | MongoDB, DynamoDB, Redis |
| **Best for accounting/finance** | Good for logs, flexible data, caching |

**Interview line:** "For financial data I'd choose a relational DB like PostgreSQL, because we need correct totals, transactions and strong relationships between tables."

---

## 2. ACID (asked a lot)

- **A**tomicity: all or nothing.
- **C**onsistency: data always follows the rules (e.g. debit = credit).
- **I**solation: two transactions at the same time don't mess each other up.
- **D**urability: once saved, it stays saved, even after a crash.

---

## 3. Keys

- **Primary key:** unique id for each row.
- **Foreign key:** a column that points to another table's primary key.
- **Unique key:** no duplicates allowed (e.g. account code per company).
- **Composite key:** primary key made of 2+ columns.

---

## 4. Sample Schema for an FDD App (practice drawing this!)

```
companies         (id, name, accounting_system)          -- QuickBooks, NetSuite, Xero...
engagements       (id, company_id FK, name, status, start_date)
accounts          (id, company_id FK, account_code, account_name, account_type)
standard_categories (id, name)                           -- Revenue, COGS, Payroll, Rent...
account_mappings  (id, account_id FK, category_id FK, mapped_by, version)
gl_entries        (id, engagement_id FK, account_id FK, entry_date, description,
                   debit DECIMAL(18,2), credit DECIMAL(18,2), source_file)
users             (id, name, email, role)                -- STAFF, MANAGER, PARTNER
audit_log         (id, entity_type, entity_id, action, old_value, new_value, changed_by, changed_at)
```

**Relationships:**
- 1 company → many engagements (one-to-many)
- 1 company → many accounts
- 1 account → many GL entries
- Account ↔ category through a mapping table

**Why `DECIMAL(18,2)` and not `FLOAT`?** FLOAT has rounding errors. Money must be exact.

---

## 5. Basic Queries

```sql
SELECT account_name, account_type FROM accounts WHERE company_id = 1;

INSERT INTO accounts (company_id, account_code, account_name, account_type)
VALUES (1, '6100', 'Office Rent', 'EXPENSE');

UPDATE accounts SET account_name = 'Rent Expense' WHERE id = 10;

DELETE FROM accounts WHERE id = 10;

SELECT * FROM gl_entries ORDER BY entry_date DESC LIMIT 50 OFFSET 0;
```

---

## 6. JOINs (most asked)

| Join | Returns |
|---|---|
| INNER JOIN | Only rows that match in both tables |
| LEFT JOIN | All rows from left + matching from right (NULL if no match) |
| RIGHT JOIN | All rows from right + matching from left |
| FULL OUTER JOIN | Everything from both |
| SELF JOIN | Table joined with itself (e.g. employee → manager) |

```sql
-- GL entries with account names
SELECT g.entry_date, a.account_name, g.debit, g.credit
FROM gl_entries g
INNER JOIN accounts a ON a.id = g.account_id;

-- Accounts that are NOT mapped yet (very real FDD task!)
SELECT a.account_code, a.account_name
FROM accounts a
LEFT JOIN account_mappings m ON m.account_id = a.id
WHERE m.id IS NULL;
```

---

## 7. GROUP BY, HAVING, Aggregates

```sql
-- Total per account
SELECT a.account_name, SUM(g.debit) AS total_debit, SUM(g.credit) AS total_credit
FROM gl_entries g
JOIN accounts a ON a.id = g.account_id
GROUP BY a.account_name;

-- Only accounts with more than 1 lakh spent
SELECT account_id, SUM(debit) AS spent
FROM gl_entries
GROUP BY account_id
HAVING SUM(debit) > 100000;

-- Monthly totals (revenue by month: a real "revenue cut")
SELECT DATE_TRUNC('month', entry_date) AS month, SUM(credit - debit) AS revenue
FROM gl_entries g
JOIN accounts a ON a.id = g.account_id
WHERE a.account_type = 'REVENUE'
GROUP BY month
ORDER BY month;
```

**WHERE vs HAVING:** WHERE filters rows **before** grouping. HAVING filters groups **after** grouping.

**Order SQL runs in:** FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT.

---

## 8. Consistency Check Queries (JD mentions "consistency check library")

```sql
-- 1. Is the trial balance balanced? (total debit must equal total credit)
SELECT SUM(debit) - SUM(credit) AS difference
FROM gl_entries
WHERE engagement_id = 5;      -- should be 0

-- 2. Duplicate entries?
SELECT account_id, entry_date, debit, credit, COUNT(*)
FROM gl_entries
GROUP BY account_id, entry_date, debit, credit
HAVING COUNT(*) > 1;

-- 3. Entries with no account
SELECT * FROM gl_entries WHERE account_id IS NULL;
```

---

## 9. Window Functions (bonus, impresses interviewers)

```sql
-- Running balance per account
SELECT account_id, entry_date, debit - credit AS amount,
       SUM(debit - credit) OVER (PARTITION BY account_id ORDER BY entry_date) AS running_balance
FROM gl_entries;

-- Rank top expenses
SELECT account_id, SUM(debit) AS total,
       RANK() OVER (ORDER BY SUM(debit) DESC) AS rnk
FROM gl_entries
GROUP BY account_id;

-- Compare this month to last month
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month
FROM monthly_revenue;
```

`ROW_NUMBER()` = 1,2,3,4. `RANK()` = 1,2,2,4. `DENSE_RANK()` = 1,2,2,3.

---

## 10. Classic Interview SQL Questions

```sql
-- 2nd highest salary
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);

-- Nth highest (using DENSE_RANK)
SELECT salary FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS r FROM employees
) t WHERE r = 3;

-- Find duplicates
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Delete duplicates but keep one (PostgreSQL)
DELETE FROM users a USING users b WHERE a.id > b.id AND a.email = b.email;

-- Employees earning more than their manager
SELECT e.name FROM employees e JOIN employees m ON e.manager_id = m.id WHERE e.salary > m.salary;

-- Department with highest total salary
SELECT dept_id, SUM(salary) s FROM employees GROUP BY dept_id ORDER BY s DESC LIMIT 1;
```

---

## 11. Indexes

- An **index** is like a book index: it helps find rows fast without reading the whole table.
- Add indexes on columns used in **WHERE, JOIN, ORDER BY** (e.g. `gl_entries(engagement_id, account_id)`).
- **Downside:** slower inserts/updates and more storage.
- Check slow queries with `EXPLAIN`.

---

## 12. Normalization (simple)

- **1NF:** one value per cell, no repeating groups.
- **2NF:** every column depends on the whole primary key.
- **3NF:** no column depends on another non-key column.
- **Why?** Avoid duplicate data and update mistakes.
- **Denormalize** sometimes for fast reports (e.g. a summary table).

---

## 13. Transactions and Isolation

```sql
BEGIN;
INSERT INTO gl_entries (...) VALUES (... debit 1000 ...);
INSERT INTO gl_entries (...) VALUES (... credit 1000 ...);
COMMIT;      -- or ROLLBACK if anything fails
```

Problems isolation prevents: **dirty read** (reading unsaved data), **non-repeatable read**, **phantom read**.
Levels: Read Uncommitted → Read Committed (default in Postgres) → Repeatable Read (default in MySQL) → Serializable.

---

## 14. JPA / ORM Quick Points

- **ORM** maps Java classes to tables (Hibernate).
- **N+1 problem:** loading 100 accounts, then running 1 extra query per account for its entries = 101 queries. Fix with `JOIN FETCH` or `@EntityGraph`.
- **LAZY vs EAGER:** LAZY loads related data only when needed (preferred).
- For big reports, write a **native SQL query** or a projection instead of loading full entities.

---

## 15. Quick Q&A

- **DELETE vs TRUNCATE vs DROP?** DELETE removes rows (can use WHERE, can rollback). TRUNCATE removes all rows fast. DROP deletes the whole table.
- **UNION vs UNION ALL?** UNION removes duplicates; UNION ALL keeps them (faster).
- **What is a view?** A saved query that acts like a table.
- **Stored procedure?** SQL code saved inside the DB.
- **What is a CTE?** `WITH t AS (SELECT ...) SELECT * FROM t;` A named temporary result. Makes long queries readable.

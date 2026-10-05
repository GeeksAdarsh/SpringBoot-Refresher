# Domain Knowledge: FDD and Accounting (Easy Notes)

⭐ **This file can make you stand out.** Most developers know nothing about accounting. Windy Street is an accounting firm. If you understand their world even a little, they will remember you.

You don't need to be an accountant. You just need to understand the **words** and the **workflow**.

---

## 1. The Story (who is who)

```
Private Equity (PE) firm  →  wants to BUY a "target company"
        │
        │ hires
        ▼
US CPA firm (accountants)  →  does Financial Due Diligence on the target
        │
        │ outsources work to
        ▼
Windy Street (India)       →  does the analysis + is building the TOOL to do it faster
        │
        ▼
Final report signed by a PARTNER at the CPA firm
```

- **Private Equity (PE):** an investment firm that buys companies, improves them, and sells them later for profit.
- **Target company:** the company being bought.
- **CPA firm:** a US accounting firm (CPA = Certified Public Accountant).
- **Partner:** the senior person who reviews and **signs** the final report. The JD says the tool should make the partner's review faster.
- **Engagement:** one project / one deal job (e.g. "Due diligence on ABC Corp").

---

## 2. What is Financial Due Diligence (FDD)?

Before buying a company, the buyer wants to know: **"Are the numbers real? Is the profit really this much? Any hidden problems?"**

FDD = a detailed check of the target's financial records. It usually takes a few weeks and ends with a report.

### The most important idea: Quality of Earnings (QoE)
- A company says "we made $10M profit (EBITDA)".
- The FDD team checks and **adjusts** it: remove one-time items (e.g. a lawsuit payment, owner's personal expenses, a one-time big sale).
- The result is **"Adjusted EBITDA"**, the real repeating profit.
- PE firms usually pay a price like **"8 × Adjusted EBITDA"**, so a $1M mistake in EBITDA = an $8M mistake in price. **That's why accuracy matters so much.**

---

## 3. Accounting Words You Must Know

| Word | Simple meaning |
|---|---|
| **General Ledger (GL)** | The full list of every money transaction of the company. Each row: date, account, description, debit, credit. This is the main input to the FDD tool. |
| **Chart of Accounts (CoA)** | The list of all account names/codes (e.g. 4000 Sales, 6100 Rent, 6200 Salaries). Every company has a different one. |
| **Account** | A bucket where money is recorded (Cash, Rent Expense, Sales). |
| **Debit / Credit** | Two sides of every entry. **Total debits must always equal total credits.** |
| **Double-entry** | Every transaction touches at least 2 accounts. Pay rent $1000: Rent Expense debit 1000, Cash credit 1000. |
| **Journal Entry (JE)** | One transaction with its debit and credit lines. |
| **Trial Balance (TB)** | A summary: every account and its total balance at a date. Debits = credits if it's correct. |
| **Income Statement / P&L** | Revenue − Expenses = Profit, for a period (month/year). |
| **Balance Sheet** | What the company owns (Assets), owes (Liabilities), and the owner's part (Equity) at a date. **Assets = Liabilities + Equity.** |
| **Cash Flow Statement** | Where cash came from and went. |
| **Revenue** | Money earned from sales. |
| **COGS** | Cost of Goods Sold: direct cost of making the product. |
| **Gross Margin** | (Revenue − COGS) / Revenue. |
| **Operating Expenses (OpEx)** | Rent, salaries, marketing, etc. |
| **EBITDA** | Earnings Before Interest, Taxes, Depreciation, Amortization. A common profit measure. |
| **Adjusted EBITDA** | EBITDA after removing one-time/unusual items. |
| **Net Working Capital (NWC)** | Current assets − current liabilities (short-term money health). Often negotiated in deals. |
| **Accounts Receivable (AR)** | Money customers still owe the company. |
| **Accounts Payable (AP)** | Money the company still owes suppliers. |
| **Accrual** | Recording income/expense when it happens, not when cash moves. |
| **Reconciliation** | Checking two sources match (e.g. GL total vs bank statement, or GL vs trial balance). |
| **Fiscal year / Period** | The company's 12-month accounting year; periods = months/quarters. |
| **TTM / LTM** | Trailing/Last Twelve Months. Very common in FDD. |
| **Audit** | Independent check that financial statements are correct. |
| **Audit trail** | Record showing who changed what, when (for trust and review). |

---

## 4. The 3 Stages From the JD (explained simply)

### Stage 1: Data Construction
- **Ingest the general ledger:** import the GL exported from the company's accounting software.
- **Map accounts:** each company names accounts differently. "Office Rent", "Rent - HQ", "Lease Expense" → all mapped to one standard category **"Rent"**. This makes analysis consistent.
- **Multi-period reconciliations:** check that numbers match across months/years, and GL totals match the trial balance and financial statements.

### Stage 2: Analysis (around 20 modules)
- **Rent schedule:** list of all leases, monthly rent, any changes. Is rent normal or is the owner's family renting at a cheap price?
- **Payroll analysis:** salaries, headcount, bonuses, are costs normal?
- **Revenue cuts:** revenue split by customer, product, region, month. Is the company too dependent on one customer?
- **Healthcare waterfalls:** for healthcare companies, the step-by-step path from **gross charges → minus discounts/denials → net revenue actually collected** (it looks like a waterfall chart).

### Stage 3: Consistency Checking and Report Assembly
- **Consistency checks:** automatic rules, e.g. "Does revenue in the payroll module match the P&L?", "Does the TB balance?", "Does the same number appear the same in every table of the report?"
- **Report assembly:** build the final Word/Excel/PowerPoint deliverable (this is where openpyxl and python-docx come in).

---

## 5. Accounting Systems Mentioned in the JD

| System | Who uses it | Notes |
|---|---|---|
| **QuickBooks** (Intuit) | Small businesses | Most common in the US. Has an online API. |
| **Xero** | Small businesses | Cloud, popular in UK/Australia, good API. |
| **NetSuite** (Oracle) | Mid-size / bigger companies | Cloud ERP, more complex. |
| **Sage Intacct** | Mid-size companies | Cloud accounting, strong in US. |

Each exports the GL in a **different format** (different column names, date formats, debit/credit in 2 columns vs 1 signed amount column). The engineer's job is to write **parsers/adapters** that turn each format into **one standard format**. See `08_Data_Pipelines_Excel_AI.md`.

---

## 6. Words From the JD Explained

- **Deterministic computation engines:** code where the same input always gives the exact same output. No randomness. Needed so partners can trust and re-check numbers.
- **AI-augmented analysis:** AI helps (e.g. suggests account mappings, writes draft commentary, flags unusual entries), but **humans review**.
- **Multi-user coordination:** many accountants work on one engagement at once (needs roles, locking, live updates).
- **Audit-trail layer:** every change is recorded so the partner can see how each number was made.
- **Signed-by-partner deliverable:** the final report that the partner approves and signs.

---

## 7. A Simple Example to Explain in the Interview

> "Suppose a target company's GL has 50,000 rows from QuickBooks. The tool imports it, then maps their 300 accounts to about 40 standard categories, with AI suggesting mappings and a staff member confirming them. Then it builds the monthly P&L, checks that debits equal credits and that the TB matches, and runs modules like revenue by customer. Any mismatch is flagged by the consistency checks. Every edit is in the audit trail, so when the partner reviews, they can click a number and see exactly where it came from."

**Practise saying this. It shows you understood their product.**

---

## 8. Smart Questions to Ask Them (pick 2 or 3)

1. "Which accounting systems are most common in your current engagements, QuickBooks or NetSuite?"
2. "How do you handle account mapping today? Is it rules-based, AI-assisted, or manual?"
3. "What's the tech stack for the backend: Python, Node or Java?"
4. "How is the consistency check library designed: rules in code or configurable?"
5. "How will a new engineer be mentored in the first 3 months?"
6. "What does success look like for this role after 6 months?"

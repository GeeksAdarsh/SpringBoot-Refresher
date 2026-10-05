# Data Pipelines, Excel/Word Automation and AI APIs (Easy Notes)

The JD says:
- *"Assist with general ledger ingestion pipelines and data mapping logic for QuickBooks, NetSuite, Xero, Sage Intacct"*
- Good to have: *"Excel/Word automation libraries (openpyxl, python-docx)"* (mentioned **two times**, so they really want it!)
- Bonus: *"AI/LLM APIs (Anthropic Claude, OpenAI)"*
- The role may move toward **backend/data** over time.

---

## 1. What is a Data Pipeline?

Moving data from a source → cleaning/changing it → saving it somewhere useful.

**ETL = Extract → Transform → Load**

| Step | In the FDD tool |
|---|---|
| **Extract** | Read GL from an uploaded Excel/CSV file, or from the QuickBooks/Xero API |
| **Transform** | Rename columns, fix dates, convert debit/credit, remove blank rows, map accounts |
| **Load** | Save clean rows into the `gl_entries` table |

**ELT** = load raw data first, transform later inside the database (common with big data warehouses).

---

## 2. GL Ingestion: Step by Step (explain this!)

```
Upload file (xlsx/csv) ──► Store original in S3 (keep for audit)
        │
        ▼
Detect source (QuickBooks? NetSuite? Xero? Sage?) by its column headers
        │
        ▼
Parse with the right ADAPTER → convert to ONE standard format:
   { date, account_code, account_name, description, debit, credit, source_row }
        │
        ▼
Validate:  dates valid? amounts numeric? debit = credit overall? duplicates?
        │
        ▼
Save to DB in a transaction (all or nothing) + record import job + audit log
        │
        ▼
Map accounts → standard categories (rules + AI suggestions + human approval)
        │
        ▼
Report errors back to the user with row numbers ("Row 1,204: date is invalid")
```

### Adapter pattern (good design word to say)
```python
class GLAdapter:
    def parse(self, file) -> list[dict]: ...

class QuickBooksAdapter(GLAdapter): ...
class XeroAdapter(GLAdapter): ...
class NetSuiteAdapter(GLAdapter): ...

ADAPTERS = {"quickbooks": QuickBooksAdapter(), "xero": XeroAdapter()}
rows = ADAPTERS[source].parse(file)
```
**Why?** Adding Sage Intacct later = just add one new adapter class. Nothing else changes. (This follows the **Open/Closed principle**.)

### Common messy-data problems
- Different date formats (`03/04/2024` = March 4 in US, April 3 in India!)
- Amounts like `"$1,200.50"`, `"(500)"` (brackets = negative in accounting)
- Debit and credit in separate columns vs one signed amount column
- Blank rows, subtotal rows, header rows in the middle
- Same account with slightly different names
- Very big files (500k+ rows) → read in **chunks**, use background jobs

---

## 3. Account Mapping Logic

**Goal:** map each company's accounts → standard categories.

| Company's account | Standard category |
|---|---|
| 6100 Office Rent | Rent |
| 6105 Rent - Warehouse | Rent |
| 6200 Wages | Payroll |
| 6210 Payroll Taxes | Payroll |

### Ways to do it
1. **Rules:** if name contains "rent" or "lease" → Rent. Account code 4000-4999 → Revenue.
2. **Reuse past mappings:** same accounting system/industry seen before.
3. **AI suggestion:** send account name + description to an LLM, get a category + confidence.
4. **Human confirms** anything uncertain. Every decision is saved in the audit trail.

---

## 4. Python + pandas (used a lot for data work)

```python
import pandas as pd

df = pd.read_excel("gl.xlsx")             # or pd.read_csv("gl.csv")
df.columns = df.columns.str.strip().str.lower()

df["date"] = pd.to_datetime(df["date"], format="%m/%d/%Y")
df["debit"] = df["debit"].fillna(0)
df["credit"] = df["credit"].fillna(0)
df = df.dropna(subset=["account"])         # remove rows without account

# Check balanced
diff = df["debit"].sum() - df["credit"].sum()
print("Balanced" if abs(diff) < 0.01 else f"Off by {diff}")

# Total by account
summary = df.groupby("account")[["debit", "credit"]].sum().reset_index()

# Monthly revenue
df["month"] = df["date"].dt.to_period("M")
monthly = df.groupby("month")["credit"].sum()

summary.to_excel("summary.xlsx", index=False)
```

Key pandas words: **DataFrame** (table), `groupby`, `merge` (= SQL join), `pivot_table`, `fillna`, `dropna`, `apply`.

---

## 5. openpyxl: Read and Write Excel (Python)

```python
from openpyxl import load_workbook, Workbook
from openpyxl.styles import Font, PatternFill

# READ
wb = load_workbook("gl.xlsx", data_only=True)   # data_only=True → values, not formulas
ws = wb.active                                   # or wb["Sheet1"]
for row in ws.iter_rows(min_row=2, values_only=True):
    date, account, debit, credit = row[:4]
    print(account, debit, credit)

# WRITE
wb = Workbook()
ws = wb.active
ws.title = "Trial Balance"
ws.append(["Account", "Debit", "Credit"])        # header
ws["A1"].font = Font(bold=True)
ws.append(["Rent", 10000, 0])
ws.append(["Cash", 0, 10000])
ws["B4"] = "=SUM(B2:B3)"                          # Excel formula
ws["B2"].number_format = "#,##0.00"
ws.column_dimensions["A"].width = 30
ws["A1"].fill = PatternFill("solid", fgColor="DDEBF7")
wb.save("trial_balance.xlsx")
```

Tips:
- For huge files, use `load_workbook(..., read_only=True)` (uses less memory).
- `data_only=True` gives the last saved values of formulas.
- In Java the equivalent is **Apache POI**. In JS: **SheetJS (xlsx)** or **ExcelJS**.

---

## 6. python-docx: Create Word Reports

```python
from docx import Document
from docx.shared import Pt

doc = Document()
doc.add_heading("Financial Due Diligence Report", level=0)
doc.add_paragraph("Target: ABC Corp | Period: FY2023 to TTM Jun-2024")

doc.add_heading("Revenue Summary", level=1)
table = doc.add_table(rows=1, cols=3)
table.style = "Light Grid Accent 1"
hdr = table.rows[0].cells
hdr[0].text, hdr[1].text, hdr[2].text = "Year", "Revenue", "Growth"

for year, rev, growth in [("FY22", "10.2M", "-"), ("FY23", "12.5M", "22.5%")]:
    cells = table.add_row().cells
    cells[0].text, cells[1].text, cells[2].text = year, rev, growth

p = doc.add_paragraph("Revenue grew mainly due to new customers.")
p.runs[0].font.size = Pt(11)
doc.save("fdd_report.docx")
```

Also know: **python-pptx** for PowerPoint (FDD output is often Excel + Word + PPT).

---

## 7. Connecting to Accounting System APIs

- **QuickBooks Online, Xero, NetSuite, Sage Intacct** all have APIs.
- Login usually uses **OAuth 2.0**: the user clicks "Connect QuickBooks" → logs in on Intuit → your app gets an **access token** (short life) + **refresh token**.
- Things to handle: **pagination** (data comes in pages), **rate limits** (retry with backoff), **token refresh**, network errors.

```python
import time, requests

def get_with_retry(url, headers, tries=3):
    for i in range(tries):
        r = requests.get(url, headers=headers, timeout=30)
        if r.status_code == 429:          # rate limited
            time.sleep(2 ** i)            # 1s, 2s, 4s  (exponential backoff)
            continue
        r.raise_for_status()
        return r.json()
    raise Exception("Too many retries")
```

---

## 8. AI / LLM APIs (bonus in the JD)

### Basics
- **LLM** = Large Language Model (Claude by Anthropic, GPT by OpenAI).
- You send a **prompt** (text) through an API → get a **response**.
- **Tokens** = pieces of words; you pay per token; there's a max limit (context window).
- **Temperature** = randomness. For finance, use **low (0 to 0.2)** for consistent answers.

### Example (Anthropic Claude, Python)
```python
import anthropic, json

client = anthropic.Anthropic()   # API key from environment variable ANTHROPIC_API_KEY

msg = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=300,
    temperature=0,
    system="You map accounting accounts to standard categories. Reply only in JSON.",
    messages=[{
        "role": "user",
        "content": 'Account: "6105 Rent - Warehouse". Categories: Rent, Payroll, Revenue, COGS, Other. '
                   'Return {"category": ..., "confidence": 0-1}'
    }],
)
result = json.loads(msg.content[0].text)
```

### Example (OpenAI, JavaScript)
```js
import OpenAI from "openai";
const client = new OpenAI();   // uses OPENAI_API_KEY
const res = await client.chat.completions.create({
  model: "gpt-4o-mini",
  temperature: 0,
  messages: [{ role: "user", content: "Summarize this revenue trend: ..." }],
});
console.log(res.choices[0].message.content);
```

### Good uses of AI in FDD
- Suggest account mappings.
- Write a first draft of report commentary ("Revenue grew 22% due to...").
- Flag unusual transactions.
- Read PDFs (leases, contracts) to fill a rent schedule.

### Important rules (say these! They show maturity)
1. **AI suggests, humans approve.** Final numbers come from **deterministic code**, not the AI.
2. **AI can be wrong ("hallucinate")**, so validate its output (check JSON shape, allowed categories).
3. **Keep client data safe:** don't send more data than needed; use providers/settings that don't train on your data.
4. Log AI suggestions + who accepted them in the **audit trail**.
5. Words: **prompt engineering**, **structured output / JSON mode**, **RAG** (giving the AI relevant documents to read before answering), **embeddings** (turning text into numbers to find similar items).

---

## 9. Quick Q&A

- **Batch vs streaming?** Batch = process a whole file at once (GL import). Streaming = process data as it arrives.
- **How to handle a 1 GB file?** Read in chunks, process in a background job, save in batches, show progress.
- **What is data validation?** Checking data is correct before saving (types, required fields, totals).
- **What is idempotent import?** Importing the same file twice doesn't duplicate data (check file hash / unique keys).

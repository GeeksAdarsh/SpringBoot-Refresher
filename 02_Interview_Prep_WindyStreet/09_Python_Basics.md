# Python Basics (Easy Notes)

Why Python? The JD lists Python as a language option, plus **FastAPI**, **openpyxl** and **python-docx** (all Python). The data side of this job will likely use Python. You know Java, so learn the Python basics to show you can switch easily.

---

## 1. Java vs Python Quick Map

| Java | Python |
|---|---|
| `int x = 5;` | `x = 5` |
| `String s = "hi";` | `s = "hi"` |
| `List<Integer> l = new ArrayList<>();` | `l = []` |
| `Map<String,Integer> m = new HashMap<>();` | `m = {}` |
| `for (int i=0;i<n;i++)` | `for i in range(n):` |
| `if (a && b)` | `if a and b:` |
| `null` | `None` |
| `System.out.println()` | `print()` |
| `{ }` blocks | Indentation (4 spaces) |
| `try/catch` | `try/except` |

---

## 2. Core Data Types

```python
nums = [1, 2, 3]              # list (can change)
point = (10, 20)              # tuple (cannot change)
user = {"name": "A", "age": 24}  # dict (key-value)
tags = {"rent", "payroll"}    # set (unique values)

nums.append(4)
user["city"] = "Gurugram"
user.get("phone", "N/A")      # safe read with default
```

---

## 3. Loops, Comprehensions

```python
for i, val in enumerate(nums):
    print(i, val)

for key, value in user.items():
    print(key, value)

squares = [x * x for x in nums]                 # list comprehension
positives = [x for x in nums if x > 0]
totals = {acc: 0 for acc in ["Rent", "Payroll"]}  # dict comprehension
```

---

## 4. Functions

```python
def total(amounts, start=0):
    return sum(amounts) + start

def summary(*args, **kwargs):     # any number of args / named args
    print(args, kwargs)

double = lambda x: x * 2
```

---

## 5. Classes

```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class LedgerRow:
    account: str
    debit: Decimal
    credit: Decimal

    def net(self) -> Decimal:
        return self.debit - self.credit

row = LedgerRow("Rent", Decimal("100.00"), Decimal("0"))
print(row.net())
```

**Use `Decimal` for money, not float.** (`0.1 + 0.2 != 0.3` in float.)

---

## 6. Error Handling and Files

```python
try:
    amount = Decimal(value)
except Exception as e:
    print("Bad amount:", value, e)
finally:
    print("done")

with open("data.csv") as f:          # "with" closes the file automatically
    for line in f:
        print(line.strip())

import csv
with open("gl.csv", newline="") as f:
    for row in csv.DictReader(f):
        print(row["Account"], row["Debit"])
```

---

## 7. Useful Built-ins

```python
from collections import defaultdict, Counter

totals = defaultdict(Decimal)
for r in rows:
    totals[r.account] += r.debit

Counter(["a", "b", "a"])          # {'a': 2, 'b': 1}
sorted(rows, key=lambda r: r.debit, reverse=True)
any(r.debit < 0 for r in rows)
zip([1, 2], ["a", "b"])           # (1,'a'), (2,'b')
```

---

## 8. Common Python Interview Questions

| Question | Answer |
|---|---|
| List vs tuple? | List can change, tuple cannot (faster, can be a dict key). |
| `is` vs `==`? | `==` same value; `is` same object in memory. |
| What is a decorator? | A function that wraps another function to add behavior (`@app.get`, `@dataclass`). |
| What is a generator? | Function with `yield` that gives values one by one; saves memory for big files. |
| Mutable default argument bug? | `def f(x=[])` shares the same list each call. Use `x=None` then `x = x or []`. |
| What is GIL? | Global Interpreter Lock: only one thread runs Python code at a time; use multiprocessing for CPU work, async for I/O. |
| `*args`, `**kwargs`? | Any number of positional / named arguments. |
| Virtual environment? | `python -m venv venv`: separate packages per project. `pip install -r requirements.txt`. |

### Generator example (good for big GL files)
```python
def read_rows(path):
    with open(path) as f:
        for line in f:
            yield line.strip().split(",")

for row in read_rows("huge_gl.csv"):   # reads one line at a time, low memory
    process(row)
```

---

## 9. Mini Practice

1. Read a CSV GL file and print total debit and credit.
2. Group by account and write the result to Excel using openpyxl.
3. Flag rows where the amount is above 3× the average for that account (unusual entries).

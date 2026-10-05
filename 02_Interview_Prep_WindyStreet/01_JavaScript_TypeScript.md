# JavaScript and TypeScript (Easy Notes)

The JD says: *"Solid fundamentals in at least one modern language (JavaScript/TypeScript, Python, or Java)"* and *"React"*. So JavaScript is very important.

---

## 1. var, let, const

| Keyword | Can change value? | Scope | Use it? |
|---|---|---|---|
| `var` | Yes | Function scope | Avoid (old style) |
| `let` | Yes | Block scope `{ }` | When value will change |
| `const` | No (cannot reassign) | Block scope | Default choice |

```js
const user = { name: "Adarsh" };
user.name = "AK";   // OK: changing inside the object is allowed
// user = {};        // Error: cannot reassign a const
```

**Interview line:** "I use `const` by default and `let` only when I need to reassign. I don't use `var` because it is function-scoped and gets hoisted, which causes bugs."

---

## 2. Data Types

- **Primitive:** `string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`
- **Reference:** `object`, `array`, `function`

`null` = "empty on purpose". `undefined` = "value not given yet".

### `==` vs `===`
- `==` changes types before comparing: `"5" == 5` is `true`
- `===` checks value **and** type: `"5" === 5` is `false`
- **Always use `===`.**

---

## 3. Functions and Arrow Functions

```js
function add(a, b) { return a + b; }       // normal function
const add2 = (a, b) => a + b;               // arrow function
```

Difference: an arrow function does **not** have its own `this`. It uses the `this` from outside.

---

## 4. Hoisting (common question)

JS moves declarations to the top before running code.
- `var` is hoisted with value `undefined`.
- `let`/`const` are hoisted but you cannot use them before the line (this is called the "temporal dead zone").
- Normal functions are fully hoisted (you can call them before writing them).

---

## 5. Closures (very common question)

A closure is when an inner function **remembers** variables from its outer function, even after the outer function has finished.

```js
function counter() {
  let count = 0;
  return () => { count++; return count; };
}
const c = counter();
c(); // 1
c(); // 2  ← it remembers "count"
```

**Use:** private data, React hooks use this idea, event handlers.

---

## 6. Array Methods (must know: you will use these in React every day)

```js
const amounts = [100, 250, -50, 400];

amounts.map(x => x * 2);              // [200, 500, -100, 800]  change every item
amounts.filter(x => x > 0);           // [100, 250, 400]        keep some items
amounts.reduce((sum, x) => sum + x, 0); // 700                  make one value
amounts.find(x => x > 200);           // 250                    first match
amounts.some(x => x < 0);             // true                   any match?
amounts.every(x => x > 0);            // false                  all match?
[...amounts].sort((a, b) => a - b);   // sort numbers (copy first!)
```

**Accounting example (good for this company):**
```js
const ledger = [
  { account: "Rent", amount: 5000 },
  { account: "Salary", amount: 20000 },
  { account: "Rent", amount: 5000 },
];

// Total per account
const totals = ledger.reduce((acc, row) => {
  acc[row.account] = (acc[row.account] || 0) + row.amount;
  return acc;
}, {});
// { Rent: 10000, Salary: 20000 }
```

---

## 7. Spread, Rest, Destructuring

```js
const a = [1, 2]; const b = [...a, 3];           // spread: [1,2,3]
const user = { name: "A", age: 24 };
const updated = { ...user, age: 25 };            // copy + change
const { name, age } = user;                      // destructuring
function sum(...nums) { return nums.reduce((s, n) => s + n, 0); } // rest
```

---

## 8. Async JavaScript (very important)

JS runs on **one thread**. Slow work (API calls, timers) runs in the background, and the **event loop** brings the result back when ready.

### Callback → Promise → async/await

```js
// Promise
fetch("/api/accounts")
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));

// async/await (cleaner, same thing)
async function loadAccounts() {
  try {
    const res = await fetch("/api/accounts");
    if (!res.ok) throw new Error("Request failed: " + res.status);
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

### Run many requests at once
```js
const [accounts, ledger] = await Promise.all([fetch("/a"), fetch("/b")]);
```
- `Promise.all`: fails if any one fails.
- `Promise.allSettled`: waits for all, tells you which passed and which failed.

### Event loop in simple words
1. Normal code runs first (call stack).
2. Then **microtasks** (Promises `.then`).
3. Then **macrotasks** (`setTimeout`).

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// Output: 1, 4, 3, 2
```

---

## 9. `this` keyword (short)

- Inside an object method: `this` = that object.
- Inside an arrow function: `this` = whatever `this` was outside.
- Alone in a normal function (strict mode): `undefined`.

---

## 10. Debounce (asked often, used in search boxes)

Wait until the user stops typing, then call the API once.

```js
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
const search = debounce((text) => console.log("Search:", text), 300);
```

---

## 11. Shallow copy vs Deep copy

- Shallow: `{...obj}` copies only the top level. Nested objects are still shared.
- Deep: `structuredClone(obj)` copies everything.

---

## 12. TypeScript (the JD says JavaScript/TypeScript)

TypeScript = JavaScript **+ types**. It finds mistakes **before** the code runs.

```ts
type LedgerRow = {
  id: number;
  account: string;
  amount: number;
  date: string;
  memo?: string;          // ? means optional
};

function total(rows: LedgerRow[]): number {
  return rows.reduce((s, r) => s + r.amount, 0);
}
```

### Key words to know

| Word | Meaning |
|---|---|
| `type` / `interface` | Describe the shape of an object. Interface can be extended/merged; type can also do unions. |
| Union `\|` | `type Status = "draft" \| "review" \| "final";` (one of these) |
| Generics `<T>` | Reusable types: `function first<T>(arr: T[]): T { return arr[0]; }` |
| `any` vs `unknown` | `any` turns off checking (avoid). `unknown` forces you to check before using. |
| `Partial<T>` | All fields optional (good for "update" forms) |
| `Pick<T, K>` / `Omit<T, K>` | Take or remove some fields |
| `Record<K, V>` | Object with keys K and values V: `Record<string, number>` |
| Optional chaining `?.` | `user?.address?.city` (no crash if missing) |
| Nullish `??` | `value ?? 0` (use 0 only if value is null/undefined) |

**Why TypeScript for a finance app?** "Money data must be correct. TypeScript catches wrong field names and wrong types early, so fewer bugs reach production."

---

## 13. Money and Numbers (important for a finance company!)

```js
0.1 + 0.2   // 0.30000000000000004  ← floating point problem
```

**How to handle money:**
- Store amounts as **integers in cents/paise** (e.g. 1050 = $10.50), or
- Use a decimal library (`decimal.js`, `big.js`), and on the backend use `BigDecimal` (Java) or `DECIMAL` in SQL, or `Decimal` in Python.
- Round only at the end, when showing the value.

**Say this in the interview. It shows you think about finance data correctly.**

---

## 14. Quick Q&A

- **What is the DOM?** The page as a tree of objects that JS can change.
- **Event bubbling?** An event on a child goes up to its parents. `e.stopPropagation()` stops it.
- **localStorage vs sessionStorage vs cookies?** localStorage stays after closing the browser; sessionStorage is cleared when the tab closes; cookies are sent to the server with every request.
- **What is CORS?** The browser blocks calls to a different domain unless the server allows it with headers. Fix on the server (e.g. `@CrossOrigin` or a CORS config in Spring Boot).
- **What is JSON?** A text format for sending data: `{"name":"A","amount":100}`.

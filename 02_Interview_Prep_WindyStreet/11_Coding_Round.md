# Coding Round (Easy Notes)

For 6 to 18 month roles, coding questions are usually **easy to medium**: arrays, strings, hash maps, plus sometimes a small **practical task** (build an API or a React component).

Solutions are in **JavaScript** (you can write the same in Java).

---

## 1. How to Answer Any Coding Question

1. **Repeat the question** in your words and ask about edge cases (empty input? negatives? duplicates?).
2. Say a **simple (brute force) idea** first.
3. Improve it (usually with a **hash map** or **two pointers**).
4. Write clean code with good names.
5. **Test** with a small example out loud.
6. Say the **time and space complexity**.

### Big-O cheat
| Big-O | Example |
|---|---|
| O(1) | Read from a map |
| O(log n) | Binary search |
| O(n) | One loop |
| O(n log n) | Sorting |
| O(n²) | Loop inside a loop |

---

## 2. Must-Practice Problems

### Two Sum (hash map)
```js
function twoSum(nums, target) {
  const seen = new Map();
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
// O(n) time, O(n) space
```

### Reverse a string / Palindrome
```js
const reverse = s => s.split("").reverse().join("");
const isPalindrome = s => {
  const clean = s.toLowerCase().replace(/[^a-z0-9]/g, "");
  return clean === reverse(clean);
};
```

### Find duplicates
```js
function hasDuplicate(arr) {
  return new Set(arr).size !== arr.length;
}
```

### Count frequency of characters / words
```js
function frequency(str) {
  const count = {};
  for (const ch of str) count[ch] = (count[ch] || 0) + 1;
  return count;
}
```

### Anagram check
```js
const isAnagram = (a, b) =>
  a.length === b.length && [...a].sort().join("") === [...b].sort().join("");
```

### Max subarray sum (Kadane's)
```js
function maxSubArray(nums) {
  let best = nums[0], cur = nums[0];
  for (let i = 1; i < nums.length; i++) {
    cur = Math.max(nums[i], cur + nums[i]);
    best = Math.max(best, cur);
  }
  return best;
}
```

### Valid parentheses (stack)
```js
function isValid(s) {
  const stack = [], pairs = { ")": "(", "]": "[", "}": "{" };
  for (const c of s) {
    if ("([{".includes(c)) stack.push(c);
    else if (stack.pop() !== pairs[c]) return false;
  }
  return stack.length === 0;
}
```

### Merge two sorted arrays
```js
function merge(a, b) {
  const res = []; let i = 0, j = 0;
  while (i < a.length && j < b.length) res.push(a[i] <= b[j] ? a[i++] : b[j++]);
  return res.concat(a.slice(i), b.slice(j));
}
```

### Binary search
```js
function binarySearch(arr, target) {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {
    const mid = Math.floor((lo + hi) / 2);
    if (arr[mid] === target) return mid;
    arr[mid] < target ? (lo = mid + 1) : (hi = mid - 1);
  }
  return -1;
}
```

### Flatten nested array
```js
const flatten = arr => arr.reduce((acc, x) => acc.concat(Array.isArray(x) ? flatten(x) : x), []);
// or simply: arr.flat(Infinity)
```

### Group by (very practical)
```js
function groupBy(rows, key) {
  return rows.reduce((acc, r) => {
    (acc[r[key]] ||= []).push(r);
    return acc;
  }, {});
}
```

### Longest substring without repeating characters (sliding window)
```js
function longestUnique(s) {
  const last = new Map(); let start = 0, best = 0;
  for (let i = 0; i < s.length; i++) {
    if (last.has(s[i]) && last.get(s[i]) >= start) start = last.get(s[i]) + 1;
    last.set(s[i], i);
    best = Math.max(best, i - start + 1);
  }
  return best;
}
```

---

## 3. Finance-Style Practical Problems (they may give something like this!)

### A. Is the ledger balanced?
```js
function isBalanced(entries) {
  // work in cents to avoid floating point errors
  const toCents = n => Math.round(n * 100);
  const debit = entries.reduce((s, e) => s + toCents(e.debit), 0);
  const credit = entries.reduce((s, e) => s + toCents(e.credit), 0);
  return debit === credit;
}
```

### B. Trial balance: net balance per account
```js
function trialBalance(entries) {
  const tb = {};
  for (const { account, debit = 0, credit = 0 } of entries) {
    tb[account] = (tb[account] || 0) + debit - credit;
  }
  return tb;
}
```

### C. Map accounts to categories using keyword rules
```js
const RULES = [
  { category: "Rent", words: ["rent", "lease"] },
  { category: "Payroll", words: ["salary", "wage", "payroll"] },
  { category: "Revenue", words: ["sales", "revenue", "income"] },
];

function mapAccount(name) {
  const lower = name.toLowerCase();
  const rule = RULES.find(r => r.words.some(w => lower.includes(w)));
  return rule ? rule.category : "Unmapped";
}
```

### D. Monthly totals
```js
function monthlyTotals(entries) {
  return entries.reduce((acc, e) => {
    const month = e.date.slice(0, 7);         // "2024-03"
    acc[month] = (acc[month] || 0) + e.amount;
    return acc;
  }, {});
}
```

### E. Find duplicate transactions
```js
function findDuplicates(entries) {
  const seen = new Set(), dups = [];
  for (const e of entries) {
    const key = `${e.date}|${e.account}|${e.amount}`;
    seen.has(key) ? dups.push(e) : seen.add(key);
  }
  return dups;
}
```

### F. Reconcile two lists (GL vs bank)
```js
function reconcile(gl, bank) {
  const bankKeys = new Set(bank.map(b => `${b.date}|${b.amount}`));
  return gl.filter(g => !bankKeys.has(`${g.date}|${g.amount}`)); // in GL but not in bank
}
```

---

## 4. Practical / Take-Home Task Tips

If they give "build a small app" (e.g. upload CSV, show table, totals):
- Keep it **simple and working** first, then improve.
- Clean folder structure, small components / layered backend.
- Handle **loading, errors, empty states**.
- Validate input.
- Add a few **unit tests**.
- Write a short **README**: how to run, decisions you made, what you'd improve with more time.
- Make clean Git commits (they may check history!).

---

## 5. Where to Practise

- **LeetCode:** Easy + Medium, topics: Array, String, Hash Table, Two Pointers, Sliding Window, Stack. Do the "Top Interview 150" easy ones.
- **SQL:** LeetCode SQL 50, HackerRank SQL.
- **JS:** build small things (debounce, groupBy, fetch with retry).

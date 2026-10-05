# React (Easy Notes)

The JD says: *"Build front-end features using React, translating UI/UX designs into responsive, functional interfaces."*

The FDD tool will have lots of **tables, forms, filters, uploads, and dashboards**. So focus on those.

---

## 1. What is React?

- A JavaScript **library** for building UIs from small pieces called **components**.
- It uses a **Virtual DOM**: React keeps a copy of the page in memory. When data changes, it compares the old copy with the new one (this is called "diffing") and updates only the parts that changed. This is fast.
- **One-way data flow:** data goes from parent → child through **props**.

---

## 2. Components, Props, State

```jsx
function AccountRow({ name, amount }) {       // props come from parent
  return <tr><td>{name}</td><td>{amount}</td></tr>;
}

function Counter() {
  const [count, setCount] = useState(0);      // state lives inside
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

| Props | State |
|---|---|
| Given by parent | Owned by the component |
| Read-only | Can change with `setState` |
| Like function arguments | Like the component's memory |

When **state or props change**, the component **re-renders**.

---

## 3. Hooks (most asked)

| Hook | What it does | Example use |
|---|---|---|
| `useState` | Store a value that changes | Form input, toggle |
| `useEffect` | Run code after render (side effects) | API call, timers |
| `useContext` | Read shared data without passing props | Logged-in user, theme |
| `useRef` | Keep a value without re-render / access a DOM element | Focus input, store timer id |
| `useMemo` | Remember a **calculated value** | Total of 10,000 ledger rows |
| `useCallback` | Remember a **function** | Pass stable function to child |
| `useReducer` | Complex state with actions | Multi-step form |

### useEffect rules
```jsx
useEffect(() => { ... });            // runs after EVERY render
useEffect(() => { ... }, []);        // runs ONCE (on first load)
useEffect(() => { ... }, [id]);      // runs when "id" changes
useEffect(() => {
  const t = setInterval(tick, 1000);
  return () => clearInterval(t);     // cleanup: runs before unmount
}, []);
```

### Rules of hooks
1. Call hooks only at the **top level** (not inside if/loops).
2. Call hooks only inside **React components or custom hooks**.

---

## 4. Fetching Data From an API (you WILL be asked)

```jsx
function LedgerTable() {
  const [rows, setRows] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    axios.get("/api/ledger")
      .then(res => setRows(res.data))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <table>
      <tbody>
        {rows.map(r => (
          <tr key={r.id}><td>{r.account}</td><td>{r.amount}</td></tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Always handle 3 states: loading, error, success.** Saying this impresses interviewers.

In real projects, people often use **React Query (TanStack Query)** for this. It handles caching, loading and refetching for you.

---

## 5. Keys in Lists

- `key` helps React know which item changed.
- Use a **unique id** (`row.id`), **not the array index** (index breaks when you sort or delete).

---

## 6. Controlled Forms

```jsx
function MappingForm() {
  const [form, setForm] = useState({ account: "", category: "" });

  const handleChange = (e) =>
    setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.account) return alert("Account is required");
    axios.post("/api/mappings", form);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="account" value={form.account} onChange={handleChange} />
      <input name="category" value={form.category} onChange={handleChange} />
      <button type="submit">Save</button>
    </form>
  );
}
```
- **Controlled** = React state holds the value (most common).
- **Uncontrolled** = the DOM holds the value, read it using `useRef`.
- Big forms: **React Hook Form** + **Zod/Yup** for validation.

---

## 7. Lifting State Up and Context

- Two sibling components need the same data → move the state to their **common parent**.
- Many components deep down need it → use **Context** (or Redux/Zustand).
- **Prop drilling** = passing props through many layers that don't use them. Context solves it.

In CodeCampus, `AuthContext` shares the logged-in user and token across all pages. **Use this as your example.**

---

## 8. Performance (asked for 6 to 18 month roles)

- `React.memo(Component)`: skip re-render if props are the same.
- `useMemo`: don't recalculate heavy things (e.g. totals of a big ledger) on every render.
- `useCallback`: keep the same function between renders.
- **Big tables (10,000+ rows):** use **pagination** or **virtualization** (`react-window`, AG Grid) so only visible rows are drawn.
- **Lazy loading:** `React.lazy(() => import("./Reports"))` + `<Suspense>` loads pages only when needed.
- **Debounce** search inputs.

**FDD apps have huge general ledger tables. Talk about pagination/virtualization. This is a strong point.**

---

## 9. React Router

```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Dashboard />} />
    <Route path="/engagements/:id" element={<Engagement />} />
  </Routes>
</BrowserRouter>
// Inside a component:
const { id } = useParams();
const navigate = useNavigate();
```

**Protected route:** if the user is not logged in, redirect to `/login`.

---

## 10. File Upload (likely in an FDD app: users upload GL Excel/CSV)

```jsx
function UploadGL() {
  const handleUpload = async (e) => {
    const file = e.target.files[0];
    const data = new FormData();
    data.append("file", file);
    await axios.post("/api/gl/upload", data);
  };
  return <input type="file" accept=".xlsx,.csv" onChange={handleUpload} />;
}
```

---

## 11. Styling and Responsive Design

- CSS options: plain CSS, CSS Modules, **Tailwind**, styled-components, MUI / Ant Design.
- Responsive: **Flexbox**, **CSS Grid**, **media queries**, relative units (`rem`, `%`).
- "Translating UI/UX designs" = turning a **Figma** design into components. Say: *"I break the design into reusable components first, then build from small to big."*

---

## 12. Common React Questions (short answers)

| Question | Answer |
|---|---|
| Class vs function components? | Function + hooks is the modern way. Class uses `this.state` and lifecycle methods. |
| Why is `setState` async? | React batches many updates together for speed. Use `setCount(prev => prev + 1)` when the new value depends on the old one. |
| What is reconciliation? | React comparing old and new virtual DOM to update only changes. |
| What is a custom hook? | Your own function starting with `use` that reuses hook logic, e.g. `useFetch(url)`. |
| What is an Error Boundary? | A component that catches crashes in its children and shows a fallback UI. |
| SSR vs CSR? | CSR: browser builds page (Vite/React). SSR: server builds HTML first (Next.js), better SEO and first load. |
| What is JSX? | HTML-like syntax inside JS; it becomes `React.createElement` calls. |
| Fragment? | `<>...</>` groups elements without adding an extra div. |

---

## 13. Custom Hook Example (good to write in an interview)

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    fetch(url)
      .then(r => r.json())
      .then(d => { if (!cancelled) setData(d); })
      .catch(e => { if (!cancelled) setError(e); })
      .finally(() => { if (!cancelled) setLoading(false); });
    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}
```

---

## 14. Practice Task (do this before the interview)

Build a small **"Ledger Viewer"**:
1. Show a table of rows: date, account, debit, credit.
2. Search box (debounced) to filter by account.
3. Dropdown to filter by month.
4. Show **total debit** and **total credit** at the bottom (use `useMemo`).
5. Show a red warning if total debit ≠ total credit (this is a "consistency check"!).

If you can build this, you can handle the React part of this job's interview.

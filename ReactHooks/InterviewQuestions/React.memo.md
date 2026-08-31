# `React.memo` — Interview Questions

[← Back to index](../README.md)

Legend: ⭐ Basic · 🔥 Important · 🧠 Deep dive

---

### ⭐ Q2. Does `React.memo` prevent re-rendering?

**Don't say:** "Yes."

**Say:**

It can allow React to skip rendering a component when its props haven't changed. However, the component can still render when its own state changes, or when context it consumes changes.

---

### 🔥 Q3. Why doesn't this work?

```jsx
const Child = React.memo(ChildComponent);

function Parent() {
  const user = {
    name: "Adrija"
  };
  return <Child user={user} />;
}
```

Because:

```
Parent renders
   ↓
new user object
   ↓
new reference
   ↓
props changed
   ↓
Child renders
```

---

### 🔥 Q4. How can you make it work?

Potentially:

```jsx
const user = useMemo(() => ({
  name: "Adrija"
}), []);
```

Then `<Child user={user} />` has a stable object reference.

But the better question is: **do we actually need to memoize this object?**
Don't add `useMemo` purely mechanically.

---

### 🔥 Q5. Why might `React.memo` not work with functions?

Because:

```jsx
const handleClick = () => {};
```

creates a new function reference on each render, so:

```
previous onClick !== next onClick
```

You may use `useCallback(...)` when stabilizing that reference provides a meaningful optimization.

---

### ⭐ Q6. `React.memo` vs `useMemo`?

| | `React.memo` | `useMemo` |
|---|---|---|
| Memoizes | Component rendering | Value |
| Input | Props | Dependencies |
| Used as | `memo(Component)` | `useMemo(fn, deps)` |
| Main purpose | Skip unnecessary component rendering | Avoid unnecessary recalculation / stabilize value |

---

### ⭐ Q7. `React.memo` vs `useCallback`?

| | `React.memo` | `useCallback` |
|---|---|---|
| Memoizes | Component | Function reference |
| Protects | Child component from prop-based re-render | Function identity |
| Typical combination | `memo` + `useCallback` | Passed to memoized child |

---

### 🔥 Q8. What happens here?

```jsx
const Child = React.memo(function Child({ count }) {
  console.log("Child");
  return <h1>{count}</h1>;
});

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Update
      </button>
      <Child count={10} />
    </>
  );
}
```

Click the button several times. Assuming no other relevant changes:

- `Parent` → renders every time
- `Child` → can skip rendering

because the `Child` `count` prop (`10`) remains the same.

---

### 🔥 Q9. What if `Child` has its own state?

```jsx
const Child = React.memo(function Child() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
});
```

Clicking the button:

```
Child state changes
      ↓
Child renders
```

`React.memo` doesn't prevent that.

---

### 🔥 Q10. What if `Child` uses Context?

If the consumed Context value changes, the component can update despite being memoized.

---

### 🧠 Q11. Is `React.memo` always beneficial?

**No.** You need to consider:

- How expensive is rendering?
- How frequently does it render?
- How often do props actually remain unchanged?
- How expensive are prop comparisons?
- Are prop references stable?

---

### 🧠 Q12. What does `React.memo` return?

It returns a **memoized component**.

Conceptually:

```jsx
const MemoizedChild = React.memo(Child);
```

Now `<MemoizedChild />` is the component you render.

---

### 🔥 Q13. Can a custom comparison function cause stale UI?

**Yes.** If:

```jsx
(prev, next) => true
```

is returned despite relevant props changing, React may skip a necessary render.

---

### 🧠 Q14. Why is immutable data important for `React.memo`?

Because memoization relies on comparing prop values/references.

If you mutate an existing object:

```jsx
user.name = "Rahul";
```

the reference can remain unchanged. React may therefore conclude:

```
same reference
      ↓
props appear unchanged
      ↓
skip
```

which can lead to **stale UI**.

---

[← Back to useCallback](useCallback.md) · [Back to index](../README.md) · [Next: useEffect →](useEffect.md)
# `useMemo` — Interview Questions

[← Back to index](../README.md)

Legend: ⭐ Basic · 🔥 Important · 🧠 Deep dive

---

### ⭐ Q1. What is `useMemo`?

**Strong answer:**

`useMemo` is a React Hook that memoizes the result of a calculation between renders. React can reuse the cached value when its dependencies haven't changed. It's primarily a performance optimization, and can also be useful for preserving referential stability of objects or arrays.

---

### ⭐ Q2. What does `useMemo` return?

The **result of the calculation** — not the function itself.

```jsx
const value = useMemo(() => {
  return 10 * 2;
}, []);
```

`value` is `20`.

---

### 🔥 Q3. Difference between `useMemo` and `useCallback`?

| Hook | Memoizes |
|---|---|
| `useMemo` | result / value |
| `useCallback` | function reference |

---

### 🔥 Q4. Difference between `useMemo` and `React.memo`?

| API | Memoizes |
|---|---|
| `React.memo` | a component, based on its props |
| `useMemo` | a calculated value |

---

### 🔥 Q5. Is `useMemo` always good for performance?

**No.** It has its own overhead.

If the calculation is cheap:

```jsx
const x = a + b;
```

wrapping it in `useMemo(...)` can make the code more complex without meaningful benefit.

---

### 🔥 Q6. Why does this `useMemo` recalculate every render?

```jsx
const options = {
  sort: "asc"
};

const result = useMemo(() => {
  return calculate(options);
}, [options]);
```

Because a **new `options` object** is created every render, so:

```
previous options !== current options
```

---

### ⭐ Q7. Can `useMemo` prevent a component from rendering?

**No.** It memoizes a value — `React.memo` is the component-level optimization.

However, a stable memoized value can help a memoized child skip rendering, because its props remain referentially equal.

---

### 🔥 Q8. Does `useMemo` guarantee that the calculation only runs once?

**No.** It's a performance optimization, not a permanent-cache guarantee.

Also, development **Strict Mode** can invoke calculation functions more than once to detect impurities.

---

### 🧠 Q9. Why should `useMemo` calculations be pure?

Because React may execute them during rendering, and may execute them more than once in development. A calculation should not cause external side effects.

---

### 🔥 Q10. What's wrong here?

```jsx
const sorted = useMemo(() => {
  return products.sort(compare);
}, [products]);
```

`sort()` **mutates** the original array.

**Fix:**

```jsx
const sorted = useMemo(() => {
  return [...products].sort(compare);
}, [products]);
```

---

### 🧠 Q11. When would `useMemo` help `React.memo`?

Suppose:

```jsx
const Child = React.memo(ChildComponent);
```

and `Parent` does:

```jsx
const data = products.filter(...);
```

Every `Parent` render creates a **new array**, so `Child` sees a new `data` reference.

With:

```jsx
const data = useMemo(
  () => products.filter(...),
  [products]
);
```

the reference can remain stable when `products` hasn't changed.

```
useMemo → stable data reference
              ↓
React.memo → can skip Child
```

> This is an excellent interview answer.

---

### 🧠 Q12. Can `useMemo` replace `useState`?

**No.**

If a value is independently mutable application state:

```jsx
const [count, setCount] = useState(0);
```

If a value is **derived** from other values:

```jsx
const total = useMemo(
  () => price * quantity,
  [price, quantity]
);
```

---

[← Back to index](../README.md) · [Next: useCallback →](useCallback.md)
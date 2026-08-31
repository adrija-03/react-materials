# `useCallback` — Interview Questions

[← Back to index](../README.md)

Legend: ⭐ Basic · 🔥 Important · 🧠 Deep dive

---

### ⭐ Q1. What is `useCallback`?

**Strong answer:**

`useCallback` is a React Hook that memoizes a function reference between renders. React returns the same function reference when its dependencies haven't changed. It's primarily useful when function identity matters — such as passing callbacks to memoized components, or using callbacks as dependencies.

---

### ⭐ Q2. Does `useCallback` memoize the result of a function?

**No.** It memoizes the **function reference**.

| Hook | Memoizes |
|---|---|
| `useMemo` | result |
| `useCallback` | function |

---

### 🔥 Q3. Why would `React.memo` and `useCallback` be used together?

Suppose:

```jsx
const Child = React.memo(ChildComponent);
```

If `Parent` creates:

```jsx
const handleClick = () => {};
```

on every render, `Child` receives a new function reference every time.

Using:

```jsx
const handleClick = useCallback(() => {}, []);
```

keeps the reference stable, allowing `React.memo` to skip the `Child` render when other props are also unchanged.

---

### 🔥 Q4. Is `useCallback` always necessary when passing a function to a child?

**No.** It matters when the function's identity affects something — for example, a memoized child.

For a normal child:

```jsx
<Child onClick={handleClick} />
```

there may be no meaningful benefit.

---

### 🔥 Q5. What's wrong with this when `count` changes?

```jsx
const handleClick = useCallback(() => {
  console.log(count);
}, []);
```

Potential **stale closure**.

**Fix:**

```jsx
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);
```

---

### 🔥 Q6. Can you remove `count` from the dependency array?

Sometimes — if you don't actually need to read it.

```jsx
const increment = useCallback(() => {
  setCount(prev => prev + 1);
}, []);
```

This is correct because the callback uses the **functional updater** rather than reading `count` directly.

---

### ⭐ Q7. What happens when a dependency changes?

```jsx
const fn = useCallback(() => {
  console.log(userId);
}, [userId]);
```

- `userId = 10 → 10` — the same reference can be returned.
- `userId = 10 → 20` — React needs a **new callback** that closes over the new `userId`.

---

### 🔥 Q8. Why doesn't this prevent the Child from rendering?

```jsx
const handleClick = useCallback(() => {}, []);

return <Child onClick={handleClick} />;
```

Because `useCallback` only stabilizes the function.

If `Child` isn't memoized, its parent-child rendering behavior hasn't been optimized by `React.memo`. And even with `React.memo`, another prop/state/context change can still cause `Child` to update.

---

### 🧠 Q9. What is referential equality?

For objects/functions, equality often depends on whether two values point to the same reference.

```jsx
const fn1 = () => {};
const fn2 = () => {};

fn1 === fn2; // false
```

With a stable callback:

```
render 1 → function A
render 2 → function A
```

the reference remains the same.

---

### 🧠 Q10. Can `useCallback` itself have a performance cost?

**Yes.** React has to track the callback and its dependencies.

Therefore, `useCallback` everywhere is **not automatically faster**.

---

### 🔥 Q11. What's the relationship between `useMemo` and `useCallback`?

Conceptually:

```jsx
useMemo(() => calculate(), [deps])   // returns a memoized calculation result
useCallback(() => calculate(), [deps]) // returns a memoized callback reference
```

You can think of `useCallback(fn, deps)` as roughly expressing:

```jsx
useMemo(() => fn, deps)
```

---

### 🧠 Q12. Does `useCallback` guarantee the function reference forever?

**No.** It is a performance optimization. React can discard memoized values in certain circumstances.

Don't use it as a semantic guarantee that an identity can never change.

---

[← Back to useMemo](useMemo.md) · [Back to index](../README.md)
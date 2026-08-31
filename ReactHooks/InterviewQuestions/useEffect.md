# `useEffect` — Interview Questions

[← Back to index](../README.md)

Legend: ⭐ Basic · 🔥 Important · 🧠 Deep dive

---

### ⭐ Q1. What is `useEffect`?

**Strong answer:**

`useEffect` is a React Hook used to synchronize a component with external systems. React runs the Effect after the component has rendered and committed, and it can optionally return a cleanup function that React runs before the Effect is re-run or when the component unmounts.

That's much better than: *"useEffect is used for side effects."*

---

### ⭐ Q2. When does `useEffect` run?

Depends on dependencies.

**No dependency array:**

```jsx
useEffect(() => {});
```

Runs after every committed render.

**Empty array:**

```jsx
useEffect(() => {}, []);
```

No reactive dependencies; it runs after the initial commit in the normal lifecycle model.

**Dependencies:**

```jsx
useEffect(() => {}, [count]);
```

Runs after the initial commit and when `count` changes.

---

### 🔥 Q3. What is the dependency array?

It tells React which reactive values the Effect depends on, so React can determine when the synchronization needs to be re-established.

---

### 🔥 Q4. Why shouldn't we use `useEffect` for derived state?

Because derived values can usually be calculated during rendering. Using an Effect introduces an unnecessary cycle:

```
render
  → effect
    → state update
      → render
```

---

### ⭐ Q5. What does the cleanup function do?

It stops or reverses whatever external synchronization the Effect established.

| Setup | Cleanup |
|---|---|
| `addEventListener` | `removeEventListener` |
| `setInterval` | `clearInterval` |
| `subscribe` | `unsubscribe` |
| `connect` | `disconnect` |

---

### 🔥 Q6. When does cleanup run?

Two important cases:

1. **Before the Effect is re-run** — when dependencies change.
2. **When the component unmounts** — when the component is removed.

---

### 🔥 Q7. Why can this cause an infinite loop?

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Because:

```
count changes
   ↓
Effect runs
   ↓
setCount
   ↓
count changes
   ↓
Effect runs
   ↓
...
```

---

### 🧠 Q8. Why does `useEffect` run after render?

Because React first needs to determine and commit the UI. Effects are used to synchronize with external systems after the render/commit process.

---

### 🔥 Q9. Why shouldn't I make the Effect callback `async`?

This:

```jsx
useEffect(async () => {
  ...
}, []);
```

makes the callback return a Promise. React expects the callback to return either **nothing** or a **cleanup function**.

So instead:

```jsx
useEffect(() => {
  async function fetchData() {
    ...
  }

  fetchData();
}, []);
```

---

### 🧠 Q10. What happens if an object is in the dependency array?

React compares the dependency using `Object.is`. For objects, this means reference identity matters.

```jsx
{} !== {}
```

because they are different references.

---

### 🔥 Q11. `useEffect` vs event handler?

A very good interview distinction:

An event handler responds to a specific user interaction such as a click or form submission. An Effect is used to synchronize the component with an external system as a consequence of rendering or reactive values changing.

```
Button click       → event handler
WebSocket connection → Effect
```

---

### 🧠 Q12. `useEffect` vs `useLayoutEffect`?

Simplified:

```
useEffect
→ normal external synchronization

useLayoutEffect
→ DOM measurement/layout work that must happen before paint
```

Use `useLayoutEffect` only when you actually need its timing.

---

### 🔥 Q13. Why can an Effect run twice in development?

Strict Mode intentionally performs an extra setup/cleanup cycle in development to expose Effects that don't correctly clean up or aren't resilient to being re-run.

---

### 🧠 Q14. Is `useEffect` guaranteed to run immediately after DOM mutation?

Don't oversimplify its timing.

The important conceptual distinction is that `useEffect` is a **passive Effect**, and React schedules it after the commit. Its exact relationship with browser paint and interaction-driven updates can depend on React's scheduling behavior.

> For interview purposes: don't treat `useEffect` as a generic "after DOM update" synchronous callback.

---

[← Back to React.memo](react-memo.md) · [Back to index](../README.md) · [Next: useState →](useState.md)
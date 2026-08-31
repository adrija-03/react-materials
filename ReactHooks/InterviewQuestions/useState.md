# `useState` — Interview Questions

[← Back to index](../README.md)

Legend: ⭐ Basic · 🔥 Important · 🧠 Deep dive

---

### ⭐ Q1. What is `useState`?

**Strong answer:**

`useState` is a React Hook that allows function components to maintain state across renders. It returns the current state value and a setter function that schedules a state update and can cause React to render the component again.

Don't simply say: *"It stores data."* That's incomplete.

---

### ⭐ Q2. Why doesn't changing a normal variable update the UI?

Because React doesn't track arbitrary local variables.

```jsx
let count = 0;
count++;
```

doesn't tell React "render this component again." State provides React with a **tracked update mechanism**.

---

### ⭐ Q3. Why doesn't state update immediately?

Because state updates are **scheduled** rather than directly mutating the current render's state variable. The current render retains its snapshot.

---

### ⭐ Q4. Why does this print `0`?

```jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(1);
  console.log(count);
}
```

**Answer:** `0` — because the handler is executing with the state snapshot from the current render. The update affects a subsequent render.

---

### 🔥 Q5. What happens here?

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

If `count = 0`, the resulting state is generally `1`, because all updates use the **same `count` snapshot**.

---

### 🔥 Q6. How do you make it `3`?

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

Result: `3`.

---

### ⭐ Q7. Why does React provide functional state updates?

When the next state depends on the previous state:

```jsx
setCount(prev => prev + 1);
```

React can apply the updater against the appropriate pending state.

---

### 🧠 Q8. Is `useState` synchronous or asynchronous?

This question is slightly misleading. Don't answer simply: *"useState is asynchronous."*

**A better answer:**

Calling the setter does not synchronously mutate the state variable in the current render. React schedules the update and processes it as part of its rendering/update system. The exact scheduling behavior depends on the execution context and React's batching mechanisms.

---

### 🔥 Q9. Why must Hooks not be called conditionally?

Because React relies on the **consistent order** of Hook calls to associate each Hook with its state across renders.

---

### 🧠 Q10. What does React internally associate state with?

For interview purposes, understand this conceptually:

```
Component instance
      ↓
    Fiber
      ↓
 Hook state
```

React maintains Hook-related state as part of its internal component/Fiber representation. You don't need to memorize implementation details like linked-list internals unless you're interviewing for a particularly deep React role.

---

### 🔥 Q11. What's the difference?

```jsx
useState(heavyFunction());
```

vs

```jsx
useState(() => heavyFunction());
```

The second passes an **initializer function**, allowing React to lazily initialize the state rather than evaluating the expensive computation as an ordinary expression on every render.

---

### ⭐ Q12. Can state contain an object?

Yes.

```jsx
const [user, setUser] = useState({
  name: "A",
  age: 24
});
```

But React doesn't automatically merge object state like class component `setState` historically did. So:

```jsx
setUser({
  name: "B"
});
```

loses `age`. You generally need:

```jsx
setUser(prev => ({
  ...prev,
  name: "B"
}));
```

---

### 🔥 Q13. Does calling `setState` always cause a DOM update?

**No.** It can schedule React work, but if the resulting state is considered unchanged, React may bail out of unnecessary work. And even when React renders, reconciliation determines whether actual DOM changes are required.

This distinction is important:

```
state update
    ≠
DOM update
```

---

### 🧠 Q14. Why shouldn't we mutate state?

Because React relies heavily on immutable updates and reference/value comparisons to determine what changed. Mutation can make changes difficult to detect and causes bugs with rendering, memoization, and predictable state management.

---

[← Back to useEffect](useEffect.md) · [Back to index](../README.md)
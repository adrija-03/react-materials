# React Context API — Notes & Interview Prep

## 1. What Problem Does It Solve?

React data normally flows one way: parent → child, via props. When deeply nested
components need data owned by a far-off ancestor, you end up **prop drilling** —
passing props through every intermediate component that doesn't actually use them,
just to relay them downward.

**Context API** lets you skip the relay. A component higher up "provides" a value;
any descendant, no matter how deep, can "consume" it directly.

```
Without Context:                    With Context:
App                                  App (Provider)
 └─ Layout (passes prop)              └─ Layout (doesn't touch it)
     └─ Sidebar (passes prop)             └─ Sidebar (doesn't touch it)
         └─ UserCard (uses prop)              └─ UserCard (useContext)
```

---

## 2. The Three Building Blocks

### a) `createContext`
Creates the Context object and an optional default value (used only when a
component consumes the context **without** any Provider above it in the tree).

```js
import { createContext } from "react";

export const ThemeContext = createContext({
  themeMode: "light",
  toggleTheme: () => {},
});
```

### b) `Provider`
A component that "broadcasts" a value to every descendant. Any component inside
it can read that value — regardless of nesting depth.

```jsx
<ThemeContext.Provider value={{ themeMode, toggleTheme }}>
  <App />
</ThemeContext.Provider>
```

> Tip: aliasing `ThemeContext.Provider` as `ThemeProvider` (`export const ThemeProvider = ThemeContext.Provider`) is a common convention purely for readability.

### c) `useContext` (the Consumer, hook form)
Reads the nearest matching Provider's value.

```js
import { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

const { themeMode, toggleTheme } = useContext(ThemeContext);
```

**Best practice:** wrap this in a custom hook so components never import
`useContext` + the raw context directly:

```js
export default function useTheme() {
  return useContext(ThemeContext);
}
```

---

## 3. Full Minimal Example

```js
// ThemeContext.js
import { createContext, useContext } from "react";

export const ThemeContext = createContext({
  themeMode: "light",
  toggleTheme: () => {},
});

export const ThemeProvider = ThemeContext.Provider;

export function useTheme() {
  return useContext(ThemeContext);
}
```

```jsx
// App.jsx
import { useState } from "react";
import { ThemeProvider } from "./ThemeContext";
import Toolbar from "./Toolbar";

function App() {
  const [themeMode, setThemeMode] = useState("light");
  const toggleTheme = () =>
    setThemeMode((prev) => (prev === "light" ? "dark" : "light"));

  return (
    <ThemeProvider value={{ themeMode, toggleTheme }}>
      <Toolbar />
    </ThemeProvider>
  );
}
```

```jsx
// Toolbar.jsx → ThemeButton.jsx (deeply nested, no props passed through Toolbar)
import { useTheme } from "./ThemeContext";

function ThemeButton() {
  const { themeMode, toggleTheme } = useTheme();
  return <button onClick={toggleTheme}>Current: {themeMode}</button>;
}
```

---

## 4. When to Use Context (and When Not To)

**Good fits:**
- Theme (light/dark)
- Authenticated user / auth state
- Locale / i18n language
- App-wide settings, feature flags
- Data needed by many components at very different nesting depths

**Not a good fit:**
- Replacing all prop passing "just because" — if only 1–2 levels deep, props are simpler and more explicit
- High-frequency updates (e.g. mouse position, animation frames, form field on every keystroke) — causes broad re-renders (see performance section)
- General app-wide state management with complex update logic, async flows, caching, etc. — libraries like Redux, Zustand, Jotai, or React Query are often better suited

---

## 5. Multiple Contexts

You can nest multiple providers — order doesn't usually matter unless one depends on another.

```jsx
<AuthProvider value={authValue}>
  <ThemeProvider value={themeValue}>
    <App />
  </ThemeProvider>
</AuthProvider>
```

For many providers, a small composition helper keeps `App.jsx` clean:

```jsx
function AppProviders({ children }) {
  return (
    <AuthProvider value={authValue}>
      <ThemeProvider value={themeValue}>{children}</ThemeProvider>
    </AuthProvider>
  );
}
```

---

## 6. Performance: The Part Everyone Gets Wrong

**Every component that consumes a context re-renders whenever the Provider's
`value` changes** — even if the component only cares about part of that value.

### Pitfall: creating a new object on every render

```jsx
// ❌ Bad — new object literal every render → every consumer re-renders every time
<ThemeProvider value={{ themeMode, toggleTheme }}>
```

Even if `themeMode` didn't change, React sees a **new object reference** and
re-renders all consumers.

### Fix: memoize the value

```jsx
const value = useMemo(() => ({ themeMode, toggleTheme }), [themeMode]);

<ThemeProvider value={value}>
```

### Fix: split contexts by concern

Instead of one giant context with everything, split frequently-changing state
from rarely-changing state (or state from dispatch functions) into separate
contexts, so consumers only re-render for what they actually use.

```js
const StateContext = createContext();
const DispatchContext = createContext(); // functions rarely change reference if using useReducer's dispatch
```

### Fix: `useReducer` + Context for complex state

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
<StateContext.Provider value={state}>
  <DispatchContext.Provider value={dispatch}>
    {children}
  </DispatchContext.Provider>
</StateContext.Provider>
```
`dispatch` from `useReducer` is stable across renders, so components that only
need to dispatch actions (not read state) won't re-render unnecessarily.

---

## 7. Common Mistakes

| Mistake | Why it's a problem |
|---|---|
| Forgetting to wrap the app in the Provider | Consumers silently fall back to the `createContext` default value |
| Passing a new object/array literal as `value` on every render | Causes unnecessary re-renders in every consumer |
| Using Context for very high-frequency updates | Can tank performance — every consumer re-renders on every update |
| One giant "God context" with unrelated state | Any small change re-renders everything; split into focused contexts |
| Calling `useContext` outside a Provider when no default is set | Returns `undefined`/default — can cause confusing bugs, not a hard error |
| Mutating context value directly instead of via state setter | React won't detect the change / won't re-render |

---

## 8. Interview Questions & Answers

### Conceptual

**Q1: What problem does Context API solve?**
A: Prop drilling — passing data through many intermediate components that don't
use it, just to get it to a deeply nested descendant.

**Q2: What are the three core pieces of the Context API?**
A: `createContext()` to create the context, a `Provider` component to supply a
value to the subtree, and `useContext()` (or the legacy `Context.Consumer`) to
read that value in a descendant.

**Q3: What happens if a component calls `useContext` but there's no Provider above it?**
A: It receives the default value passed to `createContext()`. No error is
thrown — this can hide bugs if you don't set a sensible default.

**Q4: Does Context replace Redux/state management libraries?**
A: Not entirely. Context is built for passing data down a tree without prop
drilling. It's not optimized for complex state logic, middleware, async flows,
or fine-grained update control the way libraries like Redux or Zustand are —
though `useReducer` + Context can cover many simpler cases.

**Q5: What's the difference between Context and prop drilling in terms of re-renders?**
A: With props, only the components you explicitly pass props to receive
updates. With Context, **every consumer** of a context re-renders when its
`value` reference changes — even if they only use part of the value.

### Practical / Code-based

**Q6: Why is this a performance problem?**
```jsx
<MyContext.Provider value={{ user, setUser }}>
```
A: A new object literal is created on every parent re-render, giving Context
consumers a new reference each time even if `user` hasn't actually changed —
triggering unnecessary re-renders. Fix with `useMemo`.

**Q7: How would you avoid unnecessary re-renders when using Context with `useReducer`?**
A: Split `state` and `dispatch` into two separate contexts. `dispatch` is
referentially stable, so components that only dispatch actions (and don't read
state) won't re-render when state changes.

**Q8: Can you use multiple contexts in one component?**
A: Yes — call `useContext` multiple times, once per context:
```js
const theme = useContext(ThemeContext);
const auth = useContext(AuthContext);
```

**Q9: What's the difference between `Context.Consumer` and `useContext`?**
A: `Context.Consumer` is the older render-prop pattern:
```jsx
<ThemeContext.Consumer>
  {(value) => <div>{value.themeMode}</div>}
</ThemeContext.Consumer>
```
`useContext` is the modern hook-based equivalent, cleaner and avoids extra
nesting. Functionally equivalent; `useContext` is now the standard approach.

**Q10: How do you type Context properly in TypeScript?**
A:
```ts
interface ThemeContextType {
  themeMode: "light" | "dark";
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme must be used within a ThemeProvider");
  return ctx;
}
```
This pattern also solves the "forgot to wrap in Provider" problem at Q3 — it
fails loudly instead of silently returning a default.

**Q11: Is Context a state management solution?**
A: No — Context is a **plumbing/distribution mechanism** for data, not a state
management system itself. The state itself is usually still managed with
`useState` or `useReducer`; Context just makes that state accessible without
prop drilling.

---

## 9. Quick Reference Cheat Sheet

```js
// 1. Create
const MyContext = createContext(defaultValue);

// 2. Provide
<MyContext.Provider value={someValue}>
  {children}
</MyContext.Provider>

// 3. Consume
const value = useContext(MyContext);
```

**Golden rules:**
- Memoize the `value` object with `useMemo` if it contains objects/functions
- Split contexts by concern / update frequency
- Use a custom hook (`useTheme`, `useAuth`, etc.) instead of exposing raw `useContext` calls
- Add a runtime check in the custom hook to catch "used outside Provider" bugs early (especially useful in TypeScript)
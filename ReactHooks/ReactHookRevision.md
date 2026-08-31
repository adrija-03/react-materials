# ⚛️ React Hooks — Interview Revision

A personal revision repo for common React interview questions on `useMemo` and `useCallback`, organized by difficulty so you can drill weak spots fast.

## 📚 Contents

| Topic | Questions | File |
|---|---|---|
| `useMemo` | 12 | [`docs/useMemo.md`](docs/useMemo.md) |
| `useCallback` | 12 | [`docs/useCallback.md`](docs/useCallback.md) |
| `React.memo` | 13 | [`docs/react-memo.md`](docs/react-memo.md) |
| `useEffect` | 14 | [`docs/useEffect.md`](docs/useEffect.md) |
| `useState` | 14 | [`docs/useState.md`](docs/useState.md) |

## 🏷️ Difficulty Legend

| Badge | Meaning |
|---|---|
| ⭐ | Basic — you should answer instantly |
| 🔥 | Important — comes up often, know it cold |
| 🧠 | Deep dive — separates strong candidates |

## ✅ How to use this repo

1. Read the question first, try to answer out loud before checking.
2. Re-derive the code examples yourself instead of just re-reading them.
3. Revisit anything marked 🧠 the day before an interview.
4. Add your own gotchas as you hit them in real code — this repo is meant to grow.

## 🔗 Quick Cheat Sheet

| | Memoizes | Returns |
|---|---|---|
| `useMemo` | a **value** | the computed result |
| `useCallback` | a **function reference** | the function itself |
| `React.memo` | a **component** | a wrapped component |

```jsx
// useCallback(fn, deps) is conceptually:
useMemo(() => fn, deps)
```

### Rendering vs. state vs. effects, at a glance

| Hook / API | Runs | Common trap |
|---|---|---|
| `useState` | On every render that reads it | Reading stale state right after calling the setter |
| `useEffect` | After render is committed | Making the callback `async`, or missing cleanup |
| `useMemo` | During render (only when deps change) | Memoizing a cheap calculation for no reason |
| `useCallback` | During render (only when deps change) | Assuming it alone prevents child re-renders |
| `React.memo` | Wraps a component's render decision | Forgetting it doesn't stop state/context-driven renders |

---
*Last updated: 2026-08-31*
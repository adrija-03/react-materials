Here's a clean, GitHub-ready version combining both into one well-structured Q&A:

---

## Q: What is the purpose of the `key` prop in React, and what happens if you use array indices as keys?

**A:**

The `key` prop helps React uniquely identify elements within a list during reconciliation. When a list re-renders, React uses keys to determine which items were added, removed, reordered, or updated — instead of re-rendering the entire list from scratch, it can match old elements to new ones and only update what actually changed.

```jsx
const items = [
  { id: 1, value: 'Apple' },
  { id: 2, value: 'Banana' },
];

function ItemList() {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.value}</li>
      ))}
    </ul>
  );
}
```

Here, `item.id` is a stable, unique identifier — React can reliably track each `<li>` across renders regardless of reordering.

**Why array indices are problematic as keys:**

```jsx
const items = ['Apple', 'Banana', 'Cherry'];
items.map((item, index) => <div key={index}>{item}</div>);
```

Indices work fine *only* if the list is static — never reordered, filtered, or has items inserted/removed. The moment the list changes order, the index no longer maps to the same logical item, but React still thinks it does (since the key looks unchanged from its perspective). This causes:

- **Incorrect DOM reuse** — React may reuse a DOM node for the wrong item, leading to stale UI (e.g., an input's value showing up next to the wrong label).
- **Broken component state** — if list items are stateful components, state can get silently attached to the wrong item after reordering.
- **Unnecessary re-renders** — React may fail to recognize that a specific item didn't actually change, or vice versa.

**Best practice:** Always use a stable, unique identifier (like a database ID) as the key. Only fall back to index-based keys when the list is guaranteed to be static and never reordered.
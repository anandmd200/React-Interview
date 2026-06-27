# React Machine Coding - Concise Interview Version

## 1. Pagination

```jsx
// App.jsx
import { useState, useMemo } from "react";

const ITEMS_PER_PAGE = 5;

const data = Array.from({ length: 53 }, (_, i) => ({ id: i + 1, name: `Item ${i + 1}` }));

export default function Pagination() {
  const [currentPage, setCurrentPage] = useState(1);
  const totalPages = Math.ceil(data.length / ITEMS_PER_PAGE);

  const currentData = useMemo(() => {
    const start = (currentPage - 1) * ITEMS_PER_PAGE;
    return data.slice(start, start + ITEMS_PER_PAGE);
  }, [currentPage]);

  const pageNumbers = useMemo(() => {
    const pages = [];
    for (let i = 1; i <= totalPages; i++) {
      if (
        i === 1 ||
        i === totalPages ||
        (i >= currentPage - 1 && i <= currentPage + 1)
      ) {
        pages.push(i);
      } else if (pages[pages.length - 1] !== "...") {
        pages.push("...");
      }
    }
    return pages;
  }, [currentPage, totalPages]);

  return (
    <div>
      <ul>
        {currentData.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>

      <div style={{ display: "flex", gap: 4 }}>
        <button onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
          Prev
        </button>

        {pageNumbers.map((page, i) =>
          page === "..." ? (
            <span key={i}>...</span>
          ) : (
            <button
              key={page}
              onClick={() => setCurrentPage(page)}
              disabled={page === currentPage}
              style={{ fontWeight: page === currentPage ? "bold" : "normal" }}
            >
              {page}
            </button>
          )
        )}

        <button onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
          Next
        </button>
      </div>

      <p> Page {currentPage} of {totalPages} </p>
    </div>
  );
}
```

---

## 2. Progress Bar

```jsx
// App.jsx
import { useState, useEffect, useRef } from "react";

// ---- Reusable Bar ----
function ProgressBar({ value = 0 }) {
  const clamped = Math.min(100, Math.max(0, value));
  return (
    <div style={{ background: "#eee", borderRadius: 8, height: 24, width: "100%" }}>
      <div
        style={{
          width: `${clamped}%`,
          height: "100%",
          background: clamped < 40 ? "red" : clamped < 70 ? "orange" : "green",
          borderRadius: 8,
          transition: "width 0.1s",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          color: "#fff",
          fontSize: 12,
        }}
      >
        {clamped.toFixed(0)}%
      </div>
    </div>
  );
}

// ---- Animated Controller ----
export default function App() {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState("idle"); // idle | running | paused
  const intervalRef = useRef(null);

  useEffect(() => {
    if (status === "running") {
      intervalRef.current = setInterval(() => {
        setProgress((prev) => {
          if (prev >= 100) {
            clearInterval(intervalRef.current);
            setStatus("idle");
            return 100;
          }
          return prev + 1;
        });
      }, 50);
    }
    return () => clearInterval(intervalRef.current);
  }, [status]);

  const handleStart = () => { setProgress(0); setStatus("running"); };
  const handlePause = () => setStatus("paused");
  const handleResume = () => setStatus("running");
  const handleReset = () => { setProgress(0); setStatus("idle"); };

  return (
    <div style={{ width: 400, padding: 20 }}>
      <ProgressBar value={progress} />
      <div style={{ marginTop: 10, display: "flex", gap: 8 }}>
        {status === "idle"   && <button onClick={handleStart}>Start</button>}
        {status === "running"&& <button onClick={handlePause}>Pause</button>}
        {status === "paused" && <button onClick={handleResume}>Resume</button>}
        <button onClick={handleReset}>Reset</button>
      </div>
    </div>
  );
}
```

---

## 3. Todo App

```jsx
// App.jsx
import { useReducer, useState } from "react";

// ---- Reducer ----
function reducer(state, action) {
  switch (action.type) {
    case "ADD":
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case "TOGGLE":
      return state.map((t) => t.id === action.id ? { ...t, done: !t.done } : t);
    case "DELETE":
      return state.filter((t) => t.id !== action.id);
    case "EDIT":
      return state.map((t) => t.id === action.id ? { ...t, text: action.text } : t);
    default:
      return state;
  }
}

// ---- TodoItem ----
function TodoItem({ todo, dispatch }) {
  const [editing, setEditing] = useState(false);
  const [text, setText] = useState(todo.text);

  const submitEdit = () => {
    if (text.trim()) dispatch({ type: "EDIT", id: todo.id, text: text.trim() });
    setEditing(false);
  };

  return (
    <li style={{ display: "flex", gap: 8, alignItems: "center", margin: "4px 0" }}>
      <input type="checkbox" checked={todo.done}
        onChange={() => dispatch({ type: "TOGGLE", id: todo.id })} />

      {editing ? (
        <>
          <input value={text} onChange={(e) => setText(e.target.value)}
            onKeyDown={(e) => { if (e.key === "Enter") submitEdit();
                                if (e.key === "Escape") setEditing(false); }}
            autoFocus />
          <button onClick={submitEdit}>Save</button>
        </>
      ) : (
        <>
          <span style={{ textDecoration: todo.done ? "line-through" : "none" }}>
            {todo.text}
          </span>
          <button onClick={() => setEditing(true)}>Edit</button>
        </>
      )}

      <button onClick={() => dispatch({ type: "DELETE", id: todo.id })}>Del</button>
    </li>
  );
}

// ---- Main App ----
const FILTERS = ["all", "active", "completed"];

export default function TodoApp() {
  const [todos, dispatch] = useReducer(reducer, []);
  const [input, setInput] = useState("");
  const [filter, setFilter] = useState("all");

  const addTodo = (e) => {
    e.preventDefault();
    if (!input.trim()) return;
    dispatch({ type: "ADD", text: input.trim() });
    setInput("");
  };

  const filtered = todos.filter((t) => {
    if (filter === "active") return !t.done;
    if (filter === "completed") return t.done;
    return true;
  });

  return (
    <div style={{ padding: 20, maxWidth: 400 }}>
      <h2>Todo App</h2>

      <form onSubmit={addTodo} style={{ display: "flex", gap: 8 }}>
        <input value={input} onChange={(e) => setInput(e.target.value)} placeholder="Add todo..." />
        <button type="submit">Add</button>
      </form>

      <div style={{ margin: "10px 0", display: "flex", gap: 8 }}>
        {FILTERS.map((f) => (
          <button key={f} onClick={() => setFilter(f)}
            style={{ fontWeight: filter === f ? "bold" : "normal" }}>
            {f}
          </button>
        ))}
      </div>

      <ul style={{ padding: 0, listStyle: "none" }}>
        {filtered.map((todo) => (
          <TodoItem key={todo.id} todo={todo} dispatch={dispatch} />
        ))}
      </ul>

      <p>{todos.filter((t) => !t.done).length} items left</p>
    </div>
  );
}
```

---

## 4. Virtual Scroll

```jsx
// App.jsx
import { useState, useRef, useMemo, useCallback } from "react";

const TOTAL_ITEMS = 10000;
const ITEM_HEIGHT = 40;
const CONTAINER_HEIGHT = 400;
const OVERSCAN = 5;

const allItems = Array.from({ length: TOTAL_ITEMS }, (_, i) => ({
  id: i,
  text: `Row ${i + 1} - Virtual item content`,
}));

export default function VirtualScroll() {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const visibleCount = Math.ceil(CONTAINER_HEIGHT / ITEM_HEIGHT);

  const { startIndex, visibleItems } = useMemo(() => {
    const start = Math.max(0, Math.floor(scrollTop / ITEM_HEIGHT) - OVERSCAN);
    const end = Math.min(TOTAL_ITEMS, start + visibleCount + OVERSCAN * 2);
    return {
      startIndex: start,
      visibleItems: allItems.slice(start, end),
    };
  }, [scrollTop, visibleCount]);

  const onScroll = useCallback((e) => {
    setScrollTop(e.target.scrollTop);
  }, []);

  return (
    <div>
      <h2>Virtual Scroll ({TOTAL_ITEMS} items)</h2>
      <div
        ref={containerRef}
        onScroll={onScroll}
        style={{ height: CONTAINER_HEIGHT, overflowY: "auto", border: "1px solid #ccc" }}
      >
        {/* Full height container to make scrollbar accurate */}
        <div style={{ height: TOTAL_ITEMS * ITEM_HEIGHT, position: "relative" }}>
          {/* Offset rendered items to correct position */}
          <div style={{ position: "absolute", top: startIndex * ITEM_HEIGHT, width: "100%" }}>
            {visibleItems.map((item) => (
              <div
                key={item.id}
                style={{
                  height: ITEM_HEIGHT,
                  borderBottom: "1px solid #eee",
                  display: "flex",
                  alignItems: "center",
                  padding: "0 12px",
                  background: item.id % 2 === 0 ? "#fafafa" : "#fff",
                }}
              >
                {item.text}
              </div>
            ))}
          </div>
        </div>
      </div>
      <small>Rendering items {startIndex} - {startIndex + visibleItems.length} of {TOTAL_ITEMS}</small>
    </div>
  );
}
```

---

## Bonus: Infinite Scroll (Intersection Observer)

```jsx
// App.jsx
import { useState, useEffect, useRef, useCallback } from "react";

const PAGE_SIZE = 20;

function generateItems(page) {
  return Array.from({ length: PAGE_SIZE }, (_, i) => ({
    id: (page - 1) * PAGE_SIZE + i + 1,
    text: `Item ${(page - 1) * PAGE_SIZE + i + 1}`,
  }));
}

export default function InfiniteScroll() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const sentinelRef = useRef(null);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    setLoading(true);

    // Simulate API call
    await new Promise((r) => setTimeout(r, 600));
    const newItems = generateItems(page);

    if (page >= 10) setHasMore(false); // Stop at page 10
    setItems((prev) => [...prev, ...newItems]);
    setPage((prev) => prev + 1);
    setLoading(false);
  }, [page, loading, hasMore]);

  // Intersection Observer watches sentinel div
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => { if (entries[0].isIntersecting) loadMore(); },
      { rootMargin: "100px" }
    );
    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [loadMore]);

  return (
    <div style={{ maxWidth: 400, padding: 20 }}>
      <h2>Infinite Scroll</h2>
      {items.map((item) => (
        <div key={item.id} style={{ padding: "12px", borderBottom: "1px solid #eee" }}>
          {item.text}
        </div>
      ))}

      {/* Sentinel - observer watches this element */}
      <div ref={sentinelRef} style={{ height: 20 }} />

      {loading && <p>Loading...</p>}
      {!hasMore && <p>No more items</p>}
    </div>
  );
}
```

---

## Key Concepts Cheat Sheet

```
Pagination    → useMemo for pageNumbers + slicing data
ProgressBar   → useEffect + setInterval + cleanup on return
Todo          → useReducer > useState for multi-action state
VirtualScroll → scrollTop ÷ itemHeight = startIndex,
                two divs: outer(total height) + inner(translateY)
InfScroll     → IntersectionObserver on sentinel div at bottom
```

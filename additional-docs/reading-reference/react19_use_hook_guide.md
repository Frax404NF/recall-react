# ⚡ React 19 [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) Hook — Complete Learning Guide

> **Source code studied:** `16-use` project (Padre Gino's Pizza app)

---

## 🧠 The Core Concept: What Is [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18)?

[use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) is a new React 19 hook that **unwraps a Promise** inside a component. When the promise is still pending, React **suspends** the component and shows a `<Suspense>` fallback instead.

```jsx
import { use, Suspense } from "react";

function MyComponent({ dataPromise }) {
  const data = use(dataPromise);   // 👈 Suspends until promise resolves
  return <div>{data.name}</div>;   // 👈 Only runs when data is ready
}

// Parent wraps it with Suspense
<Suspense fallback={<p>Loading...</p>}>
  <MyComponent dataPromise={fetchData()} />
</Suspense>
```

> **One-sentence summary:** [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) lets you read a promise's value directly — React handles the waiting for you.

---

## 🔍 The Evolution: Three Ways to Handle Async Data

````carousel
**Era 1: useEffect + useState (manual)**
```jsx
function MyComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/data")
      .then(res => res.json())
      .then(d => { setData(d); setLoading(false); });
  }, []);

  if (loading) return <p>Loading...</p>;
  return <div>{data.name}</div>;
}
```
3 state variables, manual loading state, effect cleanup needed
<!-- slide -->
**Era 2: useQuery (React Query)**
```jsx
function MyComponent() {
  const { isLoading, data } = useQuery({
    queryKey: ["data"],
    queryFn: () => fetch("/api/data").then(r => r.json()),
  });

  if (isLoading) return <p>Loading...</p>;
  return <div>{data.name}</div>;
}
```
Better! But still manual `if (isLoading)` check in every component
<!-- slide -->
**Era 3: use() + Suspense (React 19)**
```jsx
function MyComponent({ dataPromise }) {
  const data = use(dataPromise);  // Suspends automatically
  return <div>{data.name}</div>;  // Only renders with data
}

// Parent handles loading:
<Suspense fallback={<p>Loading...</p>}>
  <MyComponent dataPromise={query.promise} />
</Suspense>
```
No loading checks! Component assumes data exists. Loading handled by parent.
````

---

## 🔍 How It Works in the Code

### The Architecture Change

```mermaid
graph TD
    subgraph BEFORE ["Before: isLoading pattern"]
        C1["PastOrdersRoute"] --> CHECK{"isLoading?"}
        CHECK -->|Yes| LOAD["return Loading..."]
        CHECK -->|No| DATA["return data.map(...)"]
    end

    subgraph AFTER ["After: use() + Suspense"]
        PARENT["Wrapper Component"] --> SUSPENSE["&lt;Suspense fallback={Loading}&gt;"]
        SUSPENSE --> C2["PastOrdersRoute"]
        C2 --> USE["use(promise) → data"]
        USE --> DATA2["return data.map(...)"]
    end

    style BEFORE fill:#ef4444,color:#fff
    style AFTER fill:#22c55e,color:#000
```

### The Actual Code

```jsx
import { Suspense, useState, use } from "react";

// PARENT: Creates the promise and wraps with Suspense
function ErrorBoundaryWrappedPastOrderRoutes() {
  const [page, setPage] = useState(1);

  // 👈 .promise — get the raw promise from React Query
  const loadedPromise = useQuery({
    queryKey: ["past-orders", page],
    queryFn: () => getPastOrders(page),
    staleTime: 30000,
  }).promise;

  return (
    <ErrorBoundary>
      <Suspense fallback={<h2>Loading Past Orders …</h2>}>
        <PastOrdersRoute
          loadedPromise={loadedPromise}   // 👈 Pass promise as prop
          page={page}
          setPage={setPage}
        />
      </Suspense>
    </ErrorBoundary>
  );
}

// CHILD: Uses the promise — only renders when data is ready
function PastOrdersRoute({ loadedPromise, page, setPage }) {
  const data = use(loadedPromise);  // 👈 Suspends until resolved

  // No if (isLoading) check needed!
  // This code only runs when data is available
  return (
    <div className="past-orders">
      <table>
        <tbody>
          {data.map((order) => (...))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 📦 Key Concepts

### 1. [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) — The Promise Unwrapper

```jsx
const data = use(somePromise);
```

| What happens | Result |
|---|---|
| Promise is pending | Component **suspends** (shows Suspense fallback) |
| Promise resolves | Component renders with the resolved value |
| Promise rejects | Nearest ErrorBoundary catches it |

> [!IMPORTANT]
> Unlike other hooks, [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) **can** be called inside conditionals and loops. It's the only hook with this exception.

### 2. `<Suspense>` — The Loading Boundary

```jsx
<Suspense fallback={<LoadingSpinner />}>
  <ComponentThatMightSuspend />
</Suspense>
```

Suspense catches "suspensions" from child components (like [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18)) and shows the fallback until they're ready. Think of it like [ErrorBoundary](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/12-error-boundaries/src/ErrorBoundary.jsx#5-29) but for loading states.

| ErrorBoundary | Suspense |
|---|---|
| Catches **errors** | Catches **loading** |
| Shows error fallback | Shows loading fallback |
| Class component required | JSX component |
| [componentDidCatch](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/12-error-boundaries/src/ErrorBoundary.jsx#10-13) | Built into React |

### 3. `.promise` — React Query Integration

```jsx
const loadedPromise = useQuery({...}).promise;  // Get the raw promise
```

React Query's `.promise` property gives you a promise that [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) can consume. This requires:

```jsx
// In App.jsx — must enable this experimental feature
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      experimental_prefetchInRender: true,  // 👈 Required
    },
  },
});
```

---

## ❓ When Should You Use [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18)?

| Scenario | Use [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18)? |
|---|---|
| Data fetching with Suspense architecture | ✅ Yes |
| Simple component that fetches its own data | 🟡 `useQuery` is simpler |
| Server components (Next.js) | ✅ Yes — the primary use case |
| Reading Context conditionally | ✅ Yes — [use(Context)](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) works in if/else |
| Client-side data with React Query | 🟡 Experimental, but works |

> [!TIP]
> [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) shines most in **Next.js Server Components** where you fetch data on the server. For client-side React with React Query, `useQuery` with `isLoading` is still the simpler approach for most cases. But learning [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) now prepares you for the future.

---

## 💡 What Makes You a Better Engineer

### 1. **Declarative Loading States**
Instead of checking `if (loading)` in every component, you declare *where* loading states appear with `<Suspense>`. The parent decides the loading UI, the child just uses the data. This is a cleaner separation of concerns.

### 2. **Understanding React's Async Model**
React is moving toward a world where components can **suspend** — pause rendering until data is ready. Understanding this model is critical for Next.js App Router, Server Components, and streaming SSR.

### 3. **Composition Over Configuration**
The pattern of wrapping with `<Suspense>` and `<ErrorBoundary>` is **composition** — combining small, focused components to handle different states. This is a core React principle.

```jsx
<ErrorBoundary>          {/* Handles errors */}
  <Suspense fallback={}> {/* Handles loading */}
    <Component />        {/* Handles success */}
  </Suspense>
</ErrorBoundary>
```

### 4. **Interview Knowledge**

> [!NOTE]
> **Questions you can now answer:**
> - *"What's new in React 19?"*
> - *"What is the [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) hook?"*
> - *"How does Suspense work for data fetching?"*
> - *"What's the difference between `useEffect` + `useState` and [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) + Suspense?"*
> - *"How does [use()](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/usePizzaOfTheDay.jsx#3-18) differ from other hooks?" (can be called conditionally)*

---

## 📝 Summary Cheat Sheet

```
┌──────────────────────────────────────────────────┐
│           React 19 use() Cheat Sheet             │
├──────────────────────────────────────────────────┤
│                                                  │
│  BASIC USAGE:                                    │
│    const data = use(promise)                     │
│    // Suspends component until resolved          │
│                                                  │
│  WITH SUSPENSE:                                  │
│    <Suspense fallback={<Loading />}>             │
│      <ComponentUsing use() />                    │
│    </Suspense>                                   │
│                                                  │
│  WITH REACT QUERY:                               │
│    const promise = useQuery({...}).promise        │
│    // Requires experimental_prefetchInRender     │
│                                                  │
│  UNIQUE TRAIT:                                   │
│    use() CAN be called in if/else and loops      │
│    (only hook with this exception)               │
│                                                  │
│  MENTAL MODEL:                                   │
│    ErrorBoundary → catches errors                │
│    Suspense      → catches loading               │
│    Component     → handles success               │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

*Guide based on the `16-use` lesson from the Complete Intro to React v9 course. See also: [Form Actions](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react19_form_actions_guide.md) · [Error Boundaries](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/error_boundaries_guide.md) · [Testing](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/vitest_testing_guide.md)*

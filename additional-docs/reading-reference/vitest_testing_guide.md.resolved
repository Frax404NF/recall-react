# 🧪 Testing React with Vitest — Complete Learning Guide

> **Source code studied:** `14-testing` project (Padre Gino's Pizza app)

---

## 🧠 Why Test?

| Without Tests | With Tests |
|---|---|
| "It works on my machine" | Proof it works everywhere |
| Fear of refactoring | Confidence to change code |
| Bugs caught by users | Bugs caught before deploy |
| Manual clicking to verify | Automated in seconds |

> Testing isn't about proving your code works — it's about **catching when it breaks**.

---

## 🏗️ The Testing Architecture

This project uses **two testing environments** — this is a key insight:

```mermaid
graph TD
    VW["vitest.workspace.js"]
    
    subgraph NODE ["🟢 Node Tests (happy-dom)"]
        N1["Pizza.node.test.jsx"]
        N2["contact.lazy.node.test.jsx"]
        N3["usePizzaOfTheDay.node.test.jsx"]
    end
    
    subgraph BROWSER ["🔵 Browser Tests (Playwright)"]
        B1["Pizza.browser.test.jsx"]
        B2["Cart.browser.test.jsx"]
        B3["Header.browser.test.jsx"]
    end
    
    VW --> NODE
    VW --> BROWSER

    style NODE fill:#22c55e,color:#000
    style BROWSER fill:#3b82f6,color:#fff
```

### Why Two Environments?

| | Node (happy-dom) | Browser (Playwright) |
|---|---|---|
| **Speed** | ⚡ Very fast | 🐢 Slower |
| **DOM** | Simulated (fake) | Real browser |
| **Use for** | Logic, hooks, API calls | Visual rendering, user interactions |
| **File pattern** | `*.node.test.jsx` | `*.browser.test.jsx` |
| **Fidelity** | Good enough for most tests | 100% real |

> [!TIP]
> **Rule of thumb:** Start with node tests (fast). Only use browser tests when you need a real DOM (user interactions, visual assertions, context/routing providers).

---

## 📦 The Config: [vitest.workspace.js](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/vitest.workspace.js)

```jsx
import { defineWorkspace } from "vitest/config";

export default defineWorkspace([
  {
    extends: "./vite.config.js",        // Reuse your Vite config
    test: {
      include: ["**/*.node.test.{js,jsx}"],  // 👈 File naming convention
      name: "happy-dom",
      environment: "happy-dom",         // Simulated DOM in Node.js
    },
  },
  {
    extends: "./vite.config.js",
    test: {
      setupFiles: ["vitest-browser-react"],
      include: ["**/*.browser.test.{js,jsx}"],
      name: "browser",
      browser: {
        provider: "playwright",         // Real browser engine
        enabled: true,
        name: "firefox",               // Can be chromium, firefox, webkit
      },
    },
  },
]);
```

> The file naming convention (`*.node.test.*` vs `*.browser.test.*`) determines which environment runs each test.

---

## 🔍 Test Files Explained

### Test 1: [Pizza.node.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/Pizza.node.test.jsx) — Basic Component Testing

```jsx
import { render, cleanup } from "@testing-library/react";
import { afterEach, expect, test } from "vitest";
import Pizza from "../Pizza";

afterEach(cleanup);  // 🧹 Clean up DOM after each test

test("alt text renders on image", async () => {
  const name = "My Favorite Pizza";
  const src = "https://picsum.photos/200";
  
  // 1️⃣ Render the component with props
  const screen = render(
    <Pizza name={name} description="super cool pizza" image={src} />,
  );

  // 2️⃣ Find the element by its role
  const img = screen.getByRole("img");
  
  // 3️⃣ Assert what you expect
  expect(img.src).toBe(src);
  expect(img.alt).toBe(name);
});

test("to have default image if none is provided", async () => {
  const screen = render(
    <Pizza name={"Cool Pizza"} description="super cool pizza" />,
  );
  const img = screen.getByRole("img");
  expect(img.src).not.toBe("");  // Should have a fallback
});
```

**Pattern learned:** Render → Find → Assert

---

### Test 2: [Pizza.browser.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/Pizza.browser.test.jsx) — Same Test, Real Browser

```jsx
import { render } from "vitest-browser-react";     // 👈 Different import!
import { expect, test } from "vitest";
import Pizza from "../Pizza";

test("alt text renders on image", async () => {
  const name = "My Favorite Pizza";
  const src = "https://picsum.photos/200";
  const screen = render(
    <Pizza name={name} description="super cool pizza" image={src} />,
  );

  const img = await screen.getByRole("img");       // 👈 await (real browser is async)

  await expect.element(img).toBeInTheDocument();    // 👈 element() assertions
  await expect.element(img).toHaveAttribute("src", src);
  await expect.element(img).toHaveAttribute("alt", name);
});
```

**Key differences from node test:**

| Node Test | Browser Test |
|---|---|
| `@testing-library/react` | `vitest-browser-react` |
| `screen.getByRole()` | `await screen.getByRole()` |
| `expect(img.src).toBe()` | `await expect.element(img).toHaveAttribute()` |

---

### Test 3: [Cart.browser.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/Cart.browser.test.jsx) — Snapshot Testing

```jsx
test("snapshot with nothing in cart", () => {
  const { asFragment } = render(<Cart cart={[]} />);
  expect(asFragment()).toMatchSnapshot();  // 👈 Saves the rendered HTML
});

test("snapshot with some stuff in cart", () => {
  const { asFragment } = render(
    <Cart cart={[
      { pizza: { id: "pepperoni", name: "The Pepperoni Pizza", sizes: { S: 9.75, M: 12.5, L: 15.25 } }, size: "M", price: "$12.50" },
      // ... more items
    ]} />,
  );
  expect(asFragment()).toMatchSnapshot();
});
```

> [!NOTE]
> **What is snapshot testing?** The first run saves the rendered HTML to a `__snapshots__/` file. Future runs compare against it. If the output changes unexpectedly, the test fails. Run `vitest --update` to accept intentional changes.

**When snapshots are useful:**
- Catching unintentional UI changes
- Quick "did the markup change?" checks

**When they're NOT useful:**
- Logic testing
- Fragile when components change often

---

### Test 4: [Header.browser.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/Header.browser.test.jsx) — Testing with Context & Router

```jsx
import { RouterProvider, createRouter, createRootRoute } from "@tanstack/react-router";
import { CartContext } from "../contexts";

test("correctly renders a header with a zero cart count", async () => {
  // 1️⃣ Create a minimal router (Header needs <Link>, which needs a router)
  const rootRoute = createRootRoute({
    component: () => (
      // 2️⃣ Provide the context the component expects
      <CartContext.Provider value={[[]]}>  {/* Empty cart */}
        <Header />
      </CartContext.Provider>
    ),
  });
  const router = createRouter({ routeTree: rootRoute });

  // 3️⃣ Render with all required providers
  const screen = render(<RouterProvider router={router} />);

  // 4️⃣ Find by test ID (not by role or text)
  const itemsInCart = await screen.getByTestId("cart-number");

  await expect.element(itemsInCart).toHaveTextContent("0");
});
```

**Key learnings:**
- Components using `useContext` need their **Provider** in tests
- Components using `<Link>` need a **Router** wrapper
- `data-testid` is a reliable way to find elements (doesn't break when text changes)

---

### Test 5: [contact.lazy.node.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/contact.lazy.node.test.jsx) — Mocking API Calls

```jsx
import createFetchMock from "vitest-fetch-mock";
import { QueryClientProvider, QueryClient } from "@tanstack/react-query";
import { Route } from "../routes/contact.lazy";

const fetchMocker = createFetchMock(vi);
fetchMocker.enableMocks();       // 👈 Replace global fetch with mock

test("can submit contact form", async () => {
  // 1️⃣ Tell the mock what to return
  fetchMocker.mockResponse(JSON.stringify({ status: "ok" }));

  // 2️⃣ Render with required providers
  const screen = render(
    <QueryClientProvider client={new QueryClient()}>
      <Route.options.component />    {/* 👈 Render the route component directly */}
    </QueryClientProvider>,
  );

  // 3️⃣ Fill in form fields
  screen.getByPlaceholderText("Name").value = "Brian";
  screen.getByPlaceholderText("Email").value = "test@example.com";
  screen.getByPlaceholderText("Message").value = "This is a test message";

  // 4️⃣ Click submit
  screen.getByRole("button").click();

  // 5️⃣ Assert the success state
  const h3 = await screen.findByRole("heading", { level: 3 });
  expect(h3.innerText).toContain("Submitted");

  // 6️⃣ Assert the API was called correctly
  expect(fetchMocker).toHaveBeenCalledWith("/api/contact", {
    body: JSON.stringify({ name: "Brian", email: "test@example.com", message: "This is a test message" }),
    headers: { "Content-Type": "application/json" },
    method: "POST",
  });
});
```

> [!IMPORTANT]
> **Mocking** replaces real API calls with fake responses. You test your component's behavior *without needing a running server*. This is essential for CI/CD pipelines.

---

### Test 6: [usePizzaOfTheDay.node.test.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/14-testing/src/__tests__/usePizzaOfTheDay.node.test.jsx) — Testing Custom Hooks

```jsx
import { renderHook, waitFor } from "@testing-library/react";

test("to be null on initial load", async () => {
  fetch.mockResponseOnce(JSON.stringify(testPizza));
  const { result } = renderHook(() => usePizzaOfTheDay(""));
  expect(result.current).toBeNull();  // 👈 Before fetch resolves
});

test("to call the API and give back the pizza of the day", async () => {
  fetch.mockResponseOnce(JSON.stringify(testPizza));
  const { result } = renderHook(() => usePizzaOfTheDay(""));
  
  // 👈 waitFor: keep checking until the assertion passes
  await waitFor(() => {
    expect(result.current).toEqual(testPizza);
  });
  expect(fetchMocker).toBeCalledWith("/api/pizza-of-the-day");
});
```

**Key insight:** `renderHook` lets you test hooks **outside of a component**. You can't call hooks in plain functions — `renderHook` creates a minimal component wrapper for you.

---

## 📖 Deep Dive: Course Topics Explained

### 1. Testing — The Big Picture

Testing is about writing **code that checks your other code**. There are three main levels:

```mermaid
graph BT
    subgraph PYRAMID ["Testing Pyramid"]
        E2E["🔺 E2E Tests<br/>Slow, expensive, few"]
        INT["🔶 Integration Tests<br/>Medium speed, moderate count"]
        UNIT["🟩 Unit Tests<br/>Fast, cheap, many"]
    end

    UNIT --> INT --> E2E

    style UNIT fill:#22c55e,color:#000
    style INT fill:#f59e0b,color:#000
    style E2E fill:#ef4444,color:#fff
```

| Level | What It Tests | Speed | Example |
|---|---|---|---|
| **Unit** | Single function or component in isolation | ⚡ ms | "Does `<Pizza>` render the correct image?" |
| **Integration** | Multiple components working together | 🟡 seconds | "Can the user submit the contact form?" |
| **E2E** | The entire app from a user's perspective | 🐢 minutes | "Can a user order a pizza and see it in past orders?" |

> [!TIP]
> **The Testing Pyramid says:** Write many unit tests (cheap, fast), fewer integration tests, and very few E2E tests. Most bugs are caught at the unit level.

**Types of testing you'll encounter at work:**

| Type | Who Writes It | When It Runs |
|---|---|---|
| **Unit tests** | Developers | Every commit (CI) |
| **Integration tests** | Developers | Every PR / merge |
| **E2E tests** | QA / Developers | Before release |
| **Manual testing** | QA team | Before release |
| **Smoke tests** | Automated | After deploy to staging |

---

### 2. Vitest — Why This Tool?

Vitest is a **test runner** — it finds your test files, runs them, and reports pass/fail. It was built specifically for Vite projects.

**Why Vitest over Jest (the old standard)?**

| Feature | Jest | Vitest |
|---|---|---|
| Speed | 🐢 Slower (separate build) | ⚡ Faster (reuses Vite) |
| Config | Separate config needed | Extends [vite.config.js](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/11-modals/vite.config.js) |
| ESM support | Needs workarounds | Native |
| HMR for tests | No | Yes (re-runs on save) |
| Compatible API | ✅ `test`, `expect`, `vi` | ✅ Same API as Jest |

**Key Vitest concepts:**

```jsx
import { test, expect, vi, afterEach } from "vitest";

test("description", () => { ... });      // Define a test
expect(value).toBe(expected);            // Make an assertion
vi.fn()                                  // Create a mock function
vi.mock("module")                        // Mock an entire module
afterEach(() => { ... })                 // Cleanup after each test
```

> The API is almost identical to Jest — if you learn Vitest, you already know Jest too.

---

### 3. Basic React Tests

The fundamental pattern for testing any React component:

```
1. RENDER   →  Put the component in a simulated DOM
2. FIND     →  Locate the element you want to assert on
3. ASSERT   →  Check that it matches your expectations
```

```jsx
// Step 1: Render
const screen = render(<Pizza name="Pepperoni" description="Cheesy" image="/pepperoni.webp" />);

// Step 2: Find
const img = screen.getByRole("img");

// Step 3: Assert
expect(img.alt).toBe("Pepperoni");
```

**How to find elements (priority order from Testing Library docs):**

| Priority | Method | When to Use |
|---|---|---|
| 1st ✅ | `getByRole("button")` | Accessible elements (best practice) |
| 2nd | `getByLabelText("Email")` | Form inputs |
| 3rd | `getByPlaceholderText("Name")` | When no label exists |
| 4th | `getByText("Submit")` | Visible text |
| 5th | `getByTestId("cart-number")` | Fallback — always works |

> [!IMPORTANT]
> **`getByRole` is the gold standard** because it queries the way a screen reader would — if your test works with `getByRole`, your app is accessible too. Use `getByTestId` as a fallback when there's no semantic role.

---

### 4. Testing User Interaction

This goes beyond "does it render?" to "does it respond to clicks, form submissions, and state changes?"

The contact form test demonstrates this full flow:

```jsx
// 1. Render the form
render(<ContactRoute />);

// 2. Simulate user filling in fields
screen.getByPlaceholderText("Name").value = "Brian";
screen.getByPlaceholderText("Email").value = "test@example.com";

// 3. Simulate user clicking submit
screen.getByRole("button").click();

// 4. Assert what happens AFTER the interaction
const h3 = await screen.findByRole("heading", { level: 3 });
expect(h3.innerText).toContain("Submitted");
```

**Key patterns for interaction testing:**

| Interaction | How to Test |
|---|---|
| Button click | `element.click()` |
| Type in input | `element.value = "text"` |
| Select dropdown | `fireEvent.change(select, { target: { value: "option" } })` |
| Form submit | Click submit button or `fireEvent.submit(form)` |
| Wait for async result | `await screen.findByRole(...)` or `waitFor()` |

> The distinction between `getBy*` and `findBy*` is critical: `getBy` fails immediately if not found, `findBy` waits (retries) until the element appears or times out. Use `findBy` after async operations.

---

### 5. Testing Custom Hooks

Hooks can't be called outside of components — so how do you test them? With `renderHook`:

```jsx
import { renderHook, waitFor } from "@testing-library/react";
import { usePizzaOfTheDay } from "../usePizzaOfTheDay";

// renderHook creates a tiny invisible component that calls your hook
const { result } = renderHook(() => usePizzaOfTheDay());

// result.current = whatever your hook returns right NOW
expect(result.current).toBeNull();  // Initially null (loading)

// Wait for async state update
await waitFor(() => {
  expect(result.current).toEqual(testPizza);  // After fetch resolves
});
```

**Why this matters:** Custom hooks are your app's **reusable logic**. Testing them independently means you can verify they work correctly without worrying about UI.

```mermaid
graph LR
    RH["renderHook()"] --> MC["Creates mini component"]
    MC --> HC["Calls your hook inside it"]
    HC --> R["result.current = hook's return value"]
    
    style RH fill:#3b82f6,color:#fff
    style R fill:#22c55e,color:#000
```

---

### 6. Snapshot Testing

Snapshots capture the **exact HTML output** of a component and save it to a file. Future runs compare against this saved version.

**How it works:**

```
First run:  Component renders → HTML saved to __snapshots__/Cart.browser.test.jsx.snap
Next runs:  Component renders → Compare with saved snapshot
             ├── Same? → ✅ PASS
             └── Different? → ❌ FAIL (show the diff)
```

```jsx
const { asFragment } = render(<Cart cart={[]} />);
expect(asFragment()).toMatchSnapshot();
```

**The saved snapshot looks like:**

```
exports[`snapshot with nothing in cart 1`] = `
<DocumentFragment>
  <div class="cart">
    <h2>Cart</h2>
    <ul />
    <p>Total: $0.00</p>
    <button>Checkout</button>
  </div>
</DocumentFragment>
`;
```

**When a test fails, you decide:**
- Was the change intentional? → Run `vitest --update` to accept
- Was the change accidental? → Fix the bug

| ✅ Good for | ❌ Bad for |
|---|---|
| Catching unintended UI changes | Logic testing |
| Quick regression checks | Components that change frequently |
| Stable, "finished" components | Dynamic content (dates, random IDs) |

> [!WARNING]
> Don't over-rely on snapshots. They catch *that something changed*, not *what went wrong*. Developers often blindly update snapshots without reading the diff. Always use them alongside targeted assertions.

---

### 7. V8 Coverage — How Much Is Tested?

**Code coverage** measures what percentage of your code is exercised by tests. V8 is the JavaScript engine (from Chrome) that Vitest uses to track this.

```bash
npm run coverage    # Generates a coverage report
```

**Coverage metrics:**

| Metric | What It Measures |
|---|---|
| **Statements** | % of code statements executed |
| **Branches** | % of `if/else` paths taken |
| **Functions** | % of functions called |
| **Lines** | % of lines executed |

**Example output:**

```
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
 Pizza.jsx          |   100   |   100    |   100   |   100   |
 Cart.jsx           |   100   |    80    |   100   |   100   |
 Header.jsx         |    75   |    50    |   100   |    75   |
--------------------|---------|----------|---------|---------|
```

> [!TIP]
> **Don't chase 100% coverage.** 80% is a common industry target. The last 20% often tests trivial code (CSS classes, edge cases that can't happen). Focus on testing **critical paths** — the code where bugs would hurt the most.

**Coverage wisdom:**
- High coverage ≠ good tests (you can have 100% coverage with useless assertions)
- Low coverage = definitely untested code
- Coverage is a **compass**, not a destination

---

### 8. Vitest UI — Visual Test Dashboard

`npm run test:ui` opens an interactive browser-based dashboard:

```bash
npm run test:ui    # Opens at http://localhost:51204/__vitest__/
```

**What you get:**
- ✅ Visual pass/fail for every test
- 📂 File tree navigation
- 🔍 Click a test to see its code
- 📊 Coverage visualization
- 🔄 Re-runs automatically on save
- ⏱️ Execution time per test

**When to use the UI:**
- Debugging failing tests (see the full error in context)
- Exploring which tests exist
- Showing test results in a team demo / PR review

> This is a Vitest-exclusive feature — Jest doesn't have a built-in UI. It's one of the quality-of-life improvements that makes Vitest pleasant to use daily.

---

### 9. Browser Tests — The Real Deal

Browser tests run your components inside an **actual browser** (Firefox, Chrome, or Safari) using Playwright.

**Why bother when happy-dom works?**

| happy-dom (Node) | Playwright (Browser) |
|---|---|
| Simulates the DOM in JavaScript | Uses a real browser engine |
| No CSS rendering | Full CSS support |
| No real layout/positioning | Real layout calculations |
| Some browser APIs missing | All APIs available |
| ⚡ ~10ms per test | 🐢 ~500ms per test |

**When you NEED browser tests:**
- Testing CSS-dependent behavior (visibility, layout)
- Testing browser-specific APIs (IntersectionObserver, ResizeObserver)
- Testing with real events (not synthetic ones)
- When happy-dom gives false positives

**The API differences matter:**

````carousel
**Node test (synchronous):**
```jsx
import { render } from "@testing-library/react";

const screen = render(<Pizza name="Test" />);
const img = screen.getByRole("img");          // Synchronous
expect(img.src).toBe(url);                    // Direct property access
```
<!-- slide -->
**Browser test (asynchronous):**
```jsx
import { render } from "vitest-browser-react";

const screen = render(<Pizza name="Test" />);
const img = await screen.getByRole("img");    // Must await
await expect.element(img).toHaveAttribute("src", url);  // element() wrapper
```
````

> [!NOTE]
> Browser tests use `await` everywhere because they communicate with a separate process (the browser) over a wire protocol. This is the same architecture as Selenium, Cypress, and Playwright E2E tests.

---

## 🛠️ Common Testing Patterns Cheat Sheet

### Finding Elements

```jsx
screen.getByRole("button")           // By ARIA role (best practice)
screen.getByText("Submit")           // By visible text
screen.getByPlaceholderText("Name")  // By placeholder
screen.getByTestId("cart-number")    // By data-testid (most reliable)
screen.findByRole("heading")         // Same but async (waits for element)
```

### Assertions

```jsx
expect(value).toBe(expected)            // Exact match
expect(value).toEqual(object)           // Deep equality
expect(value).not.toBe(wrong)           // Negation
expect(value).toContain("text")         // String/array contains
expect(fn).toHaveBeenCalledWith(args)   // Mock was called with

// Browser-only:
await expect.element(el).toBeInTheDocument()
await expect.element(el).toHaveAttribute("src", url)
await expect.element(el).toHaveTextContent("3")
```

### Async Patterns

```jsx
// Wait for element to appear:
const el = await screen.findByRole("heading");

// Wait for condition:
await waitFor(() => {
  expect(result.current).toEqual(expected);
});
```

---

## ❓ Do Frontend Devs Test Day-to-Day?

| Company Size | Testing Culture |
|---|---|
| Startups | Minimal — focus on speed |
| Mid-size | Unit tests + some integration |
| Enterprise | Full coverage requirements |
| Open source | Tests required for PRs |

**What gets tested in practice:**
```
📊 Typical React project test breakdown:

  60% ████████████████████░░░░░░░░░░ Unit tests (components, hooks, utils)
  30% ██████████░░░░░░░░░░░░░░░░░░░░ Integration tests (user flows)
  10% ████░░░░░░░░░░░░░░░░░░░░░░░░░░ E2E tests (full app in browser)
```

---

## 💡 What Makes You a Better Engineer

### 1. **Confidence to Refactor**
Tests are a safety net. When you change code, tests tell you immediately if you broke something. This is the #1 value of testing.

### 2. **Understanding Test Doubles (Mocking)**
`vitest-fetch-mock` replaces real [fetch](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/routes/order.lazy.jsx#52-58) with fake responses — this is a **test double**. Understanding when and why to mock is a fundamental testing skill.

### 3. **Testing Pyramid Intuition**
You now know *three* levels: unit tests (fast, simple), component tests (medium), browser tests (slow, realistic). Knowing which to use and when is professional engineering judgment.

### 4. **Making Code Testable**
Adding `data-testid` to [Header.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/11-modals/src/Header.jsx) is an example of **designing for testability**. Good engineers write code that's easy to test from the start.

### 5. **Interview Knowledge**

> [!NOTE]
> **Questions you can now answer:**
> - *"How do you test React components?"*
> - *"What's the difference between unit and integration tests?"*
> - *"How do you mock API calls in tests?"*
> - *"What is snapshot testing?"*
> - *"How do you test custom hooks?"*
> - *"How do you test components that use Context?"*

---

## 📝 Summary Cheat Sheet

```
┌──────────────────────────────────────────────────┐
│            Vitest Testing Cheat Sheet            │
├──────────────────────────────────────────────────┤
│                                                  │
│  SETUP:     vitest.workspace.js                  │
│             *.node.test.jsx → happy-dom (fast)   │
│             *.browser.test.jsx → playwright      │
│                                                  │
│  COMMANDS:  npm test        → run all tests      │
│             npm test:ui     → visual UI          │
│             npm run coverage → coverage report   │
│                                                  │
│  PATTERN:   Render → Find → Assert               │
│                                                  │
│  FIND BY:   getByRole (best), getByText,         │
│             getByTestId (most reliable)           │
│                                                  │
│  ASYNC:     findBy* (waits for element)           │
│             waitFor(() => expect(...))            │
│                                                  │
│  MOCKING:   createFetchMock(vi)                  │
│             fetchMocker.mockResponse(data)        │
│                                                  │
│  HOOKS:     renderHook(() => useMyHook())         │
│                                                  │
│  SNAPSHOTS: expect(asFragment()).toMatchSnapshot()│
│             vitest --update to accept changes     │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

*Guide based on the `14-testing` lesson from the Complete Intro to React v9 course. See also: [Portals](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react_portals_guide.md) · [Error Boundaries](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/error_boundaries_guide.md) · [Lifecycle](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react_lifecycle_guide.md) · [Uncontrolled Forms](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/uncontrolled_forms_guide.md)*

# 📝 Uncontrolled Forms — Complete Learning Guide

> **Source code studied:** `13-uncontrolled-forms` project (Padre Gino's Contact page)

---

## 🧠 The Core Concept: Who Owns the Data?

Every form input has a value. The fundamental question is: **who tracks that value?**

| Approach | Who Owns the Data? | How You Get the Value |
|---|---|---|
| **Controlled** | React (via `useState`) | `state` variable |
| **Uncontrolled** | The DOM/browser itself | `FormData` API or `ref` |

### One-Sentence Summary:
> **Controlled** = React tracks every keystroke. **Uncontrolled** = let the browser handle it, grab the values when you need them.

---

## 🔍 Your Code Has BOTH — Let's Compare

### ⚡ Controlled Form (your [order.lazy.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/11-modals/src/routes/order.lazy.jsx))

```jsx
// Every input is tracked by React state — CONTROLLED
function Order() {
  const [pizzaType, setPizzaType] = useState("pepperoni");  // 👈 React owns it
  const [pizzaSize, setPizzaSize] = useState("M");          // 👈 React owns it

  return (
    <form onSubmit={handleSubmit}>
      <select
        onChange={(e) => setPizzaType(e.target.value)}  // 👈 Every change → state
        value={pizzaType}                               // 👈 React controls value
      >
        {/* options */}
      </select>
      <input
        onChange={(e) => setPizzaSize(e.target.value)}  // 👈 Every change → state
        checked={pizzaSize === "M"}                    // 👈 React controls checked
        type="radio"
      />
    </form>
  );
}
```

**What happens on each keystroke:**
```
User types → onChange fires → setState → Re-render → Input shows new value
```

### ✨ Uncontrolled Form (new [contact.lazy.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/13-uncontrolled-forms/src/routes/contact.lazy.jsx))

```jsx
// NO state for inputs — UNCONTROLLED
function ContactRoute() {
  const mutation = useMutation({
    mutationFn: function (e) {
      e.preventDefault();
      const formData = new FormData(e.target);       // 👈 Grab ALL values at once
      return postContact(
        formData.get("name"),                         // 👈 Read by name attribute
        formData.get("email"),
        formData.get("message"),
      );
    },
  });

  return (
    <form onSubmit={mutation.mutate}>
      <input name="name" placeholder="Name" />        {/* No onChange! No value! */}
      <input type="email" name="email" placeholder="Email" />
      <textarea placeholder="Message" name="message"></textarea>
      <button>Submit</button>
    </form>
  );
}
```

**What happens on each keystroke:**
```
User types → Browser handles it natively → Nothing else happens
(Only on submit: FormData collects all values at once)
```

---

## 🆚 Side-by-Side Comparison

````carousel
**Controlled (Order page):**
```jsx
// 2 state variables for 2 inputs
const [pizzaType, setPizzaType] = useState("pepperoni");
const [pizzaSize, setPizzaSize] = useState("M");

// Every input needs onChange + value
<select onChange={e => setPizzaType(e.target.value)} value={pizzaType}>
<input onChange={e => setPizzaSize(e.target.value)} checked={...}>

// On submit: values already in state
onSubmit={() => addToCart(pizzaType, pizzaSize)}
```
<!-- slide -->
**Uncontrolled (Contact page):**
```jsx
// ZERO state variables for inputs!

// Inputs just have name attributes
<input name="name" placeholder="Name" />
<input name="email" placeholder="Email" />
<textarea name="message"></textarea>

// On submit: grab values from FormData
onSubmit={(e) => {
  const formData = new FormData(e.target);
  postContact(formData.get("name"), formData.get("email"), ...);
}}
```
````

| Feature | Controlled | Uncontrolled |
|---|---|---|
| State variables needed | One per input | **Zero** |
| `onChange` handlers | Required on every input | Not needed |
| `value` prop | Required | Not set |
| Re-renders per keystroke | Yes (every keypress) | **None** |
| When you get the value | Anytime (it's in state) | Only on submit |
| Code complexity | More boilerplate | Simpler |
| Validation on each keystroke | ✅ Easy | ❌ Not built-in |
| Performance on large forms | More re-renders | Better |

---

## 📦 The `FormData` API — The Key Tool

`FormData` is a **browser-native API** (not React-specific!) that collects all form values at once:

```jsx
function handleSubmit(e) {
  e.preventDefault();

  // Creates a FormData object from the <form> element
  const formData = new FormData(e.target);

  // Get individual values by the input's "name" attribute
  const name = formData.get("name");        // matches <input name="name">
  const email = formData.get("email");      // matches <input name="email">
  const message = formData.get("message");  // matches <textarea name="message">
}
```

> [!IMPORTANT]
> The `name` attribute on each input is **critical** — that's how `FormData.get()` finds the value. Without `name`, the input is invisible to `FormData`.

### Other FormData Methods

```jsx
const formData = new FormData(form);

formData.get("field")      // Get one value
formData.getAll("colors")  // Get all values (for checkboxes)
formData.has("field")      // Check if field exists
formData.entries()         // Iterate all key-value pairs

// Convert to plain object:
const data = Object.fromEntries(formData);
// → { name: "John", email: "john@test.com", message: "Hello" }
```

---

## 🔄 The `useMutation` Hook — Bonus Concept

The contact form also introduces `useMutation` from TanStack React Query:

```jsx
const mutation = useMutation({
  mutationFn: function (e) {      // 👈 Called when mutation.mutate(e) fires
    e.preventDefault();
    const formData = new FormData(e.target);
    return postContact(...);      // 👈 Returns a promise
  },
});

// In JSX:
<form onSubmit={mutation.mutate}>  {/* 👈 Triggers the mutation */}

// Status tracking for free:
mutation.isSuccess    // true after successful submit
mutation.isLoading    // true while request is in progress
mutation.isError      // true if request failed
mutation.error        // the error object
```

| `useQuery` (what you already know) | `useMutation` (new!) |
|---|---|
| For **reading** data (GET) | For **writing** data (POST/PUT/DELETE) |
| Runs automatically | Runs when you call `mutate()` |
| Has `isLoading`, `data` | Has `isSuccess`, `isLoading`, `isError` |

---

## ❓ When Should You Use Each?

```mermaid
graph TD
    Q["Do you need the input value<br/>while the user is typing?"]
    Q -->|"YES"| CTRL["Use CONTROLLED"]
    Q -->|"NO"| UNCTRL["Use UNCONTROLLED"]

    CTRL --> C1["Real-time validation"]
    CTRL --> C2["Dynamic UI based on input"]
    CTRL --> C3["Conditional form fields"]
    CTRL --> C4["Character counters"]

    UNCTRL --> U1["Simple submit-and-done forms"]
    UNCTRL --> U2["Contact / feedback forms"]
    UNCTRL --> U3["File uploads"]
    UNCTRL --> U4["Performance-sensitive large forms"]

    style CTRL fill:#3b82f6,color:#fff
    style UNCTRL fill:#22c55e,color:#000
```

### Quick Decision Guide

| Scenario | Use |
|---|---|
| "Show remaining characters" as user types | **Controlled** |
| "Disable button unless fields are valid" | **Controlled** |
| "Show/hide fields based on a dropdown" | **Controlled** |
| Simple contact/feedback form | **Uncontrolled** ✅ |
| Login form (just email + password) | **Either works** |
| Form with 20+ fields | **Uncontrolled** (performance) |
| File upload | **Uncontrolled** |

---

## 💡 What Makes You a Better Engineer

### 1. **Knowing When NOT to Over-Engineer**
The contact form has 3 fields — it doesn't need 3 state variables and 3 `onChange` handlers. Uncontrolled forms teach you to **use the simplest tool for the job**.

### 2. **Understanding the Browser Platform**
`FormData` is a native browser API, not a React thing. Learning it makes you effective beyond React — it works with vanilla JS, Vue, Svelte, anything.

### 3. **Performance Awareness**
Controlled forms re-render on every keystroke. On large forms (20+ fields), this causes noticeable lag. Understanding this trade-off is a real engineering skill.

### 4. **Understanding Mutations**
`useMutation` separates reads from writes — a pattern from backend architecture (CQRS). Understanding this makes you think more clearly about data flow.

### 5. **Interview Knowledge**

> [!NOTE]
> **Questions you can now answer:**
> - *"What's the difference between controlled and uncontrolled components?"*
> - *"When would you choose uncontrolled over controlled?"*
> - *"What is FormData and how does it work?"*
> - *"How do you handle form submissions in React?"*
> - *"What's the performance implication of controlled forms?"*

---

## 📝 Summary Cheat Sheet

```
┌──────────────────────────────────────────────────┐
│         Controlled vs Uncontrolled Forms         │
├──────────────────────────────────────────────────┤
│                                                  │
│  CONTROLLED:                                     │
│    - useState() per input                        │
│    - onChange + value on every input              │
│    - Re-renders every keystroke                  │
│    - Use when: real-time validation,             │
│      dynamic UI, conditional fields              │
│                                                  │
│  UNCONTROLLED:                                   │
│    - No state for inputs                         │
│    - name attribute on inputs                    │
│    - FormData on submit                          │
│    - Use when: simple forms, performance         │
│      matters, just need values at submit         │
│                                                  │
│  FORMDATA API:                                   │
│    const fd = new FormData(e.target)             │
│    fd.get("fieldName")                           │
│    Object.fromEntries(fd)  // → plain object     │
│                                                  │
│  BONUS - useMutation:                            │
│    For POST/PUT/DELETE (writing data)             │
│    mutation.mutate() to trigger                  │
│    mutation.isSuccess / isLoading / isError       │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

*Guide based on the `13-uncontrolled-forms` lesson from the Complete Intro to React v9 course. See also: [Portals Guide](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react_portals_guide.md) · [Error Boundaries Guide](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/error_boundaries_guide.md) · [Lifecycle Guide](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react_lifecycle_guide.md)*

# ⚡ React 19 Form Actions — Complete Learning Guide

> **Source code studied:** `15-form-actions` project (Padre Gino's Pizza app)

---

## 🧠 The Core Concept: What Changed in React 19?

React 19 introduced a new way to handle form submissions — the `action` prop. It replaces the old `onSubmit` pattern.

````carousel
**Before React 19 (old way):**
```jsx
<form onSubmit={(e) => {
  e.preventDefault();           // 👈 Must manually prevent reload
  const formData = new FormData(e.target);
  submitForm(formData);
}}>
```
<!-- slide -->
**React 19 (new way):**
```jsx
<form action={submitForm}>      // 👈 Cleaner! No e.preventDefault()
```
React automatically:
- Prevents page reload
- Creates `FormData` from the form
- Passes it to your function
````

---

## 🔍 What Changed in the Code

### Change 1: [order.lazy.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/11-modals/src/routes/order.lazy.jsx) — `onSubmit` → `action`

```diff
+  function addToCart() {
+    setCart([...cart, { pizza: selectedPizza, size: pizzaSize, price }]);
+  }

-  <form onSubmit={(e) => {
-    e.preventDefault();
-    setCart([...cart, { pizza: selectedPizza, size: pizzaSize, price }]);
-  }}>
+  <form action={addToCart}>
```

**What's happening:**
1. The inline handler is extracted to a named function ([addToCart](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/15-form-actions/src/routes/order.lazy.jsx#60-63))
2. `onSubmit` → `action` — React 19 handles `preventDefault` for you
3. [addToCart](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/15-form-actions/src/routes/order.lazy.jsx#60-63) doesn't need `FormData` because it uses state (`pizzaType`, `pizzaSize`) instead

### Change 2: [contact.lazy.jsx](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/padre-gino/src/routes/contact.lazy.jsx) — `useFormStatus` + [ContactInput](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/15-form-actions/src/routes/contact.lazy.jsx#38-49)

```jsx
import { useFormStatus } from "react-dom";        // 👈 NEW React 19 hook

function ContactRoute() {
  const mutation = useMutation({
    mutationFn: function (formData) {              // 👈 Receives FormData directly
      return postContact(                          //    (no e.preventDefault needed!)
        formData.get("name"),
        formData.get("email"),
        formData.get("message"),
      );
    },
  });

  return (
    <form action={mutation.mutate}>                {/* 👈 action= not onSubmit= */}
      <ContactInput name="name" type="text" placeholder="Name" />
      <ContactInput name="email" type="email" placeholder="Email" />
      <textarea placeholder="Message" name="message"></textarea>
      <button>Submit</button>
    </form>
  );
}

// 👈 NEW: Separate component to access form status
function ContactInput(props) {
  const { pending } = useFormStatus();             // 👈 Is the form submitting?
  return (
    <input
      disabled={pending}                           // 👈 Disable while submitting
      name={props.name}
      type={props.type}
      placeholder={props.placeholder}
    />
  );
}
```

---

## 📦 The New React 19 APIs

### 1. `action` prop on `<form>`

| `onSubmit` (old) | `action` (React 19) |
|---|---|
| Receives the event (`e`) | Receives `FormData` directly |
| Must call `e.preventDefault()` | Automatic |
| Manual `new FormData(e.target)` | FormData passed as argument |
| No built-in pending state | Integrates with `useFormStatus` |

```jsx
// OLD: onSubmit
<form onSubmit={(e) => {
  e.preventDefault();
  const fd = new FormData(e.target);
  submit(fd);
}}>

// NEW: action (React 19)
<form action={(formData) => submit(formData)}>
// or even simpler:
<form action={submit}>
```

> [!IMPORTANT]
> When the `action` function is **async**, React automatically tracks the pending state — no manual `isLoading` state needed.

### 2. `useFormStatus` — Know When the Form Is Submitting

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending}>
      {pending ? "Sending..." : "Submit"}
    </button>
  );
}
```

**The returned object:**

| Property | Type | What It Tells You |
|---|---|---|
| `pending` | `boolean` | Is the form currently submitting? |
| `data` | `FormData | null` | The FormData being submitted |
| `method` | `string | null` | GET or POST |
| `action` | `function | null` | The action function |

> [!WARNING]
> **Critical rule:** `useFormStatus` must be called from a component that is a **child** of the `<form>` — it cannot be called in the same component that renders the form. That's why the course creates [ContactInput](file:///d:/Code-2026/frontend-master/react-v9-course/citr-v9-project/15-form-actions/src/routes/contact.lazy.jsx#38-49) as a separate component.

```jsx
// ❌ WRONG — useFormStatus in the same component as <form>
function ContactForm() {
  const { pending } = useFormStatus();  // Won't work!
  return <form action={submit}>...</form>;
}

// ✅ CORRECT — useFormStatus in a CHILD of <form>
function ContactForm() {
  return (
    <form action={submit}>
      <SubmitButton />  {/* useFormStatus works here */}
    </form>
  );
}
```

### 3. `useActionState` (Bonus — not in this lesson but good to know)

React 19 also added `useActionState` for managing form state + server responses:

```jsx
import { useActionState } from "react";

function ContactForm() {
  const [state, formAction, isPending] = useActionState(
    async (previousState, formData) => {
      const result = await submitContact(formData);
      return result;  // becomes the new state
    },
    null  // initial state
  );

  return (
    <form action={formAction}>
      {state?.error && <p className="error">{state.error}</p>}
      <input name="name" />
      <button disabled={isPending}>Submit</button>
    </form>
  );
}
```

---

## 🆚 Evolution of Form Handling in React

```mermaid
graph TD
    A["React &lt;18: Controlled Forms"] --> B["React 18: Uncontrolled + FormData"]
    B --> C["React 19: Form Actions"]
    
    A1["useState per input<br/>onChange handlers<br/>e.preventDefault()"] --> A
    B1["No state for inputs<br/>FormData on submit<br/>e.preventDefault()"] --> B
    C1["action prop<br/>FormData auto-passed<br/>useFormStatus for pending<br/>No e.preventDefault()"] --> C

    style A fill:#ef4444,color:#fff
    style B fill:#f59e0b,color:#000
    style C fill:#22c55e,color:#000
```

| Era | Pattern | Boilerplate |
|---|---|---|
| **Controlled** (React <18) | `useState` + `onChange` + `onSubmit` + `e.preventDefault()` | High |
| **Uncontrolled** (React 18) | `FormData` + `onSubmit` + `e.preventDefault()` | Medium |
| **Form Actions** (React 19) | `action` + auto `FormData` + `useFormStatus` | **Low** ✨ |

---

## ❓ Should You Use Form Actions Today?

| Framework | Support |
|---|---|
| Next.js 14+ | ✅ Full support (Server Actions) |
| Remix | ✅ Similar pattern built-in |
| Vite + React | ✅ Client-side actions work |
| React Native | ❌ No `<form>` element |

> [!TIP]
> Form actions are the **future direction** of React. Next.js already builds heavily on them with Server Actions. Learning them now means you're ahead of the curve.

---

## 💡 What Makes You a Better Engineer

### 1. **Less Boilerplate, Fewer Bugs**
Every `e.preventDefault()` you forget is a page reload. Every `new FormData(e.target)` is manual work. Form actions eliminate both — the platform does it for you.

### 2. **Progressive Enhancement**
Form actions work without JavaScript! If JS fails to load, the form still submits (as a regular HTML form). This is a huge win for accessibility and reliability.

### 3. **Understanding React's Direction**
React is moving toward tighter integration with the platform (HTML forms, streaming, server components). Understanding form actions shows you the trajectory of the framework.

### 4. **`useFormStatus` is a Design Pattern**
The rule that `useFormStatus` must be in a child component teaches you **component composition** — a fundamental React pattern. This "lift state down" pattern appears everywhere.

### 5. **Interview Knowledge**

> [!NOTE]
> **Questions you can now answer:**
> - *"What's new in React 19?"*
> - *"What are form actions?"*
> - *"What's the difference between `onSubmit` and `action`?"*
> - *"How do you show loading states during form submission?"*
> - *"What is `useFormStatus` and why must it be in a child component?"*
> - *"What is progressive enhancement?"*

---

## 📝 Summary Cheat Sheet

```
┌──────────────────────────────────────────────────┐
│        React 19 Form Actions Cheat Sheet         │
├──────────────────────────────────────────────────┤
│                                                  │
│  OLD WAY:                                        │
│    <form onSubmit={(e) => {                      │
│      e.preventDefault();                         │
│      const fd = new FormData(e.target);          │
│      submit(fd);                                 │
│    }}>                                           │
│                                                  │
│  NEW WAY (React 19):                             │
│    <form action={submit}>    // ← FormData auto  │
│                                                  │
│  PENDING STATE:                                  │
│    import { useFormStatus } from "react-dom"      │
│    const { pending } = useFormStatus()            │
│    ⚠️ Must be in a CHILD component of <form>      │
│                                                  │
│  FORM STATE:                                     │
│    import { useActionState } from "react"         │
│    const [state, action, pending] =               │
│      useActionState(fn, initialState)             │
│                                                  │
│  KEY BENEFITS:                                   │
│    ✅ No e.preventDefault()                       │
│    ✅ FormData passed automatically               │
│    ✅ Built-in pending tracking                   │
│    ✅ Progressive enhancement (works w/o JS)      │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

*Guide based on the `15-form-actions` lesson from the Complete Intro to React v9 course. See also: [Portals](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/react_portals_guide.md) · [Error Boundaries](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/error_boundaries_guide.md) · [Uncontrolled Forms](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/uncontrolled_forms_guide.md) · [Testing](file:///C:/Users/acer/.gemini/antigravity/brain/0746c6b3-cf88-47f2-88f1-4befa963e2e6/vitest_testing_guide.md)*

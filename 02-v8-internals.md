# V8 Internals — How JavaScript Actually Runs

> A developer-friendly breakdown of what happens between writing `const x = 1 + 2` and your CPU executing it.

---

## The Core Problem

Computers only understand binary (zeros and ones). JavaScript is human-readable text. Something has to bridge that gap — that's **V8**, the JavaScript engine built by Google and used in Chrome and Node.js.

---

## The Pipeline: 3 Stages

```
Your JS Code
     ↓
  Parser  →  AST (Abstract Syntax Tree)
     ↓
 Ignition  →  Bytecode  →  executes it
     ↓ (if code is "hot")
 TurboFan  →  Machine Code  →  executes it faster
```

---

### Stage 1 — Parser

The Parser reads your source code and checks whether its structure is valid. Think of it as a grammar checker.

If the structure is valid, it produces an **Abstract Syntax Tree (AST)** — a structured map of your code that the rest of the engine can work with.

```js
const x = 1 + 2;
```

Internally, this becomes a tree of nodes: a variable declaration, containing a binary expression, with two numeric literals. The Parser doesn't execute anything — it just maps.

---

### Stage 2 — Ignition (Interpreter)

Ignition takes the AST and compiles it into **Bytecode** — a simpler, lower-level set of instructions that sits between JavaScript and raw machine code.

Ignition then **executes that Bytecode line by line**.

This starts fast (no heavy optimization upfront) but is not maximally efficient — it's good enough for code that runs once or rarely.

---

### Stage 3 — TurboFan (Optimizing Compiler)

While Ignition is running Bytecode, V8 is watching. It tracks which functions are called frequently — these are called **hot functions**.

When a function becomes hot (e.g. called thousands of times), TurboFan steps in and compiles it directly to **Machine Code** — instructions your CPU can execute natively, with no interpreter overhead.

This process is called **JIT Compilation** (Just-In-Time), because the optimization happens at runtime, not ahead of time.

> **Key distinction:** Ignition handles *all* code. TurboFan only handles *hot* code.

---

## Deoptimization — When TurboFan Gets It Wrong

TurboFan's optimizations are based on **assumptions** it makes by observing how a function has been called so far.

For example, if this function has been called 5,000 times with numbers:

```js
function add(a, b) {
  return a + b;
}

add(1, 2);    // number
add(5, 3);    // number
add(10, 20);  // number
// ... 4,997 more times with numbers
```

TurboFan assumes: *"This function always receives numbers."* It compiles a highly optimized machine code version based on that assumption.

Now this happens:

```js
add("hello", "world"); // string!
```

TurboFan's assumption breaks. It **deoptimizes** — throws away the machine code and falls back to Ignition running the existing Bytecode. The Bytecode was never thrown away, so the fallback is immediate.

---

## Why This Matters in Practice

Deoptimization is a real performance concern. The fix is **type consistency** — don't change the types of values you pass to a function.

### TypeScript helps — but isn't enough

TypeScript catches type mismatches at **compile time** (while you're writing code). But after compilation, everything becomes plain JavaScript. V8 only sees JavaScript at runtime.

This means data coming from external sources can still cause deoptimizations:

```ts
function add(a: number, b: number) {
  return a + b;
}

// TypeScript can't protect you here:
const data = await fetch('/api/values'); // what if the API returns strings?
add(data.a, data.b); // runtime type is unknown
```

### Validate runtime data

This is exactly why runtime validation libraries exist and matter:

```ts
import { z } from 'zod';

const schema = z.object({
  a: z.number(),
  b: z.number(),
});

const data = schema.parse(await fetch('/api/values').then(r => r.json()));
add(data.a, data.b); // now V8's assumption holds
```

Using **Zod** or **Yup** to validate data at the boundary of your application (API responses, form inputs) ensures V8's runtime type assumptions are never broken — which means TurboFan's optimizations stay in place.

---

## Summary Table

| Stage | Input | Output | When |
|---|---|---|---|
| Parser | JS source code | AST | Always, once |
| Ignition | AST | Bytecode + execution | Always |
| TurboFan | Bytecode (hot paths) | Machine Code + execution | Only for hot functions |
| Deoptimization | Type assumption failure | Falls back to Ignition | When types change unexpectedly |

---

## Key Takeaways

- V8 doesn't compile your entire app to machine code upfront — it starts fast and optimizes lazily.
- TurboFan's optimizations are speculative. Break its assumptions and it backs off.
- Type consistency in JavaScript is not just a code style preference — it has a direct impact on runtime performance.
- TypeScript + runtime validation (Zod/Yup) together give you the best protection: compile-time safety and runtime type guarantees.

# JavaScript Objects — Deep Dive Notes

> **Series:** JS Internals Study (Month 1 — Senior Frontend Roadmap)
> **Source:** *You Don't Know JS Yet: Objects & Classes* + personal study notes
> **Goal:** Build a real mental model, not just memorize syntax

---

## 1. Myth: "Everything in JS is an Object"

**False.** JS has two distinct value categories:

| Category | Types |
|----------|-------|
| **Primitives** | `number`, `string`, `boolean`, `null`, `undefined`, `symbol`, `bigint` |
| **Objects** | `{}`, `[]`, `function`, `Map`, `Set`, etc. |

Primitives are **not** objects — but JS temporarily wraps them in object wrappers (boxing) when you call methods on them. More on this in Section 14.

---

## 2. Objects as Containers

An object is a dynamic collection of **key → value** pairs.

```js
const myObj = {
  name: "Shiva",
  age: 25
};
```

Mental model: **objects are containers, not fixed structures.** They can be modified at runtime, inherit behavior via prototype, and often serve as temporary data transport.

**Three main use cases:**
- Store multiple values
- Group related data
- Pass structured data between functions

---

## 3. Defining Properties

### Literal values
```js
const obj = {
  name: "Shiva",
  age: 25
};
```

### Computed expressions (evaluated immediately)
```js
const obj = {
  value: (10 + 5) * 2  // → 30, stored at definition time
};
```

### Lazy values (deferred evaluation)
JS does **not** support lazy evaluation natively in object literals. Use a function instead:

```js
const obj = {
  getValue: () => (10 + 5) * 2  // called when needed, not at creation
};
```

---

## 4. Property Names

Property names are **strings** by default, but JS also allows:

```js
// Numeric keys (internally stored as strings)
const obj = { 42: "number key" };

// Computed keys (evaluated at definition time)
const obj = { ["x" + 10]: true };  // → { x10: true }

// Symbol keys (unique, hidden from normal iteration)
const sym = Symbol("id");
const obj = { [sym]: "secret value" };
```

**Why Symbols?**
- Guaranteed uniqueness — no collisions
- Not visible in `for...in` or `Object.keys()`
- Used for internal/meta properties (e.g., `Symbol.iterator`)

---

## 5. Concise Syntax

### Concise properties (ES6+)
```js
const name = "Shiva";
const obj = { name };  // shorthand for { name: name }
```

### Concise methods
```js
const obj = {
  greet() {
    console.log("Hello");
  },
  *gen() {       // generator method
    yield 1;
  }
};
```

---

## 6. Object Spread

```js
const obj2 = { ...obj1 };
```

**Key rules:**
- Copies own enumerable properties only
- **Shallow copy** — nested objects are shared by reference
- Later properties overwrite earlier ones (left → right)

### The shallow copy trap

```js
const a = { nested: { x: 1 } };
const b = { ...a };

b.nested.x = 99;
console.log(a.nested.x);  // 99 — same reference!
```

> If you need a deep clone: `structuredClone(obj)` (modern) or `JSON.parse(JSON.stringify(obj))` (legacy, no functions/symbols).

---

## 7. Accessing Properties

```js
// Dot notation (most common)
obj.name

// Bracket notation (use when key is dynamic, has spaces, or is computed)
obj["name"]
obj["my key"]  // spaces in key

const key = "name";
obj[key]       // dynamic key
```

---

## 8. `Object.keys / values / entries`

```js
const obj = { a: 1, b: 2 };

Object.keys(obj);    // ["a", "b"]
Object.values(obj);  // [1, 2]
Object.entries(obj); // [["a", 1], ["b", 2]]
```

> All three return **own enumerable** properties only.

---

## 9. Destructuring

Extract values without copying the object:

```js
const obj = { name: "Shiva", age: 25 };

const { name, age } = obj;           // basic
const { name: firstName } = obj;     // rename
const { city = "unknown" } = obj;    // default value
```

> **Important:** Destructuring **extracts** values. It does not clone or copy the object.

---

## 10. Optional Chaining (`?.`)

Safe access through potentially `null`/`undefined` chains:

```js
obj?.address?.city   // undefined if any step is nullish — no crash
obj?.method?.()      // safe method call
arr?.[0]             // safe array access
```

---

## 11. Boxing (Primitive Wrapping)

When you call a method on a primitive, JS temporarily wraps it:

```js
"hello".toUpperCase();   // → "HELLO"
(42).toString();         // → "42"
```

Internally: `new String("hello")` → call method → discard wrapper.

> Primitives are **not** objects. The wrapper is purely temporary.

---

## 12. Assigning Properties

```js
obj.name = "Shiva";  // creates if missing, updates if exists
```

**Caveats:**
- May trigger a setter (if the property has one)
- May silently fail or throw in strict mode (e.g., on frozen objects)

---

## 13. `Object.assign`

Shallow merge into a target:

```js
Object.assign(target, source1, source2);

// Common usage — merge without mutating originals:
const merged = Object.assign({}, obj1, obj2);
```

> Later sources overwrite earlier ones. Same shallow-reference caveat as spread.

---

## 14. Deleting Properties

```js
delete obj.name;      // removes the property entirely
obj.name = undefined; // property still exists, value is undefined
```

| Action | Property still exists? |
|--------|----------------------|
| `= undefined` | ✅ Yes |
| `delete` | ❌ No |

This matters for `"name" in obj` checks.

---

## 15. Checking Property Existence

```js
// Checks own + prototype chain
"name" in obj

// Checks own only (legacy)
obj.hasOwnProperty("name")

// Checks own only (modern, recommended — no prototype pollution risk)
Object.hasOwn(obj, "name")
```

---

## 16. Listing Object Contents

```js
// Own enumerable only
Object.keys(obj)
Object.values(obj)
Object.entries(obj)

// Own enumerable + non-enumerable
Object.getOwnPropertyNames(obj)

// Own Symbol keys
Object.getOwnPropertySymbols(obj)
```

---

## 17. `for...in` — The Gotcha

```js
for (let key in obj) { ... }
```

**Includes inherited properties from the prototype chain.**

Prefer `Object.keys()` + `for...of` when you only want own properties:

```js
for (const key of Object.keys(obj)) { ... }
```

---

## 18. Temporary Container Pattern

Objects are frequently used just to pass structured data — they have no deep meaning beyond being a transport vehicle:

```js
function formatValues({ one, two, three }) {
  return {
    one: one.toUpperCase(),
    two: `--${two}--`,
    three: three.substring(0, 5)
  };
}

const { one, two, three } = formatValues({
  one: "Kyle",
  two: "Simpson",
  three: "getify"
});
```

> The object is not "the thing" — it's just a structured way to move values around.

---

## 19. Quick Reference Cheat Sheet

| Concept | Key point |
|---------|-----------|
| Spread `...` | Shallow copy, later keys win |
| `Object.assign` | Shallow merge, later sources win |
| `delete` | Removes property entirely |
| `= undefined` | Property still exists |
| `in` | Checks prototype chain |
| `Object.hasOwn` | Checks own only (use this) |
| `for...in` | Includes inherited — be careful |
| Destructuring | Extracts values, does not clone |
| Optional chaining `?.` | Safe access, returns `undefined` |
| Boxing | Temporary — primitives are not objects |
| Symbols | Unique keys, hidden from iteration |

---

## 20. Mental Model Summary

Think of objects as:
- **Dynamic key-value maps** (not fixed structures)
- **Mutable at runtime** — add, change, or delete properties anytime
- **Prototype-linked** — they can inherit behavior from other objects
- **Often temporary** — used as transport containers, not as meaningful entities in themselves

> ⚠️ This entire chapter is a foundation for one thing: **the Prototype system**. Without deeply understanding objects, prototype feels like magic. With this — it becomes logical.

---

## Next Up

- `[[Prototype]]` and the prototype chain
- `Object.create()` vs `class`
- `this` binding rules
- Property descriptors (`writable`, `enumerable`, `configurable`)

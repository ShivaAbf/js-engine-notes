# JavaScript Closures

## The One-Sentence Definition

A closure is a function that remembers the variables from the place where it was created.

The most important sentence:

> A function remembers where it was created.

---

# Lexical Environment

Every scope has its own Lexical Environment.

Think of it as a box that stores variables.

```js
let name = "Shiva";
let age = 23;
```

Mental model:

```js
GlobalEnvironment = {
  name: "Shiva",
  age: 23
};
```

This is not real JavaScript code. It is just a way to visualize how variables are stored.

---

# How JavaScript Finds Variables

```js
let message = "Hello";

function sayHi(name) {
  console.log(message, name);
}

sayHi("John");
```

When JavaScript looks for a variable:

1. Check the current scope.
2. If not found, check the outer scope.
3. Continue until reaching the global scope.

For `name`:

```js
{
  name: "John"
}
```

Found immediately.

For `message`:

```js
{
  message: "Hello"
}
```

Found in the outer scope.

---

# Basic Closure Example

```js
function outer() {
  let x = 20;

  return function () {
    console.log(x);
  };
}

const fn = outer();

fn();
```

Output:

```txt
20
```

Why?

Because the returned function remembers the environment where it was created.

---

# The Most Important Rule

Closures store references to variables.

They do NOT store copies of values.

---

Example:

```js
let x = 10;

function outer() {
  return function () {
    console.log(x);
  };
}

const fn = outer();

x = 50;

fn();
```

Output:

```txt
50
```

Not:

```txt
10
```

Because the closure points to the variable itself.

---

# Each Function Call Creates a New Environment

```js
function outer() {
  let x = 20;

  return function () {
    x++;
    console.log(x);
  };
}

const fn1 = outer();
const fn2 = outer();

fn1();
fn1();

fn2();
fn2();
```

Output:

```txt
21
22
21
22
```

Why?

Because every call to `outer()` creates a completely new environment.

Mental model:

```txt
fn1 -> { x: 20 }

fn2 -> { x: 20 }
```

Different variables.

Same initial value.

---

# var vs let in Closures

## var

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```txt
3
3
3
```

Why?

Because there is only ONE variable:

```txt
i
```

All callbacks reference the same variable.

After the loop finishes:

```txt
i = 3
```

Every callback sees:

```txt
3
```

---

## let

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```txt
0
1
2
```

Why?

Because `let` creates a new binding for every iteration.

Mental model:

```txt
Iteration 1 -> i = 0

Iteration 2 -> i = 1

Iteration 3 -> i = 2
```

Each callback gets its own environment.

---

# Every Function Has a Closure

Many people think closures only exist in special cases.

Not true.

```js
let x = 10;

function print() {
  console.log(x);
}
```

This is also a closure.

The function accesses a variable from an outer scope.

---

# Garbage Collection and Closures

Garbage Collection is based on reachability.

Question:

> Can JavaScript still reach this value?

If YES:

```txt
Alive
```

If NO:

```txt
Collected
```

---

Example:

```js
function outer() {
  let x = 10;

  return function () {
    return x;
  };
}

let fn = outer();
```

The environment stays alive because:

```txt
fn
 ↓
Function
 ↓
Environment
 ↓
x
```

`x` is still reachable.

---

Later:

```js
fn = null;
```

Now there is no path to the environment.

Garbage Collector can remove it.

---

# Memory Leak Example

```js
function setup() {
  const hugeArray = new Array(1000000);

  document.body.addEventListener("click", () => {
    console.log(hugeArray.length);
  });
}
```

Even after `setup()` finishes:

```txt
document
 ↓
event listener
 ↓
callback
 ↓
closure
 ↓
hugeArray
```

The array is still reachable.

Therefore it stays in memory.

---

# What To Remember

If you forget everything, remember these four rules:

1. Every scope has a Lexical Environment.
2. A function remembers where it was created.
3. Closures store references to variables, not copies of values.
4. Every function call creates a new environment.

If these four ideas make sense, you understand closures well enough for React, Event Loop, Promises, and async JavaScript.

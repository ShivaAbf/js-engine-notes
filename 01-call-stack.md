# Event Loop in JavaScript

## Basics

JavaScript runs inside the browser and uses a **call stack**.

The call stack executes code **line by line**.

It handles **synchronous (sync) operations**, meaning tasks are executed one by one in order.

---

## Asynchronous Behavior

When we use **asynchronous (async) operations**, execution changes.

JavaScript is a **single-threaded language**, so it cannot run multiple tasks at the same time.

Because of this, async operations are handled outside the call stack.

---

## Web APIs

Some operations are handled by the browser using **Web APIs**, such as:

* DOM events
* `setTimeout`
* `setInterval`

These are executed outside the call stack.

---

## Queues

After async operations complete, their callbacks are placed into queues:

* **Callback Queue (Task Queue)**
* **Microtask Queue**

---

## Microtask Queue (Higher Priority)

The **microtask queue** has higher priority.

Examples include:

* `Promise.then`
* `queueMicrotask`
* `MutationObserver`

---

## Event Loop

The **event loop** continuously checks:

1. Is the **call stack empty?**

If yes:

* First, it processes all tasks in the **microtask queue**
* Then, it processes tasks from the **callback queue**

---

## Call Stack Behavior

* The call stack must be empty before new tasks are executed.
* If it is busy, incoming tasks must wait.

---

## Rendering (Important Part)

The browser is also responsible for:

* **Layout**
* **Paint**

These are part of the **rendering process**.

Rendering is not done randomly. It is coordinated with the event loop.

The browser tries to update the UI roughly every **~16ms (about 60 FPS)**.

---

## Event Loop + Rendering Interaction

* The event loop runs continuously.
* It executes synchronous code first.
* It then processes **microtasks**, followed by **tasks (callbacks)**.
* After that, the browser gets a chance to **render (repaint the UI)**.

Rendering only happens when:

* The call stack is empty
* The browser is allowed to update the screen

---

## Important Behavior

* Microtasks always run **before** tasks (callback queue).
* Rendering does **not happen during execution of JavaScript tasks**.
* Long-running tasks block everything:

  * Event loop
  * Rendering
  * UI updates

---

## Real-World Issue

If the call stack is blocked by heavy computation:

* The UI freezes
* The page cannot re-render
* The application becomes unresponsive

Example: apps like Instagram may freeze when too much work is done on the main thread.

---

## Key Insight

Even if many things are "happening" (timers, promises, events),
JavaScript still executes everything in a **single thread**, coordinated by the **event loop**.

The illusion of concurrency comes from:

* Web APIs
* Queues
* Event loop scheduling

---

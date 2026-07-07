If there is **one topic that almost every Selenium engineer struggles with while learning Playwright**, it is this:

> **"When should I use a Locator, an ElementHandle, or a JSHandle?"**

Understanding this chapter will also make the **Handles** section much easier later.

----------

# Part 13 – Evaluating JavaScript

# Chapter 3 – JSHandle & ElementHandle

----------

# Introduction

In the previous chapter, we learned that `evaluate()` can only return **serializable values**.

For example:

```typescript
const title = await page.evaluate(() => document.title);

```

works because a string can be serialized.

But this doesn't:

```typescript
const button = await page.evaluate(() => {
    return document.querySelector('#save');
});

```

Why?

Because a DOM element cannot be serialized and transferred from the browser process to Node.js.

Instead, Playwright provides **Handles**.

----------

# What is a Handle?

A Handle is a **reference** to an object that exists inside the browser.

Think of it as a remote pointer.

```text
Node.js Test

↓

JSHandle

↓

Browser Object

```

Instead of copying the object,

Playwright gives you a reference to it.

----------

# Types of Handles

Playwright provides two major handle types.

```text
Handle
│
├── JSHandle
│
└── ElementHandle

```

----------

# JSHandle

A **JSHandle** references **any JavaScript object** inside the browser.

Examples:

-   Object
    
-   Array
    
-   Map
    
-   Set
    
-   Window
    
-   Document
    
-   Date
    
-   Custom JavaScript object
    

Example:

```typescript
const handle = await page.evaluateHandle(() => {
    return window.navigator;
});

```

Notice:

```typescript
evaluateHandle()

```

instead of

```typescript
evaluate()

```

----------

# evaluate() vs evaluateHandle()

```typescript
await page.evaluate(...)

```

returns

```text
Serializable Data

```

Whereas

```typescript
await page.evaluateHandle(...)

```

returns

```text
JSHandle

```

----------

## Example

```typescript
const handle = await page.evaluateHandle(() => {
    return {
        username: "admin",
        role: "administrator"
    };
});

```

`handle` is a `JSHandle`, not a plain JavaScript object.

----------

# Reading Values from a JSHandle

Use:

```typescript
const value = await handle.jsonValue();

```

Example:

```typescript
const handle = await page.evaluateHandle(() => {
    return {
        id: 100,
        name: "John"
    };
});

const user = await handle.jsonValue();

console.log(user);

```

Output:

```text
{
  id: 100,
  name: "John"
}

```

----------

# ElementHandle

An **ElementHandle** is a specialized JSHandle that represents a DOM element.

Example:

```typescript
const button = await page.$("#save");

```

or

```typescript
const button = await page.locator("#save").elementHandle();

```

Now:

```text
ElementHandle

↓

<button>

```

inside the browser.

----------

# ElementHandle Hierarchy

```text
JSHandle
    │
    └── ElementHandle

```

Every ElementHandle is also a JSHandle.

----------

# Interacting with ElementHandle

```typescript
const button = await page.$("#save");

await button?.click();

```

It works.

But...

This is **not the recommended approach**.

----------

# Locator vs ElementHandle

This is one of the most important comparisons.

Locator

ElementHandle

Lazy

Immediate

Auto-waits

No automatic re-query

Re-resolves element

Fixed reference

Retryable

Can become stale

Recommended

Legacy/advanced

----------

# Why Locators are Better

Suppose React re-renders a button.

```text
Button

↓

Removed

↓

New Button Created

```

A Locator:

```typescript
await page.locator("#save").click();

```

finds the new button automatically.

An ElementHandle:

```typescript
const button = await page.$("#save");

await button?.click();

```

may now reference a detached element.

----------

# Visual Comparison

```text
Locator

↓

Find Element

↓

Perform Action

↓

Done

```

versus

```text
ElementHandle

↓

Find Element

↓

Store Reference

↓

DOM Changes

↓

Reference Invalid

```

----------

# Using JSHandle in `evaluate()`

A handle can be passed back into the browser.

Example:

```typescript
const body = await page.evaluateHandle(() => {
    return document.body;
});

const tagName = await page.evaluate((element) => {
    return element.tagName;
}, body);

console.log(tagName);

```

Output:

```text
BODY

```

Notice that we're passing a **handle**, not serializing the DOM element.

----------

# locator.evaluate()

Modern Playwright often removes the need for handles.

Instead of:

```typescript
const handle = await page.$("#username");

const value = await page.evaluate((el) => {
    return el.value;
}, handle);

```

Use:

```typescript
const value = await page.locator("#username").evaluate(
    element => (element as HTMLInputElement).value
);

```

This is shorter, clearer, and retains locator benefits.

----------

# locator.evaluateAll()

Useful for lists.

Example:

```typescript
const names = await page.locator(".product").evaluateAll(
    elements =>
        elements.map(element => element.textContent)
);

```

Output:

```text
[
  "Laptop",
  "Phone",
  "Keyboard"
]

```

----------

# evaluateHandle() with Browser Objects

```typescript
const storage = await page.evaluateHandle(() => {
    return window.localStorage;
});

```

This returns a JSHandle pointing to Local Storage.

----------

# Accessing Properties

You can retrieve properties from a handle.

```typescript
const handle = await page.evaluateHandle(() => {
    return window.navigator;
});

const languageHandle = await handle.getProperty("language");

console.log(await languageHandle.jsonValue());

```

Output:

```text
en-US

```

----------

# Handle Lifecycle

Handles consume browser resources.

```text
Create Handle

↓

Use Handle

↓

Dispose Handle

↓

Memory Released

```

----------

# Disposing a Handle

```typescript
await handle.dispose();

```

Always dispose long-lived handles when they're no longer needed.

----------

# Real-World Example – Canvas

Canvas elements often require JavaScript execution.

```typescript
const canvas = await page.locator("canvas").elementHandle();

const width = await page.evaluate((c) => {
    return c.width;
}, canvas);

console.log(width);

```

----------

# Real-World Example – Reading a Complex Object

```typescript
const appState = await page.evaluateHandle(() => {
    return window.appState;
});

const json = await appState.jsonValue();

console.log(json);

```

----------

# Real-World Example – Extracting Table Data

Recommended approach:

```typescript
const rows = await page.locator("table tr").evaluateAll(
    elements =>
        elements.map(row => row.textContent?.trim())
);

console.log(rows);

```

No handles required.

----------

# page.$() vs Locator

Older style:

```typescript
const button = await page.$("#save");

```

Modern Playwright:

```typescript
await page.locator("#save").click();

```

The Locator API is preferred because it is more resilient and benefits from Playwright's auto-waiting.

----------

# JSHandle vs ElementHandle

JSHandle

ElementHandle

Any JavaScript object

DOM element only

Returned by `evaluateHandle()`

Returned by `page.$()` or `elementHandle()`

Can reference arrays, objects, maps

Represents HTML/SVG elements

Base class

Derived class

----------

# Locator vs JSHandle

Locator

JSHandle

User interactions

JavaScript objects

Auto-waiting

No auto-waiting

Re-resolves elements

Fixed reference

Recommended for tests

Advanced browser interactions

----------

# When Should You Use Handles?

Good use cases:

-   Accessing complex browser-side objects
    
-   Working with Canvas or WebGL
    
-   Passing DOM elements into `evaluate()`
    
-   Advanced framework utilities
    
-   Low-level browser APIs
    

Most everyday UI automation should use Locators instead.

----------

# Best Practices

-   Prefer Locators over ElementHandles for UI interactions.
    
-   Use `locator.evaluate()` or `locator.evaluateAll()` when possible.
    
-   Dispose of long-lived handles after use.
    
-   Use `evaluateHandle()` only when you genuinely need a reference to a browser object.
    
-   Avoid storing ElementHandles across DOM updates.
    

----------

# Common Mistakes

### ❌ Using `page.$()` everywhere

```typescript
const button = await page.$("#save");
await button?.click();

```

Prefer:

```typescript
await page.locator("#save").click();

```

----------

### ❌ Forgetting to dispose handles

Long-lived handles can consume browser resources.

----------

### ❌ Keeping ElementHandles after DOM updates

React, Angular, and Vue frequently replace DOM elements. A stored ElementHandle may no longer point to a valid element.

----------

### ❌ Using handles when `locator.evaluate()` is sufficient

Prefer the simpler, locator-based approach unless a true handle is required.

----------

# Interview Questions

### Q1. What is a JSHandle?

A JSHandle is a reference to a JavaScript object that exists inside the browser process.

----------

### Q2. What is an ElementHandle?

An ElementHandle is a specialized JSHandle that represents a DOM element.

----------

### Q3. What is the difference between `evaluate()` and `evaluateHandle()`?

-   `evaluate()` returns serializable values.
    
-   `evaluateHandle()` returns a JSHandle that references a browser object.
    

----------

### Q4. Why are Locators preferred over ElementHandles?

Locators automatically re-query elements, support auto-waiting, and are resilient to DOM updates. ElementHandles reference a specific element instance that may become detached.

----------

### Q5. When should you use `locator.evaluate()`?

When you need to execute JavaScript against an element while still benefiting from Locator semantics and automatic element resolution.

----------

# Summary

Handles provide references to objects that live inside the browser process. `JSHandle` can reference any JavaScript object, while `ElementHandle` specializes in DOM elements. Although handles remain valuable for advanced scenarios, modern Playwright applications should generally favor Locators and APIs like `locator.evaluate()` for better reliability, readability, and resilience to DOM changes.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMDI3NTI0NDVdfQ==
-->
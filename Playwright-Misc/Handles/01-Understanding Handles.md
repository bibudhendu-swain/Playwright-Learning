Now we enter the **Handles** section.

Since we've already introduced **JSHandle** and **ElementHandle** in the previous section, this chapter will focus on **how Handles work internally**, their lifecycle, APIs, and why Playwright now recommends **Locators** for most test automation.

This chapter is much deeper and will help readers understand **when Handles are appropriate and when they should be avoided**.

----------

# Part 12 – Handles

# Chapter 1 – Understanding Handles

----------

# Introduction

In the previous section, we learned that JavaScript objects inside the browser cannot always be serialized and returned to Node.js.

For example:

```typescript
const title = await page.evaluate(() => document.title);

```

works because a string is serializable.

But this does not:

```typescript
await page.evaluate(() => {
    return document.body;
});

```

Instead of copying browser objects, Playwright creates a **Handle**.

A Handle is simply a **reference** to an object living inside the browser process.

----------

# What is a Handle?

Think of a Handle as a remote pointer.

```text
Node.js Test
      │
      ▼
 Handle
      │
      ▼
Browser Object

```

The object remains in the browser.

Node.js only keeps a reference to it.

Unlike `evaluate()`, no serialization happens.

----------

# Why Handles Exist

Imagine a JavaScript object like this:

```javascript
window.shoppingCart = {
    total: 2500,
    items: 5
};

```

Returning this repeatedly using `evaluate()` means copying the object every time.

With a Handle:

```text
Browser Object

↓

JSHandle

↓

Reuse Multiple Times

```

The browser object stays where it is.

----------

# Handle Hierarchy

Playwright exposes two major handle types.

```text
JSHandle
    │
    └── ElementHandle

```

Everything starts with a JSHandle.

----------

# JSHandle

Represents any JavaScript object.

Examples:

-   Object
    
-   Array
    
-   Date
    
-   Map
    
-   Set
    
-   Window
    
-   Navigator
    
-   Local Storage
    
-   Custom application objects
    

Example

```typescript
const handle =
await page.evaluateHandle(() => {

    return window.navigator;

});

```

The returned object is **not** the Navigator.

It is a reference to it.

----------

# ElementHandle

Represents a DOM element.

Example

```typescript
const button =
await page.$("#save");

```

Internally

```text
<button>

↓

ElementHandle

```

----------

# JSHandle vs ElementHandle

JSHandle

ElementHandle

Any JavaScript object

DOM element

Base class

Derived class

Returned by `evaluateHandle()`

Returned by `page.$()` or `locator.elementHandle()`

Browser objects

HTML/SVG elements

----------

# Creating Handles

## Using evaluateHandle()

```typescript
const handle =
await page.evaluateHandle(() => {

    return {

        id: 1,

        name: "Playwright"

    };

});

```

Returns:

```text
JSHandle

```

----------

## Creating ElementHandle

```typescript
const input =
await page.locator("#username")
.elementHandle();

```

or

```typescript
const input =
await page.$("#username");

```

The Locator approach is preferred if you genuinely need an ElementHandle.

----------

# Reading Handle Values

Handles aren't plain objects.

Use:

```typescript
await handle.jsonValue();

```

Example

```typescript
const handle =
await page.evaluateHandle(() => {

    return {

        name: "John"

    };

});

const value =
await handle.jsonValue();

console.log(value);

```

----------

# Accessing Object Properties

Instead of transferring the whole object:

```typescript
const language =
await handle.getProperty("language");

```

Example

```typescript
const navigator =
await page.evaluateHandle(() => {

    return navigator;

});

const language =
await navigator.getProperty("language");

console.log(

    await language.jsonValue()

);

```

----------

# Evaluating Using a Handle

A Handle can be passed back into browser JavaScript.

```typescript
const body =
await page.evaluateHandle(() => {

    return document.body;

});

const tag =
await page.evaluate(element => {

    return element.tagName;

}, body);

console.log(tag);

```

Output

```text
BODY

```

----------

# Working with ElementHandle

Example

```typescript
const button =
await page.$("#save");

await button?.click();

```

Although valid, modern Playwright prefers:

```typescript
await page
.locator("#save")
.click();

```

----------

# Handle Lifecycle

Handles consume browser resources.

Lifecycle:

```text
Create Handle
      │
      ▼
Use Handle
      │
      ▼
Dispose Handle
      │
      ▼
Memory Released

```

----------

# Disposing a Handle

```typescript
await handle.dispose();

```

This tells Playwright the browser object is no longer needed.

----------

# Why Disposal Matters

Suppose a utility creates hundreds of handles:

```text
Handle 1

Handle 2

Handle 3

...

Handle 500

```

If they remain referenced, they consume browser-side memory until released.

In normal short-lived tests, Playwright cleans them up when the page or context closes, but long-running utilities should dispose of handles explicitly.

----------

# Passing Handles Between Evaluations

```typescript
const handle =
await page.evaluateHandle(() => {

    return window.appState;

});

const result =
await page.evaluate(state => {

    return state.user.name;

}, handle);

```

No serialization of the entire object occurs.

----------

# Handles as Parameters

Handles can be mixed with normal values.

```typescript
const body =
await page.evaluateHandle(() => {

    return document.body;

});

const info =
await page.evaluate(

(data) => {

    return {

        tag: data.element.tagName,

        label: data.text

    };

},

{

    element: body,

    text: "Root Element"

}

);

```

----------

# Handle vs Serialization

### Using evaluate()

```text
Object

↓

Serialize

↓

Copy

↓

Node.js

```

----------

### Using Handle

```text
Object

↓

Reference

↓

Browser Object Remains

```

----------

# Real-World Example – Accessing Application State

Many modern applications expose global state.

```typescript
const state =
await page.evaluateHandle(() => {

    return window.appState;

});

const version =
await page.evaluate(app => {

    return app.version;

}, state);

expect(version).toBe("2.1");

```

----------

# Real-World Example – Canvas

Canvas APIs often require JavaScript execution.

```typescript
const canvas =
await page.locator("canvas")
.elementHandle();

const width =
await page.evaluate(

element => element.width,

canvas

);

console.log(width);

```

----------

# Real-World Example – Reading Navigator

```typescript
const navigatorHandle =
await page.evaluateHandle(() => navigator);

const platform =
await navigatorHandle.getProperty("platform");

console.log(
await platform.jsonValue()
);

```

----------

# Handle APIs

API

Purpose

`jsonValue()`

Convert handle to serializable value

`getProperty()`

Read a property

`dispose()`

Release browser resources

`evaluate()`

Execute code using the handle

`evaluateHandle()`

Return another handle

----------

# When Should You Use Handles?

Good candidates:

-   Canvas APIs
    
-   Complex browser-side objects
    
-   Large JavaScript structures
    
-   Custom browser utilities
    
-   Framework development
    
-   Low-level browser interactions
    

Not for routine UI automation.

----------

# Locator vs Handle

Locator

Handle

User interaction

Browser object reference

Auto-waiting

No automatic re-query

Retryable

Fixed object reference

Preferred for tests

Advanced scenarios

----------

# Best Practices

-   Prefer Locators for UI interactions.
    
-   Use `evaluateHandle()` only when you need to keep a browser object alive across multiple operations.
    
-   Dispose of long-lived handles explicitly.
    
-   Avoid storing ElementHandles across DOM updates.
    
-   Use `locator.evaluate()` when you only need to execute JavaScript against an element once.
    

----------

# Common Mistakes

### ❌ Using Handles for every element

```typescript
const button =
await page.$("#save");

await button?.click();

```

Prefer:

```typescript
await page
.locator("#save")
.click();

```

----------

### ❌ Forgetting to dispose long-lived handles

This can increase browser memory usage in long-running utilities.

----------

### ❌ Treating a Handle like a normal JavaScript object

A Handle is only a reference. Use APIs like `jsonValue()`, `getProperty()`, or `evaluate()` to interact with the underlying object.

----------

### ❌ Keeping ElementHandles after the DOM changes

Modern frameworks frequently replace DOM elements during rendering. A Locator automatically resolves the current element; an ElementHandle does not.

----------

# Interview Questions

### Q1. What is a Handle in Playwright?

A Handle is a reference to an object that exists inside the browser process. It allows Playwright to interact with browser-side objects without serializing them.

----------

### Q2. What is the difference between `JSHandle` and `ElementHandle`?

-   `JSHandle` can reference any JavaScript object.
    
-   `ElementHandle` is a specialized `JSHandle` that references a DOM element.
    

----------

### Q3. Why should Locators usually be preferred over ElementHandles?

Locators automatically re-query elements, support auto-waiting, and are resilient to DOM updates, making them more reliable for UI automation.

----------

### Q4. Why should long-lived handles be disposed?

To release browser-side resources and prevent unnecessary memory usage in long-running sessions.

----------

### Q5. When are Handles useful?

They are most useful for advanced browser interactions, such as working with Canvas, complex JavaScript objects, custom browser utilities, or framework-level tooling.

----------

# Summary

Handles provide references to browser-side objects without copying them into the Node.js test process. `JSHandle` represents any JavaScript object, while `ElementHandle` specializes in DOM elements. Although modern Playwright tests should rely primarily on Locators, understanding Handles is essential for advanced browser automation, JavaScript evaluation, and framework development.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ3NTQwMjk2M119
-->
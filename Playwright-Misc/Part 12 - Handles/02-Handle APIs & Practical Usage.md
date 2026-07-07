This chapter will be the **practical API reference** for Handles.

The previous chapter explained **what Handles are**. This chapter explains **how to use them effectively**, what APIs are available, and, equally important, **which APIs are considered legacy and what the modern Playwright alternatives are**.

-   **✅ Recommended (Modern Playwright)**
    
-   **⚠️ Legacy / Advanced (Use only when needed)**
    

That distinction helps readers write idiomatic Playwright code instead of carrying over Selenium habits.

----------

# Part 12 – Handles

# Chapter 2 – Handle APIs & Practical Usage

----------

# Introduction

A Handle is a reference to an object inside the browser.

Once you obtain a handle, Playwright provides several APIs to:

-   Read its value
    
-   Access properties
    
-   Execute JavaScript against it
    
-   Convert between handle types
    
-   Dispose of it
    

We'll look at each of these APIs.

----------

# Handle API Overview

## JSHandle APIs

API

Purpose

`jsonValue()`

Convert handle to a serializable value

`getProperty()`

Get a property as another JSHandle

`evaluate()`

Execute JavaScript using the handle

`evaluateHandle()`

Return another handle

`asElement()`

Convert to ElementHandle (if applicable)

`dispose()`

Release browser resources

----------

## ElementHandle APIs

Because `ElementHandle` extends `JSHandle`, it inherits all JSHandle APIs.

Additionally, it supports element-specific operations such as:

-   `click()`
    
-   `fill()`
    
-   `hover()`
    
-   `screenshot()`
    
-   `boundingBox()`
    
-   `scrollIntoViewIfNeeded()`
    

> **Recommendation:** Although these methods work, prefer performing actions through a `Locator` whenever possible.

----------

# jsonValue()

## Purpose

Converts a JSHandle into a normal JavaScript value.

Example

```typescript
const handle = await page.evaluateHandle(() => {
    return {
        id: 1,
        name: "Playwright"
    };
});

const value = await handle.jsonValue();

console.log(value);

```

Output

```text
{
  id: 1,
  name: "Playwright"
}

```

----------

# When Should You Use `jsonValue()`?

Useful when you need to bring browser-side data into your Node.js test.

Examples:

-   Application state
    
-   Configuration
    
-   JSON objects
    
-   Arrays
    

----------

# getProperty()

Instead of transferring an entire object,

retrieve only the property you need.

```typescript
const navigatorHandle =
await page.evaluateHandle(() => navigator);

const language =
await navigatorHandle.getProperty("language");

console.log(
await language.jsonValue()
);

```

Output

```text
en-US

```

----------

# Chaining getProperty()

Example

```typescript
const app =
await page.evaluateHandle(() => window.appState);

const user =
await app.getProperty("user");

const name =
await user.getProperty("name");

console.log(
await name.jsonValue()
);

```

----------

# JSHandle.evaluate()

You can execute JavaScript directly against a handle.

Example

```typescript
const body =
await page.evaluateHandle(() => document.body);

const tagName =
await body.evaluate(element => element.tagName);

console.log(tagName);

```

Output

```text
BODY

```

Notice:

No need to call `page.evaluate()` separately.

----------

# JSHandle.evaluateHandle()

Returns another Handle.

Example

```typescript
const body =
await page.evaluateHandle(() => document.body);

const children =
await body.evaluateHandle(element => element.children);

```

Result:

```text
JSHandle

```

----------

# asElement()

Sometimes you're not sure whether a JSHandle represents a DOM element.

Example

```typescript
const handle =
await page.evaluateHandle(() => document.body);

const element =
handle.asElement();

```

If it represents an element:

```text
ElementHandle

```

Otherwise

```text
null

```

----------

# dispose()

Always release long-lived handles.

```typescript
await handle.dispose();

```

Example

```typescript
const appState =
await page.evaluateHandle(() => window.appState);

// Work with the handle...

await appState.dispose();

```

----------

# ElementHandle APIs

Although ElementHandles support many interaction methods,

modern Playwright recommends using Locators instead.

----------

# click()

```typescript
const button =
await page.locator("#save").elementHandle();

await button?.click();

```

Preferred

```typescript
await page.locator("#save").click();

```

----------

# fill()

Legacy style

```typescript
const input =
await page.$("#username");

await input?.fill("admin");

```

Modern style

```typescript
await page
.locator("#username")
.fill("admin");

```

----------

# hover()

```typescript
const menu =
await page.locator(".menu").elementHandle();

await menu?.hover();

```

Better

```typescript
await page
.locator(".menu")
.hover();

```

----------

# screenshot()

Sometimes useful.

```typescript
const logo =
await page.locator("#logo").elementHandle();

await logo?.screenshot({
    path: "logo.png"
});

```

Locator alternative

```typescript
await page
.locator("#logo")
.screenshot({
    path: "logo.png"
});

```

----------

# boundingBox()

Returns an element's position and dimensions.

```typescript
const box =
await page
.locator("#login")
.boundingBox();

console.log(box);

```

Output

```text
{
  x: 120,
  y: 220,
  width: 200,
  height: 50
}

```

Useful for:

-   Visual testing
    
-   Canvas interactions
    
-   Drag-and-drop calculations
    

----------

# scrollIntoViewIfNeeded()

Ensures the element is visible before interaction.

```typescript
const element =
await page
.locator("#footer")
.elementHandle();

await element?.scrollIntoViewIfNeeded();

```

Locator alternative

```typescript
await page
.locator("#footer")
.scrollIntoViewIfNeeded();

```

----------

# locator.elementHandle()

Sometimes you genuinely need an ElementHandle.

```typescript
const handle =
await page
.locator("#username")
.elementHandle();

```

Useful when passing the element into `evaluate()`.

----------

# locator.elementHandles()

Returns multiple ElementHandles.

```typescript
const rows =
await page
.locator("table tr")
.elementHandles();

```

Returns:

```text
ElementHandle[]

```

----------

# Example

```typescript
const rows =
await page
.locator("table tr")
.elementHandles();

for (const row of rows) {

    console.log(
        await row.textContent()
    );

}

```

> **Modern Alternative:** If you're only extracting data, `locator.evaluateAll()` or locator assertions are often simpler and more resilient.

----------

# Handle APIs vs Locator APIs

Handle API

Preferred Locator Alternative

`click()`

`locator.click()`

`fill()`

`locator.fill()`

`hover()`

`locator.hover()`

`check()`

`locator.check()`

`screenshot()`

`locator.screenshot()`

`scrollIntoViewIfNeeded()`

`locator.scrollIntoViewIfNeeded()`

----------

# Real-World Example – Canvas

Canvas requires an ElementHandle.

```typescript
const canvas =
await page
.locator("canvas")
.elementHandle();

const width =
await canvas?.evaluate(
    element => element.width
);

console.log(width);

```

----------

# Real-World Example – Large Application State

```typescript
const state =
await page.evaluateHandle(() => window.appState);

const version =
await state.getProperty("version");

console.log(
await version.jsonValue()
);

await state.dispose();

```

----------

# Performance Considerations

Good

```text
One Handle

↓

Reuse Several Times

↓

Dispose

```

Bad

```text
Create Handle

↓

Never Dispose

↓

Memory Grows

```

----------

# Migration from ElementHandle to Locator

Old

```typescript
const button =
await page.$("#save");

await button?.click();

```

Modern

```typescript
await page
.locator("#save")
.click();

```

----------

# When Are Handles Still Useful?

Handles remain the right choice for:

-   Canvas
    
-   WebGL
    
-   Complex browser-side JavaScript objects
    
-   Framework development
    
-   Custom Playwright utilities
    
-   Browser object inspection
    

Routine UI automation should favor Locators.

----------

# Best Practices

-   Prefer Locator APIs for UI interactions.
    
-   Use `jsonValue()` only when you need browser-side data in Node.js.
    
-   Retrieve individual properties with `getProperty()` instead of transferring entire objects.
    
-   Dispose of long-lived handles.
    
-   Use `locator.elementHandle()` only when another API specifically requires an ElementHandle.
    

----------

# Common Mistakes

### ❌ Using `page.$()` instead of Locators

Modern Playwright code should generally start with `page.locator()`.

----------

### ❌ Retrieving entire objects unnecessarily

If you only need one property, use `getProperty()` instead of `jsonValue()` on the entire object.

----------

### ❌ Holding ElementHandles after a page re-render

A Locator can recover from DOM updates; an ElementHandle cannot.

----------

### ❌ Forgetting that Handle methods don't automatically re-query the DOM

If the referenced element is removed, the handle may no longer be usable.

----------

# Interview Questions

### Q1. What does `jsonValue()` do?

It converts a JSHandle into a serializable JavaScript value that can be used in the Node.js test.

----------

### Q2. What is the purpose of `getProperty()`?

It retrieves a property from a browser-side object as another JSHandle without transferring the entire object.

----------

### Q3. What does `asElement()` return?

If the JSHandle references a DOM element, it returns an `ElementHandle`; otherwise, it returns `null`.

----------

### Q4. Why is `locator.elementHandle()` rarely needed?

Because most interactions can be performed directly through Locator APIs, which provide auto-waiting and resilience to DOM updates.

----------

### Q5. When is an ElementHandle still appropriate?

When an API specifically requires a browser-side element reference, such as certain advanced JavaScript evaluation scenarios, Canvas manipulation, or framework-level utilities.

----------

# Summary

Handles provide low-level access to browser-side objects, but modern Playwright encourages using Locators for most automation tasks. APIs such as `jsonValue()`, `getProperty()`, `evaluate()`, and `dispose()` allow you to inspect and manage browser objects efficiently, while Locator APIs remain the preferred choice for reliable user interactions. Understanding both approaches helps you choose the right abstraction for each situation.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTc0NTk2MjgyXX0=
-->
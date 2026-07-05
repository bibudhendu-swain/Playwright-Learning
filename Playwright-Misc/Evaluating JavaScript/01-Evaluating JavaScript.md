# Part 11 – Evaluating JavaScript

This is one of the most misunderstood APIs because people often think:

> "Why do I need `page.evaluate()`? I can already use locators."

The answer is:

👉 **Locators interact with the page.**  
👉 **`evaluate()` executes JavaScript inside the browser.**

This chapter will explain exactly when and why to use it.

----------

# Part 11 – Evaluating JavaScript

# Chapter 1 – JavaScript Evaluation Fundamentals

----------

# Introduction

Normally, Playwright controls the browser **from outside**.

When you write:

```typescript
await page.getByRole('button', { name: 'Login' }).click();

```

Playwright sends commands to the browser.

However, sometimes interacting with the page isn't enough.

You may want to:

-   Read Local Storage
    
-   Read Session Storage
    
-   Access `window`
    
-   Access `document`
    
-   Read global JavaScript variables
    
-   Execute custom JavaScript
    
-   Calculate values inside the page
    
-   Query browser APIs
    

This is where JavaScript evaluation comes in.

----------

# What is `page.evaluate()`?

`page.evaluate()` executes JavaScript **inside the browser page**.

Think of it like opening DevTools and typing code into the Console.

For example, if you type this in Chrome DevTools:

```javascript
document.title

```

The Playwright equivalent is:

```typescript
const title = await page.evaluate(() => {
    return document.title;
});

```

----------

# Two Different JavaScript Environments

This is the **single most important concept**.

Playwright code and browser code do **not** run in the same environment.

```
+--------------------------------------+
|      Node.js (Your Test)             |
|                                      |
| const value = 10;                    |
| await page.evaluate(...);            |
+-------------------▲------------------+
                    │
                    │ Communication
                    ▼
+--------------------------------------+
|      Browser (Web Page)              |
|                                      |
| document                            |
| window                              |
| localStorage                        |
| sessionStorage                      |
| navigator                           |
+--------------------------------------+

```

----------

# Example

This works:

```typescript
const title = await page.evaluate(() => {
    return document.title;
});

console.log(title);

```

Because `document` exists inside the browser.

----------

This does **not** work:

```typescript
console.log(document.title);

```

Why?

Because your Playwright test runs in Node.js, where there is no `document`.

----------

# Basic Syntax

```typescript
const result = await page.evaluate(() => {
    // Browser JavaScript

    return something;
});

```

----------

# Example – Get Page Title

```typescript
const title = await page.evaluate(() => {
    return document.title;
});

console.log(title);

```

Equivalent to running:

```javascript
document.title

```

inside DevTools.

----------

# Example – Get Current URL

```typescript
const url = await page.evaluate(() => {
    return window.location.href;
});

console.log(url);

```

----------

# Example – Get Domain

```typescript
const hostname = await page.evaluate(() => {
    return location.hostname;
});

```

----------

# Reading Local Storage

One of the most common real-world examples.

```typescript
const token = await page.evaluate(() => {
    return localStorage.getItem("token");
});

```

Useful for:

-   Authentication
    
-   JWT validation
    
-   API testing
    

----------

# Reading Session Storage

```typescript
const session = await page.evaluate(() => {
    return sessionStorage.getItem("user");
});

```

----------

# Writing Local Storage

```typescript
await page.evaluate(() => {
    localStorage.setItem("theme", "dark");
});

```

----------

# Removing Local Storage

```typescript
await page.evaluate(() => {
    localStorage.removeItem("token");
});

```

----------

# Clearing Storage

```typescript
await page.evaluate(() => {
    localStorage.clear();
});

```

----------

# Reading Cookies

Most cookie operations should use Playwright's BrowserContext API.

However, browser JavaScript can still read non-HTTP-only cookies.

```typescript
const cookies = await page.evaluate(() => {
    return document.cookie;
});

```

----------

# Reading Window Variables

Suppose the application defines:

```javascript
window.appVersion = "2.5";

```

Read it:

```typescript
const version = await page.evaluate(() => {
    return window.appVersion;
});

```

----------

# Reading Document Values

```typescript
const language = await page.evaluate(() => {
    return document.documentElement.lang;
});

```

----------

# Querying Elements

Although locators are preferred, you can use DOM APIs.

```typescript
const count = await page.evaluate(() => {
    return document.querySelectorAll(".product").length;
});

```

----------

# Getting Text

```typescript
const heading = await page.evaluate(() => {
    return document.querySelector("h1")?.textContent;
});

```

----------

# Executing Multiple Statements

```typescript
const result = await page.evaluate(() => {

    const products =
        document.querySelectorAll(".product");

    return products.length;

});

```

----------

# Returning Objects

```typescript
const data = await page.evaluate(() => {

    return {

        title: document.title,

        url: location.href,

        language: navigator.language

    };

});

console.log(data);

```

Output:

```text
{
  title: "...",
  url: "...",
  language: "en-US"
}

```

----------

# Returning Arrays

```typescript
const names = await page.evaluate(() => {

    return [...document.querySelectorAll(".name")]

        .map(element => element.textContent);

});

```

----------

# Returning Numbers

```typescript
const width = await page.evaluate(() => {
    return window.innerWidth;
});

```

----------

# Returning Boolean

```typescript
const online = await page.evaluate(() => {
    return navigator.onLine;
});

```

----------

# Reading Browser Information

```typescript
const browserInfo = await page.evaluate(() => {

    return {

        language: navigator.language,

        platform: navigator.platform,

        userAgent: navigator.userAgent

    };

});

```

----------

# Real-World Example – JWT Token

```typescript
const jwt = await page.evaluate(() => {

    return localStorage.getItem("jwt");

});

expect(jwt).not.toBeNull();

```

----------

# Real-World Example – Dark Mode

```typescript
const theme = await page.evaluate(() => {

    return localStorage.getItem("theme");

});

expect(theme).toBe("dark");

```

----------

# Real-World Example – Shopping Cart

```typescript
const cartItems = await page.evaluate(() => {

    return JSON.parse(

        localStorage.getItem("cart")!

    );

});

expect(cartItems.length).toBe(3);

```

----------

# `evaluate()` vs Locators

Many engineers ask:

Which should I use?

Use Locator

Use evaluate()

Click

Read Local Storage

Fill

Access `window`

Hover

Read `navigator`

Assertions

Execute JavaScript

User interactions

Browser APIs

----------

# Best Practices

✅ Prefer locators for user interactions.

✅ Use `evaluate()` for browser APIs.

✅ Keep evaluated JavaScript small and focused.

✅ Return only the data you need.

✅ Avoid using `evaluate()` to bypass Playwright features such as locators or assertions.

----------

# Common Mistakes

## ❌ Trying to access `document` directly

```typescript
console.log(document.title);

```

This fails because your test runs in Node.js.

Correct:

```typescript
const title = await page.evaluate(() => document.title);

```

----------

## ❌ Using `evaluate()` for clicks

```typescript
await page.evaluate(() => {
    document.querySelector("#save")?.click();
});

```

Better:

```typescript
await page.getByRole("button", {
    name: "Save"
}).click();

```

This benefits from Playwright's auto-waiting and actionability checks.

----------

## ❌ Returning DOM Elements

```typescript
const element = await page.evaluate(() => {
    return document.querySelector("#username");
});

```

DOM elements cannot be returned directly because they are not serializable. We'll discuss alternatives in the next chapter.

----------

# Interview Questions

### Q1. What is `page.evaluate()`?

It executes JavaScript inside the browser page rather than in the Node.js test environment.

----------

### Q2. Why can't you access `document` directly from a Playwright test?

Because Playwright tests run in Node.js, where browser globals such as `document` and `window` do not exist.

----------

### Q3. When should you use `evaluate()` instead of a locator?

Use `evaluate()` when interacting with browser APIs or executing custom JavaScript, such as reading Local Storage, Session Storage, or `window` properties. Use locators for user interactions.

----------

### Q4. Can `evaluate()` return objects and arrays?

Yes. It can return serializable values such as strings, numbers, booleans, arrays, plain objects, and `null`.

----------

### Q5. Should you use `evaluate()` to click buttons?

No. Prefer Playwright locators because they provide auto-waiting, actionability checks, and clearer intent.

----------

# Summary

`page.evaluate()` bridges the gap between your Node.js test and the browser's JavaScript environment. It allows you to execute code inside the page to access browser APIs, application state, and global objects that are otherwise unavailable from your test. Used appropriately, it complements Playwright's locator-based automation rather than replacing it.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkyNjQ0MjEyMV19
-->
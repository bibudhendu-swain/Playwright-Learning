This is **the foundation of all API mocking in Playwright**.

If someone understands **Route Interception**, they can mock almost anything.

Everything we do in the remaining chapters—mocking JSON, GraphQL, HAR, modifying requests—starts with **routes**.

----------

# Part 16 – Mock APIs

# Chapter 2 – Route Interception

----------

# Introduction

When a browser communicates with a server, every network request follows a predictable path.

Normally:

```text
Browser
    │
    ▼
Internet
    │
    ▼
Server
    │
    ▼
Response
    │
    ▼
Browser

```

Playwright inserts itself into this pipeline.

```text
Browser
    │
    ▼
Playwright Route
    │
    ├── Continue Request
    ├── Modify Request
    ├── Mock Response
    └── Abort Request
    │
    ▼
Server (Optional)

```

This interception mechanism is called **Route Interception**.

----------

# What is Route Interception?

Route Interception allows Playwright to intercept **every outgoing network request** before it reaches the server.

Once intercepted, you can decide whether to:

-   Continue the request
    
-   Modify the request
    
-   Return your own response
    
-   Abort the request
    

Think of it as a traffic controller.

```text
Incoming Request

        │

        ▼

   Route Handler

   ├── Continue

   ├── Fulfill

   ├── Abort

   └── Modify

```

----------

# Route Interception APIs

Playwright provides two APIs.

API

Scope

`page.route()`

Current page only

`browserContext.route()`

Every page inside the browser context

----------

# page.route()

Most commonly used.

Syntax

```typescript
await page.route(
    url,
    async route => {

    }
);

```

Example

```typescript
await page.route("**/products", async route => {

    await route.continue();

});

```

Now every request matching:

```text
/products

```

is intercepted.

----------

# browserContext.route()

Sometimes your application opens:

-   Popups
    
-   New Tabs
    
-   Multiple Pages
    

Instead of registering routes on every page:

```typescript
await context.route(
    "**/products",
    async route => {

        await route.continue();

    }
);

```

Now every page inside the context uses the route.

----------

# page.route() vs browserContext.route()

page.route()

browserContext.route()

Current page

Entire browser context

Simpler

Better for framework-level setup

Single-tab tests

Multi-tab applications

----------

# Route Matching

Playwright only intercepts matching URLs.

Example

```typescript
await page.route(
    "**/products",
    async route => {

        await route.continue();

    }
);

```

Intercepts

```text
/products

/api/products

/v1/products

```

----------

# Wildcards

Playwright uses glob patterns.

Example

```typescript
"**/*"

```

Matches

```text
Everything

```

----------

Example

```typescript
"**/*.png"

```

Matches

```text
logo.png

banner.png

image.png

```

----------

Example

```typescript
"**/api/**"

```

Matches

```text
/api/users

/api/products

/api/orders

```

----------

# Regular Expressions

Instead of wildcards:

```typescript
await page.route(

/.*\/products/,

async route => {

    await route.continue();

}

);

```

Useful for advanced matching.

----------

# Exact URL

```typescript
await page.route(

"https://example.com/api/login",

async route => {

    await route.continue();

}

);

```

Only one endpoint matches.

----------

# Route Lifecycle

Every request follows the same lifecycle.

```text
Request Created

↓

Route Match

↓

Route Handler

↓

Decision

↓

Continue / Fulfill / Abort

↓

Browser Receives Result

```

----------

# Continue Request

Suppose:

```text
Browser

↓

Playwright

↓

Server

```

You simply allow the request.

```typescript
await page.route(

"**/products",

async route => {

    await route.continue();

}

);

```

The server still processes it.

----------

# Abort Request

Instead:

```text
Browser

↓

Playwright

↓

STOP

```

```typescript
await page.route(

"**/*.png",

async route => {

    await route.abort();

}

);

```

Images never load.

Useful for performance testing.

----------

# Fulfill Request

Instead of contacting the server:

```text
Browser

↓

Playwright

↓

Fake Response

```

```typescript
await page.route(

"**/products",

async route => {

    await route.fulfill({

        status:200,

        body:"Hello"

    });

}

);

```

The server is never contacted.

We'll explore `fulfill()` in depth in the next chapter.

----------

# Route Precedence

Multiple routes can exist.

Example

```typescript
await page.route("**/*", ...);

await page.route("**/products", ...);

```

Which one runs?

Playwright evaluates route handlers in **last-in, first-out (LIFO)** order. The **most recently registered matching route** gets the first opportunity to handle the request.

```text
Route 1 Registered

↓

Route 2 Registered

↓

Request

↓

Route 2 Runs First

```

This behavior is important when composing framework-level and test-level routes.

----------

# Multiple Routes

Example

```typescript
await page.route(

"**/*.png",

async route => {

    await route.abort();

});

await page.route(

"**/products",

async route => {

    await route.continue();

});

```

Now:

```text
Images

↓

Aborted

---------

Products

↓

Continued

```

Different routes.

Different behavior.

----------

# Removing Routes

Sometimes mocking is temporary.

```typescript
await page.unroute("**/products");

```

The interception is removed.

You can also remove a specific handler by passing the same callback used during registration.

----------

# Real-World Example – Block Analytics

Many enterprise applications call:

```text
Google Analytics

Adobe Analytics

Mixpanel

```

These are irrelevant for UI tests.

```typescript
await page.route(

"**/analytics/**",

async route => {

    await route.abort();

});

```

Tests become faster.

----------

# Real-World Example – Ignore Images

```typescript
await page.route(

"**/*.{png,jpg,jpeg,gif}",

async route => {

    await route.abort();

});

```

UI still works.

Browser loads fewer resources.

----------

# Real-World Example – Monitor Requests

```typescript
await page.route(

"**/products",

async route => {

    console.log(

        route.request().url()

    );

    await route.continue();

});

```

Useful for debugging.

----------

# Accessing the Request

Every route provides a `Request` object.

```typescript
await page.route(

"**/login",

async route => {

    const request =
        route.request();

});

```

From here you can inspect:

-   URL
    
-   Headers
    
-   Method
    
-   Body
    
-   Cookies
    

We'll modify these in the next chapters.

----------

# Request Object APIs

API

Purpose

`url()`

Request URL

`method()`

HTTP Method

`headers()`

Request Headers

`postData()`

POST Body

`resourceType()`

Resource Type

----------

# Resource Types

You can filter by resource type.

Examples

```text
document

stylesheet

script

image

xhr

fetch

font

media

```

Example

```typescript
await page.route("**/*", async route => {

    const request = route.request();

    if (request.resourceType() === "image") {
        await route.abort();
        return;
    }

    await route.continue();

});

```

----------

# Enterprise Architecture

Instead of

```typescript
Every Test

↓

page.route()

```

Large frameworks use

```text
Framework Startup

↓

Route Manager

↓

Register Routes

↓

Tests

```

Everything is centralized.

----------

# Best Practices

-   Register routes before triggering the request you want to intercept.
    
-   Use `browserContext.route()` for application-wide or multi-tab behavior.
    
-   Keep route matching as specific as possible to avoid intercepting unrelated traffic.
    
-   Remove temporary routes when they are no longer needed.
    
-   Centralize commonly used routes in reusable utilities or framework components.
    

----------

# Common Mistakes

### ❌ Registering the route too late

```typescript
await page.click("#load");

await page.route("**/products", ...);

```

The request may already have been sent.

Always register the route first.

----------

### ❌ Using `"**/*"` for every mock

Overly broad patterns can intercept requests you never intended to handle.

----------

### ❌ Forgetting to continue or fulfill

Every intercepted request must eventually be:

-   Continued
    
-   Fulfilled
    
-   Aborted
    

Otherwise the request will remain pending and the page may hang.

----------

### ❌ Using `page.route()` for multi-tab applications

If new pages are created, consider `browserContext.route()` instead.

----------

# Interview Questions

### Q1. What is Route Interception?

Route Interception allows Playwright to intercept outgoing network requests before they reach the server, enabling them to be continued, modified, fulfilled, or aborted.

----------

### Q2. What is the difference between `page.route()` and `browserContext.route()`?

-   `page.route()` applies only to a single page.
    
-   `browserContext.route()` applies to every page created within that browser context.
    

----------

### Q3. What actions can a route handler perform?

A route handler can:

-   Continue the request
    
-   Fulfill the request with a mock response
    
-   Abort the request
    
-   Modify the request before continuing
    

----------

### Q4. Why should routes be registered before triggering requests?

Because network requests may be sent immediately. If the route is registered afterward, the request may already have reached the server.

----------

### Q5. What happens if a route handler neither continues, fulfills, nor aborts a request?

The intercepted request remains unresolved, which can cause the page to hang or the test to time out.

----------

# Summary

Route interception is the foundation of Playwright's network mocking capabilities. By intercepting requests before they reach the server, you gain complete control over network traffic—whether that means allowing requests through, blocking them, modifying them, or replacing them entirely with mock responses. Mastering route interception is essential before moving on to response mocking, request modification, HAR replay, and GraphQL mocking.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNDkyMjM3NDVdfQ==
-->
This is probably **the most important chapter** in the entire Mock API section because every API interception eventually ends with one of these three methods.

After reading this chapter, readers will understand **what actually happens to an intercepted request**.

----------

# Part 18 – Mock APIs

# Chapter 3 – Continue, Fulfill & Abort

----------

# Introduction

In the previous chapter, we learned how to intercept network requests using:

```typescript
await page.route("**/api/**", async route => {

});

```

However, once a request is intercepted, **Playwright pauses the request** and waits for your decision.

You must tell Playwright what to do next.

There are **three possible actions**.

```text
             Incoming Request
                    │
                    ▼
             Route Interception
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Continue        Fulfill        Abort
      │             │             │
      ▼             ▼             ▼
Real Server    Mock Response    Request Fails

```

These three methods form the foundation of all API mocking.

----------

# Overview

Method

Purpose

`route.continue()`

Send request to real server

`route.fulfill()`

Return mock response

`route.abort()`

Cancel request

Think of them like traffic signals.

```text
Continue  → Green Light

Fulfill   → Take a Detour

Abort      → Red Light

```

----------

# route.continue()

## What is it?

`continue()` tells Playwright:

> "I intercepted this request, but allow it to continue to the real server."

```text
Browser

↓

Playwright

↓

Real Server

↓

Real Response

```

----------

## Basic Example

```typescript
await page.route("**/products", async route => {
    await route.continue();
});

```

Nothing is mocked.

The request still reaches the server.

----------

# Why Use continue()?

Many engineers ask:

> "Why intercept a request only to continue it?"

Because you may want to:

-   Log requests
    
-   Modify requests
    
-   Add headers
    
-   Change authentication
    
-   Change POST body
    
-   Debug network traffic
    

----------

# Example – Logging Requests

```typescript
await page.route("**/products", async route => {

    console.log(route.request().url());

    await route.continue();

});

```

Output

```text
GET /api/products

```

----------

# Example – Logging Headers

```typescript
await page.route("**/products", async route => {

    console.log(
        route.request().headers()
    );

    await route.continue();

});

```

Useful for debugging.

----------

# Modifying Requests While Continuing

One of the biggest advantages of `continue()` is that you can modify the request before sending it.

Example

```typescript
await page.route("**/products", async route => {

    await route.continue({

        headers: {
            ...route.request().headers(),
            Authorization: "Bearer mock-token"
        }

    });

});

```

The server receives the modified request.

----------

# Changing the URL

```typescript
await page.route("**/products", async route => {

    await route.continue({

        url: "https://staging.example.com/products"

    });

});

```

Useful for:

-   Redirecting environments
    
-   Proxy testing
    
-   Migration testing
    

----------

# Changing HTTP Method

```typescript
await page.route("**/orders", async route => {

    await route.continue({

        method: "POST"

    });

});

```

Although technically possible, changing the HTTP method can affect server behavior and should be used with care.

----------

# route.fulfill()

## What is it?

Instead of sending the request to the server,

Playwright immediately returns a response.

```text
Browser

↓

Playwright

↓

Mock Response

↓

Browser

```

The server is never contacted.

----------

# Basic Example

```typescript
await page.route("**/products", async route => {

    await route.fulfill({

        status: 200,

        contentType: "application/json",

        body: JSON.stringify([
            {
                id: 1,
                name: "Mock Laptop"
            }
        ])

    });

});

```

The application believes the response came from the backend.

----------

# Response Properties

`fulfill()` accepts several options.

Property

Purpose

`status`

HTTP status code

`headers`

Response headers

`contentType`

MIME type

`body`

Response body

`json`

JSON response (preferred for JSON payloads)

`path`

Serve response from a file

----------

# Using `json`

Instead of:

```typescript
body: JSON.stringify({
    success: true
})

```

You can simply write:

```typescript
await route.fulfill({

    status: 200,

    json: {
        success: true
    }

});

```

Playwright serializes the object and sets the appropriate content type.

----------

# Mock Login API

```typescript
await page.route("**/login", async route => {

    await route.fulfill({

        status: 200,

        json: {

            token: "abc123",

            user: "Admin"

        }

    });

});

```

----------

# Returning HTML

```typescript
await route.fulfill({

    status: 200,

    contentType: "text/html",

    body: "<h1>Hello</h1>"

});

```

----------

# Returning CSS

```typescript
await route.fulfill({

    contentType: "text/css",

    body: "body { background: green; }"

});

```

----------

# Returning JavaScript

```typescript
await route.fulfill({

    contentType: "application/javascript",

    body: "console.log('Mock Script');"

});

```

----------

# route.abort()

## What is it?

Abort completely cancels the request.

```text
Browser

↓

Playwright

↓

Request Cancelled

```

----------

# Basic Example

```typescript
await page.route("**/*.png", async route => {

    await route.abort();

});

```

Images never load.

----------

# Why Abort Requests?

Common enterprise use cases:

-   Block analytics
    
-   Disable advertisements
    
-   Simulate offline mode
    
-   Test error handling
    
-   Improve execution speed
    

----------

# Example – Block Analytics

```typescript
await page.route("**/analytics/**", async route => {

    await route.abort();

});

```

----------

# Example – Simulate Network Failure

```typescript
await page.route("**/products", async route => {

    await route.abort("failed");

});

```

Now the browser receives a network failure instead of a response.

----------

# Abort Error Codes

Playwright supports several abort reasons.

Examples include:

Code

Meaning

`"failed"`

Generic failure

`"aborted"`

Request cancelled

`"timedout"`

Timeout

`"connectionrefused"`

Connection refused

`"internetdisconnected"`

Offline simulation

These are useful for testing application error handling.

----------

# Continue vs Fulfill vs Abort

Continue

Fulfill

Abort

Real server

Mock response

Cancel request

Backend executes

Backend skipped

Backend skipped

Good for request modification

Good for API mocking

Good for failure testing

----------

# Decision Flow

```text
Intercept Request

        │

        ▼

Need Real Backend?

     Yes ─────────► Continue

      │

      No

      │

Need Mock Response?

      │

     Yes ─────────► Fulfill

      │

      No

      ▼

Abort

```

----------

# Real-World Example – Login

Development environment:

```text
Login

↓

Mock JWT

↓

Dashboard

```

Use

```typescript
route.fulfill(...)

```

----------

Production smoke test:

```text
Login

↓

Real Authentication Server

```

Use

```typescript
route.continue()

```

----------

Network resilience test:

```text
Login

↓

Network Failure

```

Use

```typescript
route.abort()

```

----------

# Combining Logic

A single route handler can make decisions dynamically.

```typescript
await page.route("**/products", async route => {

    const request = route.request();

    if (request.method() === "GET") {

        await route.fulfill({

            json: [{ id: 1, name: "Laptop" }]

        });

    } else {

        await route.continue();

    }

});

```

This mocks only GET requests while allowing other methods through.

----------

# Best Practices

-   Use `continue()` when you need the real backend but want to inspect or modify requests.
    
-   Use `fulfill()` for deterministic test data and frontend isolation.
    
-   Use `abort()` to simulate network failures or block unnecessary resources.
    
-   Keep mocked responses close to real API contracts.
    
-   Register route handlers before the application triggers the request.
    

----------

# Common Mistakes

### ❌ Calling multiple route actions

```typescript
await route.continue();

await route.fulfill(...);

```

A request can only be resolved once.

----------

### ❌ Forgetting to resolve the route

```typescript
await page.route("**/api/**", async route => {

    console.log("Intercepted");

});

```

The request remains pending because neither `continue()`, `fulfill()`, nor `abort()` was called.

----------

### ❌ Mocking every endpoint

Only mock the APIs necessary for the scenario. Keep some tests running against the real backend to validate integrations.

----------

### ❌ Returning unrealistic responses

Mock responses should reflect the structure, status codes, and behavior of the real service whenever practical.

----------

# Interview Questions

### Q1. What are the three actions available for an intercepted request?

-   `route.continue()`
    
-   `route.fulfill()`
    
-   `route.abort()`
    

----------

### Q2. When should you use `route.continue()`?

When you want the request to reach the real server, possibly after inspecting or modifying it.

----------

### Q3. When should you use `route.fulfill()`?

When you want to replace the server response with a mock response for deterministic testing.

----------

### Q4. When should you use `route.abort()`?

When you want to simulate network failures or block requests such as analytics, advertisements, or images.

----------

### Q5. Can a route handler call both `continue()` and `fulfill()`?

No. Each intercepted request must be resolved exactly once.

----------

# Summary

Every intercepted request in Playwright must eventually be continued, fulfilled, or aborted. These three actions define whether the request reaches the real server, receives a mocked response, or fails entirely. Choosing the appropriate action allows you to build fast, reliable, and highly targeted tests while still preserving meaningful end-to-end coverage where it matters.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgwNzE3ODYzOF19
-->
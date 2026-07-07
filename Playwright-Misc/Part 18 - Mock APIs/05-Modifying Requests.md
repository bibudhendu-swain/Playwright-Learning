This chapter is one of the most useful in real enterprise projects.

Unlike response mocking, where we change **what comes back from the server**, this chapter focuses on changing **what gets sent to the server**.

This is extremely useful for:

-   Authentication
    
-   Environment switching
    
-   Security testing
    
-   Negative testing
    
-   API debugging
    
-   Proxy testing
    

In large organizations, request modification is often used in middleware, gateways, and API virtualization.

----------

# Part 18 – Mock APIs

# Chapter 5 – Modifying Requests

----------

# Introduction

In the previous chapter, we learned how to mock responses using:

```typescript
await route.fulfill({...});

```

Now we'll learn how to modify the **request itself** before it reaches the backend.

Normally the flow looks like this:

```text
Browser

↓

Server

↓

Response

```

With Playwright:

```text
Browser

↓

Playwright

↓

Modify Request

↓

Server

↓

Response

```

The backend still receives the request—but it may be different from what the browser originally sent.

----------

# Why Modify Requests?

Many enterprise applications need to test scenarios where requests differ from normal browser behavior.

Examples:

-   Inject authentication tokens
    
-   Switch environments
    
-   Test invalid payloads
    
-   Modify headers
    
-   Change query parameters
    
-   Simulate different users
    
-   Rewrite URLs
    
-   Security testing
    

----------

# Request Lifecycle

```text
Browser

↓

Original Request

↓

Playwright Route

↓

Modify Request

↓

Server

↓

Response

```

Unlike `route.fulfill()`, the request still reaches the server.

----------

# route.continue()

Request modification happens using:

```typescript
await route.continue({
    ...
});

```

The object passed to `continue()` overrides parts of the original request.

----------

# What Can Be Modified?

Property

Can Modify

URL

✅

Headers

✅

Method

✅

POST Data

✅

> **Note:** Cookies are typically managed through the browser context (`context.addCookies()`) or by modifying the `Cookie` header. There isn't a dedicated `cookies` option on `route.continue()`.

----------

# Modifying Headers

Suppose the browser sends:

```http
GET /products

Accept: application/json

```

Playwright can inject additional headers.

```typescript
await page.route("**/products", async route => {

    await route.continue({

        headers: {

            ...route.request().headers(),

            Authorization:
                "Bearer mock-token"

        }

    });

});

```

The server receives:

```http
GET /products

Authorization: Bearer mock-token

```

----------

# Why Use Header Modification?

Common enterprise scenarios:

-   OAuth
    
-   JWT
    
-   API Gateway
    
-   Correlation IDs
    
-   Feature Flags
    
-   Tenant IDs
    

----------

# Example – Correlation ID

```typescript
await page.route("**/*", async route => {

    await route.continue({

        headers: {

            ...route.request().headers(),

            "X-Correlation-Id":

                crypto.randomUUID()

        }

    });

});

```

Useful for tracing requests across distributed systems.

----------

# Example – Feature Flag

```typescript
await page.route("**/products", async route => {

    await route.continue({

        headers: {

            ...route.request().headers(),

            "X-Feature":

                "NewCheckout"

        }

    });

});

```

The backend enables a feature without changing application code.

----------

# Modifying URL

Suppose the application calls:

```text
https://prod.company.com

```

Playwright can redirect it.

```typescript
await page.route("**/products", async route => {

    await route.continue({

        url:

"https://staging.company.com/products"

    });

});

```

Useful for:

-   Environment switching
    
-   Migration
    
-   Proxy servers
    
-   Canary deployments
    

----------

# URL Rewrite Flow

```text
Browser

↓

Production URL

↓

Playwright

↓

Staging URL

↓

Server

```

The application is unaware of the change.

----------

# Modifying HTTP Method

Original

```http
GET /orders

```

Playwright

```typescript
await page.route("**/orders", async route => {

    await route.continue({

        method: "POST"

    });

});

```

Server receives

```http
POST /orders

```

> **Use with caution:** Changing the HTTP method can fundamentally change how the backend processes the request.

----------

# Reading POST Data

Suppose the application sends:

```json
{
    "username":"john",
    "password":"secret"
}

```

Read it.

```typescript
await page.route("**/login", async route => {

    const data =
        route.request().postDataJSON();

    console.log(data);

    await route.continue();

});

```

----------

# Modifying POST Body

```typescript
await page.route("**/login", async route => {

    const body =
        route.request().postDataJSON();

    body.username = "admin";

    await route.continue({

        postData:

            JSON.stringify(body)

    });

});

```

Server receives:

```json
{
    "username":"admin",
    "password":"secret"
}

```

----------

# Injecting Test Data

Original

```json
{
    "country":"India"
}

```

Playwright

```typescript
await page.route("**/address", async route => {

    const body =
        route.request().postDataJSON();

    body.country = "USA";

    await route.continue({

        postData:

            JSON.stringify(body)

    });

});

```

Useful for:

-   Localization
    
-   Region testing
    
-   Tax calculation
    
-   Shipping
    

----------

# Modifying Query Parameters

Original

```text
/products?page=1

```

Example

```typescript
await page.route("**/products**", async route => {

    const url = new URL(route.request().url());

    url.searchParams.set("page", "5");

    await route.continue({

        url: url.toString()

    });

});

```

Server receives:

```text
/products?page=5

```

----------

# Multiple Modifications

Everything can be changed together.

```typescript
await page.route("**/orders", async route => {

    const body =
        route.request().postDataJSON();

    body.priority = "High";

    await route.continue({

        url:

"https://staging.company.com/orders",

        method: "POST",

        headers: {

            ...route.request().headers(),

            Authorization:

                "Bearer mock"

        },

        postData:

            JSON.stringify(body)

    });

});

```

----------

# Modifying Cookies

Cookies are usually managed using the browser context.

```typescript
await context.addCookies([
    {
        name: "sessionId",
        value: "mock-session",
        domain: "example.com",
        path: "/"
    }
]);

```

If you specifically want to intercept and change the cookie sent with a request, you can modify the `Cookie` header:

```typescript
await page.route("**/profile", async route => {

    await route.continue({

        headers: {

            ...route.request().headers(),

            Cookie: "sessionId=mock-session"

        }

    });

});

```

----------

# Enterprise Example – Authentication

Instead of logging in every test:

```text
Login

↓

JWT

↓

Dashboard

```

Inject

```http
Authorization:
Bearer MockToken

```

The backend treats the request as authenticated.

----------

# Enterprise Example – Multi-Tenant

Suppose one backend serves multiple customers.

```text
Tenant A

Tenant B

Tenant C

```

Inject

```http
X-Tenant-ID

```

Different customers.

Same application.

----------

# Enterprise Example – Security Testing

Modify payload.

Original

```json
{
    "amount":100
}

```

Modified

```json
{
    "amount":1000000
}

```

Verify server validation.

----------

# Enterprise Example – API Version Testing

Application calls

```text
/api/v1/products

```

Playwright

```text
/api/v2/products

```

without changing frontend code.

Useful for migration testing.

----------

# Request Modification Flow

```text
Original Request

↓

Read

↓

Modify

↓

Continue

↓

Server

↓

Response

```

----------

# Best Practices

-   Modify only the parts of the request required for the scenario.
    
-   Preserve existing headers using the spread operator (`...route.request().headers()`).
    
-   Validate that modified requests still conform to API contracts.
    
-   Use request modification for testing infrastructure, authentication, and edge cases—not to hide application defects.
    
-   Register request interceptors before the request is triggered.
    

----------

# Common Mistakes

### ❌ Replacing all headers

```typescript
headers:{

Authorization:"..."

}

```

This removes important headers such as `Accept`, `Content-Type`, or custom application headers.

Better:

```typescript
headers:{

...route.request().headers(),

Authorization:"..."

}

```

----------

### ❌ Sending invalid JSON

Always ensure modified request bodies are valid JSON before serializing.

----------

### ❌ Forgetting API validation

If you modify a request, make sure your test verifies the application's behavior rather than assuming the backend accepted it.

----------

### ❌ Using request modification for every test

Reserve request rewriting for scenarios that genuinely require it. Overusing interception can make tests harder to understand.

----------

# Continue vs Modify vs Mock

Goal

Method

Use real request

`route.continue()`

Modify request then send

`route.continue({...})`

Replace response

`route.fulfill()`

Cancel request

`route.abort()`

----------

# Interview Questions

### Q1. What is the difference between `route.continue()` and `route.fulfill()`?

-   `route.continue()` sends the request to the real server (optionally with modifications).
    
-   `route.fulfill()` bypasses the server and returns a mock response.
    

----------

### Q2. What parts of a request can be modified?

You can modify:

-   URL
    
-   Headers
    
-   HTTP method
    
-   POST data
    

----------

### Q3. Why should you preserve existing headers when modifying requests?

Replacing all headers may remove essential values such as authentication, content negotiation, or application-specific headers. Using the spread operator helps preserve them.

----------

### Q4. Can Playwright modify the request body?

Yes. You can read the body using methods like `postDataJSON()`, update it, and pass the new payload via `postData` in `route.continue()`.

----------

### Q5. When is request modification commonly used in enterprise projects?

Authentication token injection, environment switching, multi-tenant testing, feature flags, API version migration, and security or negative testing.

----------

# Summary

Request modification allows Playwright to intercept outgoing network requests, inspect them, and alter key properties before they reach the backend. By changing headers, URLs, methods, query parameters, or request bodies, you can simulate a wide range of real-world scenarios without modifying application code. Combined with response mocking, request modification provides complete control over network interactions during testing.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0ODI5Nzk0MDNdfQ==
-->
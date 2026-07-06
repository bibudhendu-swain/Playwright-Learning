This chapter is arguably **the most important chapter** in the API Testing section because **everything in Playwright API testing revolves around `APIRequestContext`**.

If someone understands this class thoroughly, they can build an enterprise-grade API framework.

----------

# Part 18 – API Testing

# Chapter 2 – APIRequestContext

----------

# Introduction

In the previous chapter, we learned that Playwright provides a built-in HTTP client.

The heart of that client is:

```typescript
APIRequestContext

```

Think of it as Playwright's equivalent of:

-   Rest Assured Client
    
-   Axios Instance
    
-   HttpClient
    
-   Postman Collection Runner
    

Everything starts here.

----------

# What is APIRequestContext?

`APIRequestContext` represents an isolated HTTP client capable of sending requests to backend services.

It manages:

-   Base URL
    
-   Authentication
    
-   Headers
    
-   Cookies
    
-   SSL
    
-   Timeouts
    
-   Proxy configuration
    
-   Request lifecycle
    

----------

# Architecture

```text
Test

↓

APIRequestContext

↓

HTTP Request

↓

Server

↓

HTTP Response

```

Unlike UI automation,

no browser is required.

----------

# Creating an APIRequestContext

The simplest approach:

```typescript
import { request } from "@playwright/test";

const api = await request.newContext();

```

Now

```text
api

↓

GET

POST

PUT

DELETE

PATCH

```

----------

# Request Context Lifecycle

```text
Create Context

↓

Configure Context

↓

Send Requests

↓

Read Responses

↓

Dispose Context

```

----------

# Why Use a Context?

Imagine every request required:

```text
Authorization

Headers

Cookies

Base URL

Timeout

```

Without a context:

```typescript
request.get(...)

request.post(...)

request.put(...)

```

Every request repeats the same configuration.

With a context,

everything is configured once.

----------

# Basic Example

```typescript
import { request } from "@playwright/test";

const api = await request.newContext();

const response = await api.get(

    "https://example.com/api/products"

);

expect(response.ok()).toBeTruthy();

await api.dispose();

```

Notice

```typescript
dispose()

```

releases resources associated with the context.

----------

# Using the Built-in Test Fixture

When using Playwright Test,

a preconfigured `request` fixture is available.

```typescript
import { test } from "@playwright/test";

test("Get Products", async ({ request }) => {

    const response =
    await request.get("/products");

});

```

In this case,

Playwright creates and manages the request context automatically.

----------

# Configuring Base URL

Instead of

```typescript
"https://api.company.com/products"

```

configure

```typescript
const api =
await request.newContext({

    baseURL:

        "https://api.company.com"

});

```

Now

```typescript
await api.get("/products");

await api.get("/orders");

await api.get("/customers");

```

Much cleaner.

----------

# Base URL Flow

```text
Base URL

↓

https://api.company.com

↓

/products

↓

https://api.company.com/products

```

----------

# Default Headers

Many APIs require common headers.

Instead of repeating

```http
Authorization

Content-Type

Accept

```

configure once.

```typescript
const api =
await request.newContext({

    extraHTTPHeaders: {

        Authorization:

            "Bearer token",

        Accept:

            "application/json"

    }

});

```

Every request automatically includes them.

----------

# Authentication

Example

```typescript
const api =
await request.newContext({

    extraHTTPHeaders:{

        Authorization:

        `Bearer ${token}`

    }

});

```

Now

```typescript
await api.get("/orders");

await api.get("/users");

await api.get("/payments");

```

All requests are authenticated.

----------

# Cookies

APIRequestContext maintains its own cookie storage.

Suppose:

```text
Login

↓

Server

↓

Session Cookie

```

Subsequent requests automatically reuse that cookie.

Example:

```typescript
await api.post("/login", {
    data: {
        username: "admin",
        password: "password"
    }
});

const response =
await api.get("/profile");

```

The session cookie is sent automatically.

----------

# Ignoring SSL Errors

Some test environments use self-signed certificates.

Example

```typescript
const api =
await request.newContext({

    ignoreHTTPSErrors: true

});

```

Useful for:

-   QA
    
-   Dev
    
-   Internal environments
    

Avoid using this in production validation unless required.

----------

# Request Timeout

Default timeouts may not suit every API.

Configure

```typescript
const api =
await request.newContext({

    timeout:30000

});

```

Meaning

```text
30 Seconds

↓

Fail Request

```

if no response is received.

----------

# Proxy Configuration

Some organizations route traffic through proxies.

Example

```typescript
const api =
await request.newContext({

    proxy: {

        server:

            "http://proxy.company.com:8080"

    }

});

```

Useful for:

-   Corporate networks
    
-   Security testing
    
-   Traffic inspection
    

----------

# HTTP Credentials

Some servers use Basic Authentication.

```typescript
const api =
await request.newContext({

    httpCredentials: {

        username: "admin",

        password: "secret"

    }

});

```

The Authorization header is generated automatically.

----------

# Combining Configuration

```typescript
const api =
await request.newContext({

    baseURL:

        "https://api.company.com",

    extraHTTPHeaders:{

        Authorization:

            "Bearer token"

    },

    timeout:30000,

    ignoreHTTPSErrors:true

});

```

Everything is configured once.

----------

# Sending Multiple Requests

```typescript
const users =
await api.get("/users");

const orders =
await api.get("/orders");

const products =
await api.get("/products");

```

All requests share the same context.

----------

# Disposing Context

When finished,

dispose of the context.

```typescript
await api.dispose();

```

This closes connections and frees resources.

If you're using Playwright Test's built-in `request` fixture, you generally **do not** need to call `dispose()` yourself.

----------

# Context Reuse

Large frameworks often create one context per test or per worker, depending on isolation requirements.

```text
Worker

↓

API Context

↓

Many Requests

↓

Dispose

```

Avoid sharing mutable authenticated contexts across unrelated tests unless intentionally designed.

----------

# Enterprise API Client

Instead of

```typescript
await api.get("/users");

await api.post("/orders");

```

Use dedicated client classes.

```typescript
class UserApi {

    constructor(
        private api: APIRequestContext
    ) {}

    async getUsers(){

        return this.api.get("/users");

    }

}

```

Usage

```typescript
const userApi =
new UserApi(api);

await userApi.getUsers();

```

----------

# Suggested Folder Structure

```text
api/

├── ApiClient.ts

├── UserApi.ts

├── ProductApi.ts

├── OrderApi.ts

├── AuthApi.ts

└── BaseApi.ts

```

Each class manages one business domain.

----------

# Request Context vs Browser Context

BrowserContext

APIRequestContext

Controls browser

Controls HTTP client

Creates pages

Sends requests

UI automation

API automation

Stores browser cookies

Stores HTTP cookies

----------

# Sharing Authentication with UI

One powerful Playwright feature is that API authentication can be reused by browser tests.

Example workflow:

```text
API Login

↓

Authenticated State

↓

Open Browser

↓

Already Logged In

```

This is commonly achieved by saving and reusing storage state, which we'll cover in a later chapter.

----------

# Enterprise Architecture

```text
Tests

↓

API Fixtures

↓

API Client

↓

APIRequestContext

↓

Backend

```

Tests never call raw endpoints directly.

----------

# Best Practices

-   Configure `baseURL` instead of hardcoding full URLs.
    
-   Centralize common headers and authentication.
    
-   Create reusable API client classes.
    
-   Dispose manually created contexts when they are no longer needed.
    
-   Keep one responsibility per API client (Users, Orders, Products, etc.).
    

----------

# Common Mistakes

### ❌ Hardcoding URLs

Bad

```typescript
"https://company.com/api/users"

```

Better

```typescript
baseURL

```

----------

### ❌ Repeating Headers

Avoid

```typescript
Authorization

Accept

Content-Type

```

on every request.

Configure them once.

----------

### ❌ Forgetting dispose()

If you create contexts manually,

remember

```typescript
await api.dispose();

```

----------

### ❌ Mixing UI and API Logic

Avoid

```typescript
page.click(...);

api.get(...);

page.click(...);

api.post(...);

```

Keep API operations inside dedicated service classes or fixtures.

----------

# Interview Questions

### Q1. What is `APIRequestContext`?

It is Playwright's built-in HTTP client used for sending API requests without launching a browser.

----------

### Q2. Why should `baseURL` be configured?

It avoids repeating the server URL for every request and makes tests easier to maintain across environments.

----------

### Q3. Does `APIRequestContext` manage cookies?

Yes. It maintains its own cookie storage, allowing authenticated sessions to persist across requests within the same context.

----------

### Q4. When should `dispose()` be called?

When you create an `APIRequestContext` manually using `request.newContext()`. The built-in Playwright Test `request` fixture is managed automatically.

----------

### Q5. Why should enterprise frameworks wrap `APIRequestContext` inside API client classes?

To encapsulate endpoint logic, improve code reuse, simplify maintenance, and keep tests focused on business behavior instead of HTTP details.

----------

# Summary

`APIRequestContext` is the foundation of API testing in Playwright. It provides a configurable HTTP client that manages authentication, cookies, headers, base URLs, timeouts, and other request settings. By centralizing configuration and wrapping endpoint interactions inside reusable API client classes, teams can build scalable, maintainable, and enterprise-ready API automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2ODA1MDgxM119
-->
This chapter is where API mocking becomes practical.

So far we've learned:

-   How requests are intercepted
    
-   How to continue, fulfill, and abort requests
    

Now we'll learn **how to build realistic mock responses**. This is where most enterprise teams spend their time because a good mock should behave like a real backend—not just return `"Hello World"`.

----------

# Part 18 – Mock APIs

# Chapter 4 – Mocking Responses

----------

# Introduction

In the previous chapter, we learned that `route.fulfill()` allows Playwright to return a custom response instead of contacting the real server.

```text
Browser
      │
      ▼
Playwright
      │
      ▼
Mock Response
      │
      ▼
Browser

```

The application cannot tell whether the response came from the real server or Playwright.

This allows us to test almost every scenario imaginable.

----------

# What is Response Mocking?

Response mocking means replacing the server's response with one that you define.

Instead of:

```text
GET /products

↓

Server

↓

[
  {
    "id":1,
    "name":"Laptop"
  }
]

```

Playwright returns

```text
GET /products

↓

Playwright

↓

[
  {
    "id":999,
    "name":"Mock Laptop"
  }
]

```

----------

# Basic JSON Response

The simplest mock.

```typescript
await page.route("**/products", async route => {

    await route.fulfill({

        status: 200,

        json: [

            {
                id: 1,
                name: "Laptop"
            }

        ]

    });

});

```

Notice we used

```typescript
json:

```

instead of

```typescript
body: JSON.stringify(...)

```

Playwright automatically:

-   Converts to JSON
    
-   Sets `Content-Type: application/json`
    

----------

# Mocking an Empty Response

Sometimes you want:

```json
[]

```

Example

```typescript
await page.route("**/products", async route => {

    await route.fulfill({

        status: 200,

        json: []

    });

});

```

Useful for

-   Empty search results
    
-   No notifications
    
-   No orders
    

----------

# Mocking a Single Object

```typescript
await page.route("**/user/profile", async route => {

    await route.fulfill({

        status: 200,

        json: {

            id: 101,

            name: "John",

            role: "Administrator"

        }

    });

});

```

----------

# Mocking Nested JSON

Real APIs usually return nested objects.

```typescript
await page.route("**/orders", async route => {

    await route.fulfill({

        json: {

            id: 1001,

            customer: {

                id: 50,

                name: "Alice"

            },

            items: [

                {

                    id: 1,

                    name: "Laptop"

                }

            ]

        }

    });

});

```

----------

# Mocking Large Responses

Instead of writing huge JSON inside the test,

store it separately.

Example

```text
mocks/

    users.json

    products.json

    orders.json

```

Then

```typescript
import products from "../mocks/products.json";

await page.route("**/products", async route => {

    await route.fulfill({

        json: products

    });

});

```

This keeps tests clean and promotes reuse.

----------

# Mocking Different HTTP Status Codes

Not every response is `200 OK`.

----------

## 201 Created

```typescript
await route.fulfill({

    status: 201,

    json: {

        success: true

    }

});

```

----------

## 400 Bad Request

```typescript
await route.fulfill({

    status: 400,

    json: {

        message: "Invalid Request"

    }

});

```

----------

## 401 Unauthorized

```typescript
await route.fulfill({

    status: 401,

    json: {

        message: "Unauthorized"

    }

});

```

Test login expiration.

----------

## 403 Forbidden

```typescript
await route.fulfill({

    status: 403,

    json: {

        message: "Access Denied"

    }

});

```

Useful for role-based testing.

----------

## 404 Not Found

```typescript
await route.fulfill({

    status: 404,

    json: {

        message: "Product Not Found"

    }

});

```

----------

## 500 Internal Server Error

```typescript
await route.fulfill({

    status: 500,

    json: {

        message: "Server Error"

    }

});

```

Great for testing error banners.

----------

## 503 Service Unavailable

```typescript
await route.fulfill({

    status: 503,

    json: {

        message: "Service Unavailable"

    }

});

```

Useful for maintenance scenarios.

----------

# Returning HTML

Suppose your application loads

```text
/help

```

Instead of JSON

return HTML.

```typescript
await page.route("**/help", async route => {

    await route.fulfill({

        status: 200,

        contentType: "text/html",

        body: `

            <html>

                <body>

                    <h1>Mock Help Page</h1>

                </body>

            </html>

        `

    });

});

```

----------

# Returning CSS

```typescript
await page.route("**/*.css", async route => {

    await route.fulfill({

        contentType: "text/css",

        body: `
            body{
                background:lightblue;
            }
        `

    });

});

```

----------

# Returning JavaScript

```typescript
await page.route("**/config.js", async route => {

    await route.fulfill({

        contentType:
            "application/javascript",

        body:
            `window.mode="Mock";`

    });

});

```

----------

# Mocking Images

Example

```typescript
await page.route("**/logo.png", async route => {

    await route.fulfill({

        path: "./mocks/logo.png"

    });

});

```

Instead of downloading the real image,

Playwright serves the local file.

----------

# Mocking PDF

```typescript
await page.route("**/invoice.pdf", async route => {

    await route.fulfill({

        path: "./mocks/invoice.pdf"

    });

});

```

----------

# Mocking ZIP Files

```typescript
await page.route("**/download.zip", async route => {

    await route.fulfill({

        path: "./mocks/download.zip"

    });

});

```

----------

# Mocking Binary Files

```typescript
await page.route("**/sample.bin", async route => {

    await route.fulfill({

        path: "./mocks/sample.bin"

    });

});

```

----------

# Delayed Response

Sometimes the backend is slow.

Simulate it.

```typescript
await page.route("**/products", async route => {

    await new Promise(resolve =>
        setTimeout(resolve, 3000)
    );

    await route.fulfill({

        json: [

            {
                id: 1,
                name: "Laptop"
            }

        ]

    });

});

```

Useful for testing:

-   Loading spinners
    
-   Skeleton screens
    
-   Progress bars
    

> **Note:** This delays the route handler using real time. Later in the handbook, we'll combine network mocking with Playwright's Clock API where appropriate for deterministic time-based testing.

----------

# Dynamic Response

Instead of always returning the same response,

read the request.

```typescript
await page.route("**/products/*", async route => {

    const url =
        route.request().url();

    const id =
        url.split("/").pop();

    await route.fulfill({

        json: {

            id,

            name: `Product ${id}`

        }

    });

});

```

Now

```text
/products/10

```

returns

```json
{
    "id":"10",
    "name":"Product 10"
}

```

----------

# Conditional Response

```typescript
await page.route("**/login", async route => {

    const body =
        route.request().postDataJSON();

    if (body.username === "admin") {

        await route.fulfill({

            json: {

                success: true

            }

        });

    } else {

        await route.fulfill({

            status: 401,

            json: {

                success: false

            }

        });

    }

});

```

Great for testing both positive and negative login scenarios.

----------

# Enterprise Response Builder

Instead of

```typescript
await route.fulfill({

    json:{...}

});

```

Enterprise frameworks often use helper methods.

```typescript
await route.fulfill(

ResponseBuilder.success(

    mockProduct

)

);

```

or

```typescript
await route.fulfill(

ResponseBuilder.notFound()

);

```

This keeps response formatting consistent across hundreds of tests.

----------

# Response Mocking Flow

```text
Request

↓

Intercept

↓

Analyze Request

↓

Generate Response

↓

Fulfill

↓

Browser

```

----------

# Best Practices

-   Store reusable mock payloads in dedicated files.
    
-   Keep mock responses aligned with the real API contract.
    
-   Test both successful and failure responses.
    
-   Use realistic HTTP status codes.
    
-   Build reusable response factories for large frameworks.
    

----------

# Common Mistakes

### ❌ Hardcoding JSON in every test

Instead of:

```typescript
json:{
...
}

```

reuse shared mock files or builders.

----------

### ❌ Returning unrealistic responses

If production returns:

```json
{
    "data": {
        ...
    }
}

```

don't mock:

```json
{
    "id":1
}

```

unless you're intentionally testing malformed data.

----------

### ❌ Always returning 200

Real applications must also handle:

-   400
    
-   401
    
-   403
    
-   404
    
-   500
    
-   503
    

These deserve dedicated test cases.

----------

### ❌ Ignoring response headers

Some applications depend on headers such as:

-   `Content-Type`
    
-   `Cache-Control`
    
-   `Location`
    
-   `Set-Cookie`
    

Mock them when they are part of the behavior you're testing.

Example:

```typescript
await route.fulfill({
    status: 200,
    headers: {
        "Cache-Control": "no-cache"
    },
    json: { success: true }
});

```

----------

# Interview Questions

### Q1. What is `route.fulfill()`?

It intercepts a request and returns a custom response without contacting the real server.

----------

### Q2. Why should mock data be stored in separate files?

To improve readability, promote reuse, and keep tests aligned with real API payloads.

----------

### Q3. Can Playwright mock resources other than JSON?

Yes. It can mock HTML, CSS, JavaScript, images, PDFs, ZIP files, binary files, and other network resources.

----------

### Q4. Why should you test different HTTP status codes?

Applications often behave differently for success, validation errors, authorization failures, and server errors. Mocking these responses verifies that the UI handles each case correctly.

----------

### Q5. What is the advantage of dynamic response generation?

It allows mock responses to change based on the incoming request, making tests more realistic and reducing the need for many static mock files.

----------

# Summary

Response mocking allows Playwright to replace backend responses with realistic, deterministic data. Whether you're returning JSON, HTML, images, or error responses, `route.fulfill()` gives you complete control over what the browser receives. By combining reusable mock files, realistic status codes, and dynamic response generation, you can create stable UI tests that accurately simulate real-world backend behavior.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcyNDkxNDcxXX0=
-->
Now we move into what separates **basic API mocking** from **enterprise-grade API virtualization**.

Up until now, our mocks have been static:

```text
GET /products

↓

Always returns the same JSON

```

Real applications don't behave that way.

Sometimes:

-   The first request succeeds
    
-   The second request fails
    
-   The third request retries successfully
    
-   Responses take different amounts of time
    
-   Different users receive different responses
    
-   The backend maintains state
    

This chapter teaches how to simulate all of these scenarios.

----------

# Part 16 – Mock APIs

# Chapter 6 – Advanced API Mocking

----------

# Introduction

Simple mocks return the same response every time.

```text
Request 1

↓

200 OK

----------------

Request 2

↓

200 OK

----------------

Request 3

↓

200 OK

```

Real servers rarely behave like this.

Playwright allows us to build intelligent mocks that behave much more like production systems.

----------

# Advanced Mocking Scenarios

In enterprise projects, you often need to simulate:

-   Sequential responses
    
-   Conditional responses
    
-   Stateful APIs
    
-   Delayed responses
    
-   Retry scenarios
    
-   Network failures
    
-   Rate limiting
    
-   Partial mocking
    
-   Multiple endpoint coordination
    

----------

# Sequential Responses

Suppose an application polls an endpoint.

```
GET /status

```

Expected behavior:

```text
Request 1

↓

Processing

----------------

Request 2

↓

Processing

----------------

Request 3

↓

Completed

```

----------

## Implementation

```typescript
let requestCount = 0;

await page.route("**/status", async route => {

    requestCount++;

    if (requestCount < 3) {

        await route.fulfill({
            json: {
                status: "Processing"
            }
        });

    } else {

        await route.fulfill({
            json: {
                status: "Completed"
            }
        });

    }

});

```

----------

# Real World Example

Payment Processing

```
Payment

↓

Pending

↓

Authorizing

↓

Completed

```

Instead of one response,

simulate the actual lifecycle.

----------

# Conditional Mocking

Different requests receive different responses.

Example

```
GET /users/1

↓

John

------------

GET /users/2

↓

Alice

```

----------

## Example

```typescript
await page.route("**/users/*", async route => {

    const id =
        route.request()
            .url()
            .split("/")
            .pop();

    await route.fulfill({

        json: {

            id,

            name:
                id === "1"
                    ? "John"
                    : "Alice"

        }

    });

});

```

----------

# Mocking Based on Query Parameters

Application

```
GET /products?category=laptop

```

Example

```typescript
await page.route("**/products**", async route => {

    const url =
        new URL(route.request().url());

    const category =
        url.searchParams.get("category");

    if (category === "laptop") {

        await route.fulfill({

            json: [

                {

                    id: 1,

                    name: "Laptop"

                }

            ]

        });

        return;

    }

    await route.fulfill({

        json: []

    });

});

```

----------

# Mocking Based on Headers

Different users.

Different responses.

```typescript
await page.route("**/profile", async route => {

    const role =
        route.request()
            .headers()["x-role"];

    await route.fulfill({

        json: {

            admin:

                role === "admin"

        }

    });

});

```

Useful for

-   Role testing
    
-   Multi-tenant systems
    
-   Feature flags
    

----------

# Mocking Based on Request Body

Login example.

```typescript
await page.route("**/login", async route => {

    const body =
        route.request().postDataJSON();

    if (body.username === "admin") {

        await route.fulfill({

            status: 200,

            json: {

                success: true

            }

        });

        return;

    }

    await route.fulfill({

        status: 401,

        json: {

            success: false

        }

    });

});

```

----------

# Stateful Mocking

This is one of the most useful enterprise techniques.

Example:

Shopping Cart.

```
Cart

↓

Empty

↓

Add Item

↓

1 Item

↓

Add Item

↓

2 Items

```

The mock remembers previous requests.

----------

## Example

```typescript
const cart: any[] = [];

await page.route("**/cart", async route => {

    if (route.request().method() === "POST") {

        const item =
            route.request().postDataJSON();

        cart.push(item);

    }

    await route.fulfill({

        json: cart

    });

});

```

Now every request changes the mock's internal state.

----------

# Simulating Delays

Some APIs are slow.

```typescript
await page.route("**/products", async route => {

    await new Promise(resolve =>
        setTimeout(resolve, 5000)
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

Useful for

-   Loading indicators
    
-   Skeleton screens
    
-   Progress bars
    

----------

# Retry Simulation

Suppose the application retries after failures.

Expected behavior

```
Call 1

↓

500

--------------

Call 2

↓

500

--------------

Call 3

↓

200

```

----------

## Example

```typescript
let attempts = 0;

await page.route("**/orders", async route => {

    attempts++;

    if (attempts < 3) {

        await route.fulfill({

            status: 500,

            json: {

                message: "Temporary Failure"

            }

        });

        return;

    }

    await route.fulfill({

        status: 200,

        json: {

            success: true

        }

    });

});

```

Perfect for retry logic.

----------

# Simulating Network Failure

Instead of

```text
500

```

simulate

```
Connection Lost

```

```typescript
await page.route("**/products", async route => {

    await route.abort("internetdisconnected");

});

```

----------

# Simulating Timeout

```typescript
await page.route("**/products", async route => {

    await route.abort("timedout");

});

```

Useful for resilience testing.

----------

# Simulating Rate Limiting

Many APIs return

```
429 Too Many Requests

```

Example

```typescript
await page.route("**/search", async route => {

    await route.fulfill({

        status: 429,

        json: {

            message:

                "Too Many Requests"

        }

    });

});

```

Verify

-   Retry logic
    
-   User notification
    
-   Backoff strategy
    

----------

# Partial Mocking

One of the most common enterprise approaches.

Example

```
Login API

↓

Real Backend

----------------

Products API

↓

Mock

----------------

Inventory API

↓

Mock

----------------

Payments

↓

Real Backend

```

Implementation

```typescript
await page.route("**/products", async route => {

    await route.fulfill({

        json: mockProducts

    });

});

await page.route("**/inventory", async route => {

    await route.fulfill({

        json: mockInventory

    });

});

```

Everything else uses the real backend.

----------

# Mocking Multiple APIs Together

Order Page

```
Orders

↓

Customers

↓

Products

↓

Inventory

```

Each endpoint can be mocked independently.

```typescript
await page.route("**/orders", ...);

await page.route("**/customers", ...);

await page.route("**/products", ...);

await page.route("**/inventory", ...);

```

----------

# Enterprise Mock Architecture

Instead of

```
Test

↓

Huge Route Logic

```

Large frameworks use

```
Test

↓

Mock Manager

↓

Response Builder

↓

Mock Data

↓

Route Handler

```

Example

```typescript
MockManager.mockProducts(page);

MockManager.mockInventory(page);

MockManager.mockOrders(page);

```

Each mock is reusable across the entire framework.

----------

# Using Mock Builders

Instead of:

```typescript
await route.fulfill({

    json: {

        id: 1,

        name: "Laptop"

    }

});

```

Use:

```typescript
await route.fulfill({

    json: ProductBuilder
        .withName("Laptop")
        .withPrice(1500)
        .build()

});

```

This keeps test data expressive and maintainable.

----------

# Combining with Fixtures

Instead of mocking inside every test:

```typescript
test.beforeEach(async ({ page }) => {

    await MockManager.setup(page);

});

```

Your tests stay focused on behavior rather than setup.

----------

# Best Practices

-   Build reusable mock managers instead of duplicating route logic.
    
-   Keep mock data close to real production contracts.
    
-   Use stateful mocks only when the scenario genuinely requires it.
    
-   Reset mock state between tests to avoid cross-test contamination.
    
-   Prefer partial mocking over mocking every service.
    

----------

# Common Mistakes

### ❌ Using global variables without resetting

```typescript
let count = 0;

```

If not reset between tests, later tests may fail unpredictably.

----------

### ❌ Mocking business logic

Mocks should simulate backend behavior—not replace backend validation or business rules entirely.

----------

### ❌ Returning unrealistic data

Keep field names, types, and structures consistent with the real API.

----------

### ❌ Mocking everything

Maintain a healthy balance between mocked UI tests and real integration/end-to-end tests.

----------

# Enterprise Pattern

A scalable folder structure might look like:

```text
mocks/

├── data/
│   ├── users.json
│   ├── products.json
│   └── orders.json
│
├── builders/
│   ├── ProductBuilder.ts
│   ├── UserBuilder.ts
│   └── OrderBuilder.ts
│
├── handlers/
│   ├── ProductRoutes.ts
│   ├── UserRoutes.ts
│   └── OrderRoutes.ts
│
└── MockManager.ts

```

This separation makes mocks reusable, testable, and easy to maintain.

----------

# Interview Questions

### Q1. What is sequential mocking?

Sequential mocking returns different responses for successive requests to the same endpoint, allowing you to simulate state changes such as polling or retries.

----------

### Q2. What is stateful mocking?

Stateful mocking maintains internal state across requests, enabling scenarios like shopping carts, order workflows, or session-based interactions.

----------

### Q3. What is partial mocking?

Partial mocking intercepts only selected endpoints while allowing all other requests to reach the real backend.

----------

### Q4. Why simulate HTTP 429 or network failures?

To verify that the application correctly handles rate limiting, retries, error messages, and recovery behavior.

----------

### Q5. Why should enterprise frameworks centralize mock logic?

Centralization improves reuse, consistency, maintainability, and keeps test cases focused on business scenarios rather than network setup.

----------

# Summary

Advanced API mocking moves beyond static responses to simulate realistic backend behavior. By combining sequential responses, conditional logic, stateful data, delays, retries, and partial mocking, you can build highly reliable UI tests that closely resemble production environments. Organizing mocks into reusable managers and builders further improves maintainability and scalability in enterprise automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4MTA2MTYwMV19
-->
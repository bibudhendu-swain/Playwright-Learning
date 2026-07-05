This is one of Playwright features that can eliminate hundreds of lines of mocking code.

Many engineers spend days writing mock responses like:

```typescript
await page.route("**/products", async route => {
    await route.fulfill({
        json: products
    });
});

```

When in many cases, Playwright can simply **record real network traffic once and replay it forever**.

This is called **HAR Replay**.

----------

# Part 16 – Mock APIs

# Chapter 7 – HAR Replay

----------

# Introduction

Imagine your application makes **100 API calls** while loading a page.

Without HAR Replay:

```text
Browser

↓

100 Requests

↓

Real Backend

↓

100 Responses

```

or you manually mock all 100 APIs.

With HAR Replay:

```text
Browser

↓

Playwright

↓

HAR File

↓

Recorded Responses

```

No backend.

No internet.

No custom mocks.

Everything comes from a recorded HAR file.

----------

# What is a HAR File?

HAR stands for

> **HTTP Archive**

It is a standardized JSON format that records every network request and response during a browsing session.

A HAR file contains information such as:

-   Request URL
    
-   HTTP Method
    
-   Headers
    
-   Cookies
    
-   Query Parameters
    
-   POST Body
    
-   Response Headers
    
-   Response Body
    
-   Status Code
    
-   Timing Information
    

Think of it as a movie of your browser's network traffic.

----------

# HAR Structure

```text
HAR File

│

├── Request 1

├── Response 1

│

├── Request 2

├── Response 2

│

├── Request 3

├── Response 3

│

└── ...

```

----------

# Why Use HAR Replay?

Instead of writing:

```typescript
Mock Login

Mock Products

Mock Orders

Mock Customers

Mock Inventory

Mock Payment

```

You simply record once.

```text
Record

↓

HAR File

↓

Replay

```

Everything is mocked automatically.

----------

# Recording a HAR File

Playwright records HAR files at the browser context level.

Example

```typescript
const context = await browser.newContext({

    recordHar: {

        path: "./har/shop.har"

    }

});

```

Every network request made by this context is saved.

----------

# Recording Flow

```text
Browser

↓

Real Server

↓

Responses

↓

HAR File Saved

```

The application behaves normally while Playwright records the traffic.

----------

# Example

```typescript
const context = await browser.newContext({

    recordHar: {

        path: "./har/products.har"

    }

});

const page = await context.newPage();

await page.goto("https://example.com");

await context.close();

```

When the context closes, the HAR file is written to disk.

----------

# Replaying a HAR File

Now we no longer need the backend.

```typescript
const context = await browser.newContext();

await context.routeFromHAR(
    "./har/products.har"
);

const page =
await context.newPage();

```

Every matching request is served from the HAR file.

----------

# Replay Flow

```text
Browser

↓

Playwright

↓

HAR

↓

Recorded Response

```

The backend is never contacted.

----------

# What Happens During Replay?

Suppose the browser requests

```text
GET /products

```

Instead of

```text
GET

↓

Backend

↓

JSON

```

Playwright looks inside the HAR.

```text
GET

↓

HAR

↓

JSON

```

The application cannot tell the difference.

----------

# Matching Requests

Playwright compares the incoming request with entries in the HAR file.

It considers information such as:

-   URL
    
-   HTTP method
    
-   Query parameters
    
-   Request body (where applicable)
    

If a matching entry is found, the recorded response is returned.

----------

# Example

Recorded

```text
GET /products?id=10

```

Replay

```text
GET /products?id=10

```

Perfect match.

Response served from HAR.

----------

# Missing Requests

Suppose the HAR does **not** contain:

```text
GET /categories

```

What happens?

By default, the request fails because no matching entry exists.

Alternatively, you can configure Playwright to fall back to the network for unmatched requests.

----------

# Fallback to Network

```typescript
await context.routeFromHAR(
    "./har/shop.har",
    {
        notFound: "fallback"
    }
);

```

Now

```text
Found in HAR

↓

Replay

--------------

Not Found

↓

Real Backend

```

This is called **partial replay**.

----------

# Complete Offline Testing

Without fallback:

```text
Browser

↓

HAR

↓

Done

```

Internet is not required.

Very useful for:

-   CI
    
-   Air-gapped environments
    
-   Demos
    
-   Training
    

----------

# Updating HAR Files

Applications change.

Endpoints change.

Responses change.

Simply record again.

```text
New Backend

↓

Record HAR

↓

Replace Old HAR

```

No mocking code needs updating.

----------

# HAR Recording Modes

Playwright supports different update behaviors when replaying HARs.

A common option is:

```typescript
await context.routeFromHAR(
    "./shop.har",
    {
        update: true
    }
);

```

This allows Playwright to update the HAR with newly observed network traffic.

> **Note:** Depending on your Playwright version, additional update options may be available. Check the API reference for the exact behavior supported by your version.

----------

# Real-World Example

Shopping Website

Instead of mocking

```text
Products

Customers

Orders

Cart

Wishlist

Inventory

Coupons

```

Record everything.

One HAR.

Done.

----------

# Enterprise Example

Suppose a dashboard loads

```text
15 APIs

```

Writing mocks

```text
15 route.fulfill()

15 JSON Files

15 Builders

```

versus

```text
One HAR

```

Huge productivity improvement.

----------

# HAR vs Manual Mocking

HAR Replay

Manual Mock

Real recorded responses

Custom responses

Minimal setup

More code

Fast to create

More flexible

Great for stable APIs

Better for dynamic behavior

----------

# HAR Replay vs API Mocking

```text
HAR

↓

Replay Reality

-----------------

Mocking

↓

Create Reality

```

HAR replays what actually happened.

Manual mocks simulate what you want to happen.

----------

# When Should You Use HAR?

Good candidates

-   Stable APIs
    
-   Large applications
    
-   Offline testing
    
-   CI pipelines
    
-   Demo environments
    
-   Regression suites
    

----------

# When Should You Avoid HAR?

Avoid HAR replay when:

-   Responses must change dynamically
    
-   The application depends on timestamps
    
-   Tokens expire
    
-   Session IDs change
    
-   Business logic depends on generated values
    
-   Stateful workflows require different responses over time
    

Manual mocks are usually better for these scenarios.

----------

# Combining HAR with Manual Mocks

Enterprise teams often combine both.

Example

```text
Products

↓

HAR

--------------

Payments

↓

Manual Mock

--------------

Analytics

↓

Abort

```

Everything works together.

----------

# Folder Structure

```text
project/

├── har/
│   ├── login.har
│   ├── products.har
│   ├── checkout.har
│   └── dashboard.har
│
├── mocks/
│
└── tests/

```

Keep HAR files organized by feature or application module.

----------

# Enterprise Architecture

Instead of

```text
Tests

↓

HAR

```

Large frameworks use

```text
Tests

↓

Mock Manager

↓

HAR Manager

↓

HAR Files

```

Example

```typescript
await HARManager.load(
    context,
    "checkout.har"
);

```

Centralized.

Reusable.

Maintainable.

----------

# Performance Considerations

Real backend

```text
200 APIs

↓

Network

↓

30 Seconds

```

HAR

```text
200 APIs

↓

Local File

↓

3 Seconds

```

Especially useful in CI environments.

----------

# Best Practices

-   Record HAR files from stable environments with representative test data.
    
-   Organize HAR files by feature or business flow.
    
-   Regenerate HAR files when API contracts change.
    
-   Use fallback mode only when mixing recorded and live traffic intentionally.
    
-   Review HAR files before committing them to source control to ensure they don't contain secrets.
    

----------

# Common Mistakes

### ❌ Recording authentication tokens

HAR files may capture:

-   JWT tokens
    
-   Session cookies
    
-   Authorization headers
    

Sanitize or regenerate these before sharing.

----------

### ❌ Using one gigantic HAR

Instead of

```text
500 MB HAR

```

Create

```text
login.har

checkout.har

products.har

```

Smaller files are easier to maintain.

----------

### ❌ Using HAR for dynamic APIs

If responses change every request,

manual mocks are usually a better solution.

----------

### ❌ Forgetting to update HAR files

If backend APIs change but HAR files don't,

tests may no longer reflect reality.

----------

# Interview Questions

### Q1. What is a HAR file?

A HAR (HTTP Archive) file is a JSON-based recording of browser network traffic, including requests, responses, headers, cookies, and timing information.

----------

### Q2. What is HAR Replay?

HAR Replay allows Playwright to serve recorded network responses from a HAR file instead of contacting the real backend.

----------

### Q3. What is the advantage of HAR Replay over manual mocks?

HAR Replay requires very little code and uses real recorded responses, making it ideal for large applications with many API calls.

----------

### Q4. When should you avoid HAR Replay?

Avoid it for dynamic, stateful, or frequently changing APIs where handcrafted mocks provide greater flexibility.

----------

### Q5. Can HAR Replay be combined with manual mocks?

Yes. Enterprise frameworks often replay stable APIs from HAR files while manually mocking dynamic or business-critical endpoints.

----------

# Summary

HAR Replay is one of Playwright's most productive network testing features. Instead of manually mocking every endpoint, you can record real application traffic once and replay it across future test runs. This approach dramatically reduces mocking effort, improves test speed, and enables offline execution. For dynamic workflows, HAR replay can be combined with handcrafted mocks to achieve both realism and flexibility.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjUzMDc1NjczXX0=
-->
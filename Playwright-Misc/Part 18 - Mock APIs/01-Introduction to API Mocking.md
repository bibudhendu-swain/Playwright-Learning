# Part 18 – Mock APIs

## Chapter List

### Chapter 1 – Introduction to API Mocking ⭐⭐⭐⭐⭐

-   What is API Mocking?
    
-   Why Mock APIs?
    
-   API Mocking vs API Stubbing
    
-   Mocking vs Live APIs
    
-   How Playwright intercepts requests
    
-   Request Lifecycle
    
-   When to Mock
    
-   When NOT to Mock
    
-   Real-world scenarios
    
-   Advantages & Limitations
    

----------

### Chapter 2 – Route Interception

-   page.route()
    
-   browserContext.route()
    
-   Route lifecycle
    
-   Multiple routes
    
-   Route matching
    
-   Wildcards
    
-   Regex
    
-   Route precedence
    

----------

### Chapter 3 – Continue, Abort & Fulfill

-   route.continue()
    
-   route.abort()
    
-   route.fulfill()
    
-   Practical examples
    

----------

### Chapter 4 – Mocking Responses

-   Static JSON
    
-   Dynamic JSON
    
-   HTML
    
-   CSS
    
-   JavaScript
    
-   Images
    
-   Binary
    
-   PDF
    
-   ZIP
    

----------

### Chapter 5 – Modifying Requests

-   Headers
    
-   Query Parameters
    
-   POST Body
    
-   Cookies
    
-   Authorization
    
-   URL Rewrite
    

----------

### Chapter 6 – Advanced Mocking

-   Sequential responses
    
-   Delayed responses
    
-   Retry simulation
    
-   Network failure
    
-   HTTP errors
    
-   Partial mocking
    
-   Multiple APIs
    

----------

### Chapter 7 – HAR Replay

-   Record HAR
    
-   Replay HAR
    
-   Update HAR
    
-   Offline testing
    
-   Enterprise examples
    

----------

### Chapter 8 – GraphQL Mocking

-   Query
    
-   Mutation
    
-   Variables
    
-   Operation Name
    
-   Dynamic response
    

----------

### Chapter 9 – Enterprise Mock Architecture

-   Route Manager
    
-   Mock Builders
    
-   Response Factory
    
-   Test Data Factory
    
-   Environment Strategy
    
-   Best Practices
    

----------

# Let's start.

----------

# Part 16 – Mock APIs

# Chapter 1 – Introduction to API Mocking

----------

# Introduction

Modern web applications rarely work in isolation.

Almost every user action communicates with one or more backend APIs.

For example:

```text
User Clicks Login
        │
        ▼
POST /login
        │
        ▼
Authentication Server
        │
        ▼
JWT Token
        │
        ▼
Dashboard Opens

```

Similarly,

```text
Search Product
        │
        ▼
GET /products
        │
        ▼
Product Service
        │
        ▼
JSON Response
        │
        ▼
Products Displayed

```

Without APIs, most modern applications cannot function.

----------

# What is API Mocking?

API Mocking means intercepting an HTTP request and returning a predefined response instead of allowing it to reach the real backend server.

Instead of:

```text
Browser

↓

Real Server

↓

Real Response

```

Playwright changes the flow to:

```text
Browser

↓

Playwright Route

↓

Mock Response

↓

Browser

```

The backend server is never contacted.

----------

# Real Backend Flow

Normally

```text
User

↓

Browser

↓

Internet

↓

Backend

↓

Database

↓

Backend

↓

Browser

```

Many external systems are involved.

----------

# Mocked Flow

With Playwright

```text
User

↓

Browser

↓

Playwright

↓

Mock Response

↓

Browser

```

Everything happens locally.

----------

# Why Mock APIs?

There are many situations where relying on a live backend is undesirable.

Examples include:

-   Backend service is unavailable
    
-   Third-party API has rate limits
    
-   Test data is inconsistent
    
-   Expensive API calls
    
-   Slow network
    
-   Simulating error conditions
    
-   Testing edge cases
    
-   Developing the frontend before the backend is ready
    

----------

# Example

Suppose:

```text
GET /products

```

Normally returns

```json
[
  {
    "id": 1,
    "name": "Laptop"
  }
]

```

Playwright can instead return

```json
[
  {
    "id": 100,
    "name": "Mock Laptop"
  }
]

```

without changing the application code.

----------

# API Mocking vs API Stubbing

People often use these terms interchangeably.

There is a subtle difference.

## API Mocking

Replaces the actual server response.

```text
Request

↓

Playwright

↓

Fake Response

```

----------

## API Stubbing

Usually refers to providing predefined responses for a specific test scenario.

```text
Test

↓

Known Response

↓

Verification

```

In Playwright, both concepts are generally implemented using route interception.

----------

# How Playwright Mocks APIs

Internally,

Playwright sits between the browser and the network.

```text
Browser

↓

Route Interceptor

↓

Internet

```

Every request can be:

-   Continued
    
-   Modified
    
-   Fulfilled
    
-   Aborted
    

We'll cover each of these in later chapters.

----------

# Request Lifecycle

Without mocking:

```text
Request

↓

Server

↓

Response

↓

Browser

```

With mocking:

```text
Request

↓

Playwright Route

↓

Mock Response

↓

Browser

```

The server never receives the request.

----------

# What Can Be Mocked?

Almost any network resource.

Examples:

Resource

Can Mock

REST API

✅

GraphQL

✅

HTML

✅

CSS

✅

JavaScript

✅

Images

✅

Fonts

✅

PDF

✅

ZIP

✅

Binary Files

✅

----------

# Common Enterprise Use Cases

## 1. Backend Not Ready

Frontend team starts before backend.

```text
Frontend

↓

Mock APIs

↓

Development Continues

```

----------

## 2. Stable Test Data

Instead of relying on production data:

```text
Customer

↓

Always John Smith

```

Every test receives identical data.

----------

## 3. Simulating Errors

Real APIs rarely return:

```text
500

503

504

429

```

exactly when you need them.

Mocking allows these scenarios to be tested consistently.

----------

## 4. Third-Party APIs

Example:

```text
Google Maps

Stripe

PayPal

Salesforce

```

Mocking avoids:

-   Charges
    
-   Quotas
    
-   Rate limits
    
-   External outages
    

----------

## 5. Performance

Instead of

```text
API

↓

2 Seconds

```

Mock

```text
API

↓

5 Milliseconds

```

Tests become much faster.

----------

# Advantages

✅ Faster execution

✅ Stable tests

✅ Offline execution

✅ Deterministic responses

✅ Easy error simulation

✅ Reduced infrastructure dependency

----------

# Limitations

Mocking is not a replacement for integration testing.

A mocked API cannot verify:

-   Server-side validation
    
-   Database behavior
    
-   Business logic in the backend
    
-   Authentication implementation
    
-   Network infrastructure
    

These still require integration or end-to-end testing against real services.

----------

# When Should You Mock?

Good candidates include:

-   Third-party APIs
    
-   Unstable environments
    
-   Error scenarios
    
-   Slow services
    
-   Edge-case testing
    
-   Frontend development
    
-   Demonstrations
    

----------

# When Should You NOT Mock?

Avoid mocking when validating:

-   End-to-end integrations
    
-   Backend business logic
    
-   Authentication flows
    
-   Database updates
    
-   API contracts
    
-   Production deployment verification
    

A healthy automation strategy usually combines mocked tests with real end-to-end tests.

----------

# Mocking Strategy

A common enterprise testing pyramid looks like this:

```text
              Few
      End-to-End Tests
   (Real APIs & Backend)

        Integration Tests
      (Mostly Real Services)

          UI Tests
 (Mock External/Slow Services)

     Component / Unit Tests
        (Fully Mocked)

             Many

```

The higher you go, the fewer tests you typically have—but they exercise more of the real system.

----------

# Real-World Example

Suppose you're testing a shopping cart.

Instead of depending on inventory services:

```text
Add To Cart

↓

Inventory API

↓

Stock Available

```

Mock

```text
Add To Cart

↓

Mock API

↓

Always In Stock

```

Now the UI behavior is isolated and predictable.

----------

# Best Practices

-   Mock only the services necessary for the scenario being tested.
    
-   Keep mock responses realistic and representative of production.
    
-   Validate business outcomes, not just mocked payloads.
    
-   Store reusable mock responses separately from test logic.
    
-   Combine mocked UI tests with a smaller set of real end-to-end tests.
    

----------

# Common Mistakes

### ❌ Mocking everything

If every API is mocked, you lose confidence that the application works with the real backend.

----------

### ❌ Returning unrealistic responses

Mock data should reflect actual API contracts and business rules.

----------

### ❌ Hardcoding the same response in every test

Create reusable mock builders or response factories instead.

----------

### ❌ Never running tests against real services

A balanced test suite includes both mocked and non-mocked scenarios.

----------

# Interview Questions

### Q1. What is API Mocking?

API Mocking intercepts network requests and returns predefined responses instead of communicating with the real backend service.

----------

### Q2. Why is API Mocking useful?

It improves test speed, stability, reproducibility, and allows testing scenarios that are difficult to reproduce with live systems.

----------

### Q3. Does API Mocking replace end-to-end testing?

No. It complements end-to-end testing but cannot validate real backend behavior or system integrations.

----------

### Q4. What types of resources can Playwright mock?

REST APIs, GraphQL, HTML, CSS, JavaScript, images, fonts, PDFs, ZIP files, and other network resources.

----------

### Q5. What is the biggest mistake teams make with API Mocking?

Over-mocking. If every service is mocked, tests may pass while real integrations are broken.

----------

# Summary

API Mocking is one of Playwright's most powerful capabilities. By intercepting network requests and supplying controlled responses, you can build fast, deterministic, and highly reliable UI tests that are independent of backend availability. However, mocking should be used strategically—it complements, rather than replaces, integration and end-to-end testing against real services.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA4ODEwMTgyM119
-->
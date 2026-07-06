# Part 18 – API Testing

Unlike Selenium, Playwright is not just a UI automation tool. It has a **built-in API testing framework**, allowing you to test REST APIs, GraphQL APIs, authentication flows, and backend services without using external libraries like Rest Assured or SuperTest.

This section will start from the fundamentals and gradually move to enterprise-level API testing.

----------

# Part 18 – API Testing

# Chapter 1 – Introduction to API Testing in Playwright

----------

# Introduction

Traditionally, UI automation and API automation were performed using different tools.

For example:

Purpose

Tool

UI Automation

Selenium

API Automation

Rest Assured

Performance

JMeter

Mobile

Appium

Playwright changes this approach by allowing UI and API testing within the same framework.

This means one automation framework can perform:

-   UI Testing
    
-   API Testing
    
-   Authentication
    
-   Contract Validation
    
-   Backend Setup
    
-   Backend Cleanup
    

without switching tools.

----------

# What is API Testing?

API Testing verifies that an application's backend services work correctly without interacting with the user interface.

Instead of clicking buttons,

```text
User

↓

Browser

↓

UI

↓

Backend

```

we directly communicate with the backend.

```text
Test

↓

API Request

↓

Backend

↓

API Response

```

This makes API tests:

-   Faster
    
-   More reliable
    
-   Easier to debug
    

----------

# Why API Testing?

Imagine an application that creates a customer.

UI Flow

```text
Login

↓

Navigate

↓

Click New Customer

↓

Fill Form

↓

Submit

↓

Customer Created

```

API Flow

```text
POST /customers

↓

Customer Created

```

Instead of performing five UI actions,

we make one API request.

----------

# Why Playwright API Testing?

Playwright provides a built-in HTTP client.

No additional libraries are required.

Everything is included.

```text
Playwright

│

├── UI Testing

├── API Testing

├── Browser Automation

├── Authentication

└── Network Mocking

```

----------

# Playwright API Architecture

The core API object is:

```typescript
APIRequestContext

```

It behaves like an HTTP client.

```text
Test

↓

APIRequestContext

↓

HTTP Request

↓

Backend

↓

HTTP Response

```

----------

# Creating an API Request Context

The simplest approach is:

```typescript
import { request } from '@playwright/test';

const apiContext = await request.newContext();

```

This creates an isolated HTTP client.

----------

# Request Lifecycle

Every API request follows the same lifecycle.

```text
Create Request

↓

Send Request

↓

Server Processes

↓

Receive Response

↓

Validate Response

```

----------

# Supported HTTP Methods

Playwright supports all standard HTTP methods.

Method

Purpose

GET

Read data

POST

Create data

PUT

Replace data

PATCH

Update data

DELETE

Delete data

HEAD

Headers only

OPTIONS

Supported operations

----------

# Basic GET Request

```typescript
import { test, expect } from "@playwright/test";

test("Get Products", async ({ request }) => {

    const response = await request.get(

        "https://api.example.com/products"

    );

    expect(response.ok()).toBeTruthy();

});

```

----------

# Basic POST Request

```typescript
test("Create Product", async ({ request }) => {

    const response = await request.post(

        "https://api.example.com/products",

        {

            data: {

                name: "Laptop",

                price: 1500

            }

        }

    );

    expect(response.status())

        .toBe(201);

});

```

----------

# Reading JSON Response

```typescript
const response = await request.get(

    "/products"

);

const data = await response.json();

console.log(data);

```

----------

# Reading Text Response

```typescript
const text = await response.text();

```

Useful for:

-   HTML
    
-   Plain Text
    
-   XML
    

----------

# Reading Response Headers

```typescript
const headers = response.headers();

console.log(headers);

```

----------

# Reading Status Code

```typescript
expect(response.status())

.toBe(200);

```

----------

# Common Response APIs

Method

Description

`status()`

HTTP status code

`ok()`

Success status (2xx)

`json()`

Parse JSON response

`text()`

Read text response

`headers()`

Response headers

`body()`

Raw response bytes

----------

# API Testing vs UI Testing

API Testing

UI Testing

Fast

Slower

No browser rendering

Full browser rendering

Backend validation

End-to-end validation

Less flaky

Can be affected by UI changes

Business logic

User workflows

----------

# When Should You Use API Tests?

API tests are ideal for:

-   CRUD operations
    
-   Authentication
    
-   Data validation
    
-   Business rules
    
-   Backend integration
    
-   Test data setup
    
-   Test data cleanup
    

----------

# When Should You Use UI Tests?

UI tests are better for:

-   User journeys
    
-   Navigation
    
-   Visual behavior
    
-   Accessibility
    
-   Cross-browser validation
    
-   End-to-end workflows
    

----------

# Combining API and UI

One of Playwright's biggest advantages is combining both.

Example:

```text
API

↓

Create Customer

↓

Open Browser

↓

Verify Customer Appears

```

Instead of creating test data through the UI.

----------

# Real-World Example

Traditional UI test

```text
Login

↓

Create Order

↓

Search Order

↓

Verify Order

```

Optimized Playwright approach

```text
API

↓

Create Order

↓

UI

↓

Search Order

↓

Verify Order

```

The UI test becomes significantly faster because setup happens through the API.

----------

# Enterprise Architecture

A typical enterprise project separates API logic.

```text
project/

├── api/
│   ├── Client.ts
│   ├── CustomerApi.ts
│   ├── ProductApi.ts
│   └── OrderApi.ts
│
├── tests/
│
├── pages/
│
└── fixtures/

```

UI tests never call raw endpoints directly.

Instead:

```typescript
await customerApi.createCustomer(customer);

```

----------

# Advantages Over Rest Assured

Playwright

Rest Assured

Same framework as UI

Separate framework

Shared authentication

Separate authentication

Shared reporting

Separate reporting

Shared fixtures

Separate setup

TypeScript support

Java-centric

----------

# Best Practices

-   Use API tests for backend validation and UI tests for user experience.
    
-   Create reusable API client classes instead of calling endpoints directly from tests.
    
-   Use APIs to prepare and clean up test data.
    
-   Validate status codes, headers, and response bodies—not just one of them.
    
-   Keep API tests independent and deterministic.
    

----------

# Common Mistakes

### ❌ Using UI for test data setup

Instead of:

```text
Open Browser

↓

Create Customer

```

Prefer:

```text
API

↓

Create Customer

```

----------

### ❌ Hardcoding URLs

Instead of:

```typescript
"https://api.company.com"

```

Use configuration.

```typescript
baseURL

```

----------

### ❌ Ignoring Response Validation

Don't only check:

```typescript
expect(response.ok()).toBeTruthy();

```

Also validate:

-   Status code
    
-   Response body
    
-   Headers
    
-   Business rules
    

----------

### ❌ Mixing API Logic Inside UI Tests

Avoid:

```typescript
await request.post(...);

await page.goto(...);

await request.delete(...);

```

Move API operations into dedicated client classes.

----------

# Interview Questions

### Q1. What is `APIRequestContext` in Playwright?

`APIRequestContext` is Playwright's built-in HTTP client used to send API requests without launching a browser.

----------

### Q2. Why use Playwright for API testing instead of a separate library?

It enables API and UI testing within the same framework, sharing authentication, configuration, fixtures, and reporting.

----------

### Q3. Which HTTP methods does Playwright support?

GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS.

----------

### Q4. When should API testing be preferred over UI testing?

For backend validation, business logic verification, and test data setup/cleanup where the user interface is not the focus.

----------

### Q5. What is one major enterprise advantage of combining API and UI testing?

API calls can prepare test data quickly, allowing UI tests to focus only on validating user behavior, resulting in faster and more reliable test suites.

----------

# Summary

Playwright's built-in API testing capabilities eliminate the need for separate HTTP testing libraries in many projects. Using `APIRequestContext`, you can communicate directly with backend services, create and validate test data, and seamlessly combine API and UI testing within a single automation framework. This approach produces faster, cleaner, and more maintainable enterprise test suites.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NDE5MzIwNjVdfQ==
-->
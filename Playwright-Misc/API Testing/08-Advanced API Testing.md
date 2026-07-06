Real enterprise projects deal with scenarios like:

-   Chained APIs
    
-   Async job processing
    
-   Polling
    
-   Retry mechanisms
    
-   Rate limiting
    
-   Versioned APIs
    
-   File APIs
    
-   GraphQL
    
-   Long-running workflows
    

This chapter brings all those concepts together.

----------

# Part 18 – API Testing

# Chapter 8 – Advanced API Testing

----------

# Introduction

Modern enterprise APIs are rarely simple CRUD operations.

Instead, they often involve workflows such as:

```text
Create Order

↓

Returns Job ID

↓

Background Processing

↓

Poll Status

↓

Completed

↓

Download Report

```

A single business transaction may involve multiple API calls, asynchronous processing, retries, and validations.

----------

# Advanced API Testing Overview

Enterprise API testing commonly includes:

Topic

Example

Chained Requests

Login → Customer → Order

Polling

Wait until job finishes

Retry Logic

Retry transient failures

Async APIs

Background processing

Batch APIs

Bulk create/update

File APIs

Upload & Download

Rate Limiting

HTTP 429

Versioning

`/v1`, `/v2`

GraphQL

Queries & Mutations

Contract Validation

Schema verification

----------

# Chained API Requests

Many APIs depend on previous responses.

Example

```text
Login

↓

Access Token

↓

Create Customer

↓

Customer ID

↓

Create Order

↓

Order ID

↓

Get Invoice

```

----------

## Example

```typescript
const login = await request.post("/login", {
    data: credentials
});

const token =
(await login.json()).accessToken;

const customer =
await request.post("/customers", {
    headers: {
        Authorization: `Bearer ${token}`
    },
    data: customerData
});

const customerId =
(await customer.json()).id;

await request.post("/orders", {
    headers: {
        Authorization: `Bearer ${token}`
    },
    data: {
        customerId
    }
});

```

Each request depends on the previous one.

----------

# Long-Running APIs

Many APIs respond immediately but complete processing later.

Example

```text
POST /report

↓

202 Accepted

↓

Job Running

↓

Completed

```

----------

# Polling APIs

Instead of sleeping,

poll until completion.

```text
Start Job

↓

Check Status

↓

Still Running

↓

Check Again

↓

Completed

```

----------

## Example

```typescript
let status = "RUNNING";

while (status !== "COMPLETED") {

    const response =
    await request.get("/jobs/100");

    status =
    (await response.json()).status;

}

```

----------

# Better Polling

Always limit retries.

```typescript
const maxAttempts = 10;

for (let i = 0; i < maxAttempts; i++) {

    const response =
    await request.get("/jobs/100");

    const body =
    await response.json();

    if (body.status === "COMPLETED")
        break;

    await new Promise(r =>
        setTimeout(r, 1000)
    );
}

```

This prevents infinite loops.

----------

# Retry Logic

Sometimes APIs fail temporarily.

Example

```text
503

↓

Retry

↓

200

```

----------

## Example

```typescript
for (let i = 0; i < 3; i++) {

    const response =
    await request.get("/orders");

    if (response.ok())
        break;

}

```

Retry only transient failures such as:

-   429
    
-   502
    
-   503
    
-   504
    

Avoid retrying permanent failures like **400 Bad Request** unless the business flow specifically requires it.

----------

# Exponential Backoff

Instead of retrying immediately,

increase wait time.

```text
Retry 1

↓

1 Second

Retry 2

↓

2 Seconds

Retry 3

↓

4 Seconds

```

Example

```typescript
for (let i = 0; i < 3; i++) {

    const response =
    await request.get("/orders");

    if (response.ok())
        break;

    await new Promise(r =>
        setTimeout(r, 1000 * Math.pow(2, i)));

}

```

----------

# Batch APIs

Many enterprise APIs process multiple records together.

Example

```json
[
    {
        "id":1
    },
    {
        "id":2
    },
    {
        "id":3
    }
]

```

Playwright

```typescript
await request.post("/customers/batch", {

    data: customers

});

```

----------

# File Upload APIs

```typescript
await request.post("/upload", {

    multipart: {

        file: {

            name: "users.csv",

            mimeType: "text/csv",

            buffer:
            fs.readFileSync("./users.csv")

        }

    }

});

```

Useful for:

-   CSV import
    
-   Excel upload
    
-   Images
    
-   PDFs
    

----------

# File Download APIs

```typescript
const response =
await request.get("/report");

const pdf =
await response.body();

```

Validate

```typescript
expect(pdf.length)

.toBeGreaterThan(0);

```

----------

# API Rate Limiting

Many APIs protect themselves.

Example

```text
Too Many Requests

↓

429

```

Headers often include:

```http
Retry-After: 60

```

Test

```typescript
expect(response.status())

.toBe(429);

expect(

response.headers()["retry-after"]

)

.toBeDefined();

```

----------

# API Versioning

Large systems expose multiple versions.

```text
/v1/customers

/v2/customers

```

Test both versions independently.

```typescript
await request.get("/v1/users");

await request.get("/v2/users");

```

----------

# GraphQL Testing

GraphQL uses one endpoint.

```text
POST

↓

/graphql

↓

Operation

```

Example

```typescript
await request.post("/graphql", {

    data: {

        operationName:
        "GetUsers",

        query,

        variables

    }

});

```

Validate

```typescript
const body =
await response.json();

expect(body.data)

.toBeDefined();

```

----------

# Async Job Workflow

```text
Upload File

↓

Job Created

↓

Job ID

↓

Polling

↓

Completed

↓

Download Result

```

Very common in enterprise systems.

----------

# Parallel API Execution

Independent APIs can be executed concurrently.

```typescript
const [users, orders] =
await Promise.all([

    request.get("/users"),

    request.get("/orders")

]);

```

This reduces overall execution time.

Use parallel execution only when the requests do not depend on one another.

----------

# Response Caching

Some APIs return:

```http
Cache-Control

ETag

Last-Modified

```

Validate

```typescript
expect(

response.headers()["etag"]

)

.toBeDefined();

```

----------

# Idempotency Keys

Payment APIs often require idempotency.

```http
Idempotency-Key:
abc123

```

Example

```typescript
await request.post("/payments", {

    headers: {

        "Idempotency-Key":

        crypto.randomUUID()

    },

    data: payment

});

```

Repeated requests with the same key should not create duplicate transactions.

----------

# Contract Validation

Schema

↓

Response

↓

Compare

↓

Pass

Libraries such as AJV are commonly used.

```typescript
expect(validate(body))

.toBeTruthy();

```

----------

# Enterprise Workflow Example

```text
Login

↓

Customer

↓

Order

↓

Invoice

↓

Payment

↓

Shipment

↓

Tracking

```

Each step validates both:

-   HTTP response
    
-   Business state
    

----------

# Enterprise API Service

```typescript
class OrderWorkflow {

    async createOrder(){

    }

    async payOrder(){

    }

    async shipOrder(){

    }

}

```

Tests become

```typescript
await workflow.createOrder();

await workflow.payOrder();

await workflow.shipOrder();

```

Business logic is separated from the tests.

----------

# Suggested Folder Structure

```text
api/

├── BaseApi.ts

├── workflows/

│   ├── OrderWorkflow.ts

│   ├── PaymentWorkflow.ts

│   └── CustomerWorkflow.ts

├── validators/

├── schemas/

└── utils/

```

----------

# Enterprise Architecture

```text
Tests

↓

Workflow

↓

API Clients

↓

Validators

↓

Backend

```

Tests remain readable while complex orchestration is encapsulated.

----------

# Best Practices

-   Chain requests only when business logic requires it.
    
-   Implement retry logic for transient failures only.
    
-   Use polling instead of fixed delays for asynchronous operations.
    
-   Execute independent requests in parallel with `Promise.all()`.
    
-   Separate workflows from low-level API clients.
    
-   Validate both HTTP responses and business outcomes.
    

----------

# Common Mistakes

### ❌ Using `waitForTimeout()` Instead of Polling

Bad

```typescript
await page.waitForTimeout(10000);

```

Better

```text
Poll Until

↓

Completed

```

----------

### ❌ Infinite Polling

Always define:

-   Maximum retries
    
-   Timeout
    
-   Failure condition
    

----------

### ❌ Retrying Every Error

Do not retry:

-   400
    
-   401
    
-   403
    
-   404
    

These usually indicate business or authentication problems rather than temporary failures.

----------

### ❌ Sequential Execution of Independent Requests

Instead of

```typescript
await request.get("/users");

await request.get("/orders");

```

Use

```typescript
await Promise.all([
    request.get("/users"),
    request.get("/orders")
]);

```

when there is no dependency.

----------

### ❌ Mixing Workflow Logic Inside Tests

Avoid

```typescript
login();

createCustomer();

createOrder();

payOrder();

shipOrder();

```

inside every test.

Move workflows into reusable service classes.

----------

# Interview Questions

### Q1. Why is polling preferred over fixed waits?

Polling completes as soon as the required condition is met, making tests faster and more reliable than waiting for a fixed amount of time.

----------

### Q2. Which HTTP status codes are commonly retried?

Transient failures such as:

-   429 (Too Many Requests)
    
-   502 (Bad Gateway)
    
-   503 (Service Unavailable)
    
-   504 (Gateway Timeout)
    

----------

### Q3. Why use `Promise.all()` for API requests?

It executes independent requests concurrently, reducing total execution time.

----------

### Q4. What is an idempotency key?

An identifier sent with requests (commonly payment requests) to ensure that repeating the same request does not create duplicate operations.

----------

### Q5. Why should enterprise frameworks use workflow classes?

Workflow classes encapsulate multi-step business processes, keeping tests readable, reusable, and focused on business scenarios instead of implementation details.

----------

# Summary

Advanced API testing goes beyond individual requests. Enterprise automation often involves chained operations, asynchronous processing, polling, retries, file handling, rate limiting, API versioning, GraphQL interactions, and business workflows. By encapsulating these patterns into reusable workflow and service classes, teams can build scalable, maintainable, and production-ready API automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY4ODEzNjJdfQ==
-->
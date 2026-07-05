This chapter is one of the **most valuable chapters in the entire handbook**.

If I had to choose **one Playwright feature that separates a Senior Automation Engineer from a Mid-Level Automation Engineer**, it would be **understanding network events**.

Many UI failures are actually **API failures**.

Instead of debugging only the UI, experienced Playwright engineers inspect the network.

This chapter will teach exactly that.

----------

# Part 10 – Events

# Chapter 2 – Network Events

----------

# Introduction

Every modern web application communicates with backend services.

For example, when a user logs in:

```text
User Clicks Login
        │
        ▼
POST /login
        │
        ▼
Server Validates User
        │
        ▼
Returns Token
        │
        ▼
Dashboard Opens

```

Even though the user only sees a button click, several network events happen behind the scenes.

Playwright lets us observe these events.

----------

# Request Lifecycle

Every request follows a lifecycle.

```text
Request Created
      │
      ▼
Request Sent
      │
      ▼
Server Processing
      │
      ▼
Response Received
      │
      ▼
Request Finished

```

Or if something goes wrong:

```text
Request Created
      │
      ▼
Request Sent
      │
      ▼
Network Error
      │
      ▼
Request Failed

```

----------

# Network Events

Playwright exposes four important request events.

Event

Triggered When

request

Request is sent

response

Response headers are received

requestfinished

Request completes successfully

requestfailed

Request fails

Understanding when each event occurs is extremely important.

----------

# request Event

## Purpose

Triggered immediately after the browser sends a request.

Example:

```typescript
page.on('request', request => {

    console.log(request.url());

});

```

Every outgoing request is captured.

----------

## Example Output

```text
GET /products

POST /login

GET /orders

```

----------

## Capturing HTTP Method

```typescript
page.on('request', request => {

    console.log(request.method());

});

```

Output

```text
GET

POST

PUT

DELETE

```

----------

## Capturing URL

```typescript
page.on('request', request => {

    console.log(request.url());

});

```

----------

## Capturing Headers

```typescript
page.on('request', request => {

    console.log(request.headers());

});

```

Useful for:

-   Authorization
    
-   Custom headers
    
-   API version validation
    

----------

## Capturing POST Data

```typescript
page.on('request', request => {

    console.log(
        request.postData()
    );

});

```

Example

```json
{
  "username":"admin",
  "password":"password"
}

```

Very useful for debugging.

----------

# response Event

Triggered when response headers are received.

Example

```typescript
page.on('response', response => {

    console.log(response.status());

});

```

----------

## Capture Status Code

```typescript
page.on('response', response => {

    console.log(response.status());

});

```

Output

```text
200

404

500

```

----------

## Capture Response URL

```typescript
page.on('response', response => {

    console.log(response.url());

});

```

----------

## Read Response Body

```typescript
page.on('response', async response => {

    if (response.url().includes('/users')) {

        const body =
            await response.json();

        console.log(body);

    }

});

```

Excellent for API validation.

----------

# requestfinished Event

Occurs after the request completes successfully.

```typescript
page.on(
    'requestfinished',
    request => {

        console.log(
            request.url()
        );

    }
);

```

Useful when measuring request completion.

----------

# requestfailed Event

Triggered when a request fails.

```typescript
page.on(
    'requestfailed',
    request => {

        console.log(
            request.url()
        );

    }
);

```

----------

## Failure Example

```text
GET /products

↓

500 Internal Server Error

```

or

```text
DNS Failure

↓

Connection Refused

```

----------

## Failure Reason

```typescript
page.on(
    'requestfailed',
    request => {

        console.log(
            request.failure()
        );

    }
);

```

Example Output

```text
net::ERR_CONNECTION_RESET

```

----------

# Logging Every API Call

One of the most useful debugging utilities.

```typescript
page.on('request', request => {

    console.log(
        '>>',
        request.method(),
        request.url()
    );

});

page.on('response', response => {

    console.log(
        '<<',
        response.status(),
        response.url()
    );

});

```

Example output:

```text
>> POST /login
<< 200 /login

>> GET /products
<< 200 /products

>> GET /orders
<< 404 /orders

```

This provides a simple request/response log for your test.

----------

# Filtering Requests

Instead of logging everything:

```typescript
page.on('request', request => {

    if (
        request.url().includes('/login')
    ) {

        console.log(
            request.url()
        );

    }

});

```

----------

# Filtering Responses

```typescript
page.on('response', async response => {

    if (
        response.url().includes('/orders')
    ) {

        console.log(
            await response.json()
        );

    }

});

```

----------

# Capturing Authentication Token

Suppose:

```text
POST /login

↓

Returns JWT

```

```typescript
page.on('response', async response => {

    if (
        response.url().includes('/login')
    ) {

        const body =
            await response.json();

        console.log(
            body.token
        );

    }

});

```

Very useful for API testing or subsequent authenticated requests.

----------

# Measuring Response Time

```typescript
const start = Date.now();

page.on(
    'requestfinished',
    request => {

        console.log(
            Date.now() - start
        );

    }
);

```

> **Note:** This measures elapsed time from when `start` was recorded, not the true network duration of each request. For accurate performance measurements, use browser timing APIs or Playwright tracing. This simple approach can still be useful for rough diagnostics.

----------

# Real-World Example – Login Monitoring

```typescript
page.on('request', request => {

    if (
        request.url().includes('/login')
    ) {

        console.log(
            request.postData()
        );

    }

});

page.on('response', response => {

    if (
        response.url().includes('/login')
    ) {

        console.log(
            response.status()
        );

    }

});

await page.getByRole('button', {
    name: 'Login'
}).click();

```

Useful when investigating login issues.

----------

# Real-World Example – Product Search

```typescript
page.on('response', async response => {

    if (
        response.url().includes('/products')
    ) {

        const body =
            await response.json();

        console.log(body);

    }

});

await page.getByRole('button', {
    name: 'Search'
}).click();

```

----------

# Request vs Response

Request

Response

Outgoing

Incoming

Contains request payload

Contains response payload

Method

Status

Headers

Headers

POST data

JSON body

----------

# Request vs requestfinished

request

requestfinished

Sent immediately

Fires after successful completion

Good for logging outgoing traffic

Good for completion tracking

----------

# requestfinished vs requestfailed

```text
Request

↓

Sent

↓

Success

↓

requestfinished

```

or

```text
Request

↓

Sent

↓

Failure

↓

requestfailed

```

Only one of these terminal events occurs for a given request.

----------

# Best Practices

-   Filter network events to focus on relevant APIs rather than logging every request.
    
-   Use response handlers to inspect API responses during debugging.
    
-   Capture request payloads when validating client-side behavior.
    
-   Monitor failed requests during test execution to identify backend issues early.
    
-   Keep network event handlers lightweight to avoid slowing tests.
    

----------

# Common Mistakes

### ❌ Logging every request in every test

This creates noisy logs and makes failures harder to diagnose.

----------

### ❌ Assuming `request` means success

A request event only means the request was sent. It does **not** indicate that the server responded successfully.

----------

### ❌ Reading every response body

Parsing large responses unnecessarily can slow down test execution. Read bodies only for the APIs you're interested in.

----------

### ❌ Treating network monitoring as a substitute for assertions

Network logging is useful for debugging, but tests should still verify meaningful business outcomes through assertions.

----------

# Interview Questions

### Q1. What is the difference between `request` and `response` events?

-   `request` fires when the browser sends a request.
    
-   `response` fires when the browser receives the response headers.
    

----------

### Q2. When does `requestfinished` occur?

After a request completes successfully.

----------

### Q3. When does `requestfailed` occur?

When a request cannot be completed because of a network or transport error. Server responses such as HTTP 404 or 500 still produce a response event—they do not trigger `requestfailed`.

----------

### Q4. Can you inspect request headers and payloads?

Yes.

```typescript
request.headers();

request.postData();

```

----------

### Q5. Can you inspect response JSON?

Yes.

```typescript
const body = await response.json();

```

----------

# Summary

Network events provide deep visibility into how your application communicates with backend services. By monitoring requests, responses, successful completions, and failures, you can debug issues more effectively, validate API interactions, and understand the true cause of many UI problems. These event APIs also form the foundation for more advanced topics such as request interception and API mocking, which we'll cover later in the handbook.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk1MDkyNTE1XX0=
-->
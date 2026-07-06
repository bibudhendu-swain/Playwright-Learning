This chapter is one of the biggest differentiators between a **beginner API automation engineer** and an **enterprise API automation architect**.

Many engineers validate APIs like this:

```typescript
expect(response.status()).toBe(200);

```

That is **not enough**.

A response can return **200 OK** and still contain:

-   Wrong data
    
-   Missing fields
    
-   Incorrect business logic
    
-   Invalid schema
    
-   Empty arrays
    
-   Null values
    
-   Slow response times
    

Enterprise API testing validates **the entire response contract**, not just the status code.

----------

# Part 18 – API Testing

# Chapter 6 – API Response Validation

----------

# Introduction

Every API response should be validated.

Simply receiving a successful response does not guarantee the API behaved correctly.

Example

```text
Client

↓

GET /users

↓

200 OK

↓

Wrong Data

```

Your test should verify:

-   HTTP status
    
-   Headers
    
-   Response body
    
-   Business rules
    
-   Performance
    
-   Contract
    

----------

# Response Validation Flow

```text
API Request

↓

HTTP Response

↓

Status Validation

↓

Header Validation

↓

Body Validation

↓

Business Validation

↓

Test Pass

```

----------

# Levels of Validation

Enterprise teams usually validate responses at multiple levels.

Level

Example

Status Code

200 OK

Headers

Content-Type

Response Body

JSON

Schema

Required fields

Business Rules

Correct values

Performance

< 1000 ms

----------

# Status Code Validation

The first validation is always the HTTP status.

```typescript
const response = await request.get("/users");

expect(response.status())

.toBe(200);

```

Common status codes:

Status

Meaning

200

Success

201

Created

202

Accepted

204

No Content

400

Bad Request

401

Unauthorized

403

Forbidden

404

Not Found

409

Conflict

500

Internal Server Error

----------

# Using ok()

Instead of

```typescript
expect(response.status())

.toBe(200);

```

You may also use

```typescript
expect(response.ok())

.toBeTruthy();

```

Remember:

`ok()` returns true for all successful **2xx** responses.

----------

# Response Header Validation

Always verify important headers.

Example

```typescript
expect(

response.headers()["content-type"]

)

.toContain("application/json");

```

Another example

```typescript
expect(

response.headers()["cache-control"]

)

.toContain("no-cache");

```

----------

# JSON Response Validation

```typescript
const body =

await response.json();

expect(body.id)

.toBe(10);

expect(body.name)

.toBe("John");

```

----------

# Object Validation

Instead of validating one property,

validate multiple fields.

```typescript
expect(body).toEqual({

    id: 10,

    name: "John",

    city: "London"

});

```

----------

# Partial Object Matching

Sometimes APIs return many fields.

You only care about a few.

```typescript
expect(body).toMatchObject({

    id: 10,

    name: "John"

});

```

Useful when APIs include generated timestamps or IDs.

----------

# Property Validation

```typescript
expect(body)

.toHaveProperty("id");

expect(body)

.toHaveProperty("email");

```

Nested property

```typescript
expect(body)

.toHaveProperty(

"address.city"

);

```

----------

# Type Validation

```typescript
expect(typeof body.id)

.toBe("number");

expect(typeof body.name)

.toBe("string");

```

----------

# Null Validation

```typescript
expect(body.email)

.not.toBeNull();

```

----------

# Undefined Validation

```typescript
expect(body.phone)

.toBeUndefined();

```

----------

# Boolean Validation

```typescript
expect(body.active)

.toBe(true);

```

----------

# Array Validation

Example response

```json
[
    {
        "id":1
    },
    {
        "id":2
    }
]

```

Validation

```typescript
expect(body.length)

.toBeGreaterThan(0);

```

----------

# Validate Array Items

```typescript
body.forEach(user => {

    expect(user.id)

        .toBeDefined();

    expect(user.name)

        .toBeTruthy();

});

```

----------

# Array Contains

```typescript
expect(body)

.toContainEqual(

    expect.objectContaining({

        id: 10

    })

);

```

----------

# Nested Object Validation

Example

```json
{
    "customer":{

        "address":{

            "city":"London"

        }

    }
}

```

Validation

```typescript
expect(

body.customer.address.city

)

.toBe("London");

```

----------

# Deep Object Matching

```typescript
expect(body)

.toStrictEqual(expectedCustomer);

```

Useful when validating complete API contracts.

----------

# Schema Validation

Enterprise projects often validate response schemas.

Example schema

```json
{
    "id":"number",

    "name":"string",

    "email":"string"
}

```

Libraries such as **AJV** are commonly used.

Example

```typescript
const valid = validate(body);

expect(valid)

.toBeTruthy();

```

This ensures required fields and types are correct.

----------

# Required Fields

Verify mandatory fields exist.

```typescript
expect(body)

.toHaveProperty("id");

expect(body)

.toHaveProperty("name");

expect(body)

.toHaveProperty("email");

```

----------

# Business Rule Validation

Example

```json
{
    "price":1200,

    "discount":200,

    "finalPrice":1000
}

```

Validate

```typescript
expect(

body.price -

body.discount

)

.toBe(

body.finalPrice

);

```

Business validation is often more valuable than simply checking field values.

----------

# Date Validation

```typescript
expect(

new Date(body.createdDate)

.toString()

)

.not.toContain(

"Invalid"

);

```

----------

# Regex Validation

Email

```typescript
expect(body.email)

.toMatch(

/^[^\s@]+@[^\s@]+\.[^\s@]+$/

);

```

Phone

```typescript
expect(body.phone)

.toMatch(

/^\d{10}$/

);

```

----------

# Response Time Validation

Measure response time.

```typescript
const start = Date.now();

const response =

await request.get("/users");

const duration =

Date.now() - start;

expect(duration)

.toBeLessThan(1000);

```

Useful for performance thresholds.

----------

# Empty Response Validation

Example

```http
204 No Content

```

Validation

```typescript
expect(

response.status()

)

.toBe(204);

expect(

await response.text()

)

.toBe("");

```

----------

# Error Response Validation

Example

```json
{
    "message":"User not found",

    "error":"Not Found"
}

```

Validation

```typescript
expect(

response.status()

)

.toBe(404);

const body =

await response.json();

expect(body.message)

.toBe(

"User not found"

);

```

----------

# Contract Testing

Contract validation ensures the API response still matches the agreed format.

Example

```text
Frontend

↓

Expected Schema

↓

Backend

↓

Same Schema

```

If the backend removes a field unexpectedly, the contract test fails.

----------

# Enterprise Validation Helper

Instead of repeating validations,

create reusable helpers.

```typescript
class ApiValidator {

    static async success(response){

        expect(response.ok())

            .toBeTruthy();

        expect(

            response.headers()["content-type"]

        )

        .toContain(

            "application/json"

        );

    }

}

```

Usage

```typescript
await ApiValidator.success(response);

```

----------

# Base Response Validator

```typescript
class BaseValidator {

    static validateStatus(

        response,

        expected

    ){

        expect(

            response.status()

        )

        .toBe(expected);

    }

}

```

----------

# Suggested Folder Structure

```text
api/

├── validators/

│   ├── ApiValidator.ts

│   ├── CustomerValidator.ts

│   ├── OrderValidator.ts

│   └── ProductValidator.ts

├── schemas/

│   ├── customer.schema.json

│   ├── product.schema.json

│   └── order.schema.json

└── utils/

```

----------

# Validation Lifecycle

```text
Response

↓

Status

↓

Headers

↓

Schema

↓

Business Rules

↓

Performance

↓

Pass

```

----------

# Enterprise Example

Customer API

```json
{
    "id":10,

    "name":"John",

    "email":"john@test.com",

    "status":"ACTIVE"
}

```

Validation

```typescript
expect(body.id)

.toBeGreaterThan(0);

expect(body.status)

.toBe("ACTIVE");

expect(body.email)

.toContain("@");

```

----------

# Best Practices

-   Validate more than just the HTTP status code.
    
-   Use `toMatchObject()` for partial validation when appropriate.
    
-   Separate schema validation from business rule validation.
    
-   Create reusable validators for common response types.
    
-   Validate performance for critical APIs.
    
-   Keep validation logic out of test cases whenever possible.
    

----------

# Common Mistakes

### ❌ Validating Only Status Code

Bad

```typescript
expect(response.status())

.toBe(200);

```

Good

```text
Status

↓

Headers

↓

Body

↓

Business Rules

```

----------

### ❌ Hardcoding Entire Responses

Avoid validating every field if only a subset matters.

Prefer

```typescript
toMatchObject()

```

for flexible assertions.

----------

### ❌ Ignoring Error Responses

Always test

-   400
    
-   401
    
-   403
    
-   404
    
-   409
    
-   500
    

Success scenarios alone are not sufficient.

----------

### ❌ Mixing Validation Logic Into Tests

Instead of

```typescript
expect(...)

expect(...)

expect(...)

```

inside every test,

move validations into reusable validator classes.

----------

# Interview Questions

### Q1. Is validating only the HTTP status code sufficient?

No. Enterprise API tests should also validate headers, response body, business rules, schema, and sometimes performance.

----------

### Q2. What is the difference between `toEqual()` and `toMatchObject()`?

-   `toEqual()` requires the entire object to match.
    
-   `toMatchObject()` validates only the specified properties, allowing additional fields.
    

----------

### Q3. Why is JSON Schema validation useful?

It verifies that the API response structure, required fields, and data types remain consistent, helping detect contract-breaking changes.

----------

### Q4. Why should response time be validated?

To ensure APIs meet expected performance requirements and to detect regressions before they impact users.

----------

### Q5. Why should validation logic be centralized?

Centralized validators reduce duplication, improve maintainability, and enforce consistent validation across the test suite.

----------

# Summary

API response validation is much more than checking for a `200 OK`. Enterprise-grade API automation validates HTTP status codes, headers, response bodies, schemas, business rules, error handling, and performance. By organizing this logic into reusable validators and schema checks, teams can build reliable contract tests that quickly detect both functional and structural regressions.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAzNDAzODQ2XX0=
-->
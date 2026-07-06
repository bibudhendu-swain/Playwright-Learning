This chapter is the heart of Playwright API testing.

Every REST API revolves around HTTP methods. Once someone understands these methods well, they can automate almost any REST service.

In this chapter, we'll not only cover the Playwright syntax but also explain the HTTP concepts, enterprise usage, and best practices.

----------

# Part 18 – API Testing

# Chapter 3 – HTTP Methods (GET, POST, PUT, PATCH, DELETE, HEAD & OPTIONS)

----------

# Introduction

Every interaction with a REST API uses an **HTTP Method**.

Think of HTTP methods as verbs.

HTTP Method

Meaning

GET

Read

POST

Create

PUT

Replace

PATCH

Update

DELETE

Remove

HEAD

Retrieve headers only

OPTIONS

Discover supported operations

Example

```text
Customer API

GET      /customers

POST     /customers

PUT      /customers/10

PATCH    /customers/10

DELETE   /customers/10

```

----------

# CRUD Operations

Most APIs follow CRUD.

```text
Create

↓

Read

↓

Update

↓

Delete

```

HTTP mapping

CRUD

HTTP Method

Create

POST

Read

GET

Update

PUT / PATCH

Delete

DELETE

----------

# GET Request

## Purpose

Retrieve data.

No data should be modified.

Example

```text
GET

↓

Server

↓

Customer List

```

----------

## Syntax

```typescript
const response = await request.get(

    "/customers"

);

```

----------

## Example

```typescript
test("Get all customers", async ({ request }) => {

    const response = await request.get(

        "/customers"

    );

    expect(response.status())

        .toBe(200);

    const customers =

        await response.json();

    expect(customers.length)

        .toBeGreaterThan(0);

});

```

----------

## GET with Query Parameters

Suppose

```text
/customers?page=2

```

Instead of concatenating strings,

Playwright provides:

```typescript
const response = await request.get(

    "/customers",

    {

        params: {

            page: 2,

            size: 20

        }

    }

);

```

Generated URL

```text
/customers?page=2&size=20

```

----------

## GET with Headers

```typescript
await request.get(

    "/customers",

    {

        headers: {

            Authorization:

                `Bearer ${token}`

        }

    }

);

```

----------

# POST Request

## Purpose

Create new data.

Example

```text
POST

↓

Customer

↓

Database

↓

Created

```

----------

## Syntax

```typescript
await request.post(

    "/customers",

    {

        data: {

            name: "John"

        }

    }

);

```

----------

## Example

```typescript
test("Create Customer", async ({ request }) => {

    const response =

    await request.post(

        "/customers",

        {

            data: {

                firstName: "John",

                lastName: "Doe",

                city: "New York"

            }

        }

    );

    expect(response.status())

        .toBe(201);

});

```

----------

# Sending JSON

Playwright automatically serializes JavaScript objects when using:

```typescript
data:

```

Example

```typescript
data:{

name:"Laptop"

}

```

becomes

```json
{
  "name":"Laptop"
}

```

No need for

```typescript
JSON.stringify()

```

in most cases.

----------

# Sending Form Data

Some APIs accept URL-encoded form data instead of JSON.

```typescript
await request.post("/login", {

    form: {

        username: "admin",

        password: "password"

    }

});

```

----------

# Sending Multipart Form Data

Useful for file uploads.

```typescript
import fs from "fs";

await request.post("/upload", {

    multipart: {

        file: {

            name: "report.pdf",

            mimeType: "application/pdf",

            buffer: fs.readFileSync("./files/report.pdf")

        }

    }

});

```

----------

# PUT Request

## Purpose

Replace an existing resource.

Think of PUT as:

```text
Old Object

↓

Replace Entire Object

↓

New Object

```

----------

## Example

```typescript
await request.put(

    "/customers/10",

    {

        data: {

            id: 10,

            firstName: "John",

            lastName: "Smith",

            city: "Chicago"

        }

    }

);

```

Usually,

the complete object is sent.

----------

# PATCH Request

## Purpose

Modify only selected fields.

Example

```text
Customer

↓

City Changed

↓

PATCH

```

----------

## Example

```typescript
await request.patch(

    "/customers/10",

    {

        data: {

            city: "Chicago"

        }

    }

);

```

Only

```text
city

```

changes.

----------

# PUT vs PATCH

PUT

PATCH

Replace entire resource

Update specific fields

Larger payload

Smaller payload

Complete object

Partial object

Usually idempotent

Often idempotent, depending on API implementation

----------

# DELETE Request

## Purpose

Delete existing data.

Example

```text
DELETE

↓

Customer Removed

```

----------

## Example

```typescript
await request.delete(

    "/customers/10"

);

```

Verify

```typescript
expect(response.status())

.toBe(204);

```

or

```typescript
.toBe(200);

```

depending on API implementation.

----------

# HEAD Request

HEAD is similar to GET,

but returns only headers.

```text
HEAD

↓

Headers

↓

No Body

```

Useful for:

-   File existence
    
-   Metadata
    
-   Content-Length
    
-   Last-Modified
    

----------

## Example

```typescript
const response =

await request.head(

    "/files/report.pdf"

);

expect(

response.headers()["content-type"]

)

.toContain("application/pdf");

```

----------

# OPTIONS Request

OPTIONS tells us which methods an endpoint supports.

Example

```text
OPTIONS

↓

GET

POST

PUT

DELETE

```

----------

## Example

```typescript
const response =

await request.fetch("/customers", {

    method: "OPTIONS"

});

expect(response.status())

.toBe(200);

```

> **Note:** Playwright does not provide a dedicated `request.options()` helper. Use `request.fetch()` (or another generic request mechanism) with `method: "OPTIONS"`.

----------

# Request Body

Many methods support request bodies.

```text
POST

PUT

PATCH

```

Example

```typescript
data:{

id:1,

name:"Laptop"

}

```

----------

# Query Parameters

Useful for filtering.

Example

```text
/products?

category=laptop

&page=2

```

Playwright

```typescript
params:{

category:"laptop",

page:2

}

```

----------

# Response Validation

Always validate more than status.

```typescript
expect(response.status())

.toBe(200);

expect(response.ok())

.toBeTruthy();

const body =

await response.json();

expect(body.id)

.toBe(10);

```

----------

# Idempotent Methods

An important interview topic.

## Idempotent

Calling multiple times produces the same end result.

```text
GET

PUT

DELETE

HEAD

OPTIONS

```

----------

## Non-Idempotent

Every call creates new state.

```text
POST

```

Example

```text
POST

↓

Customer Created

POST Again

↓

Another Customer Created

```

----------

# Enterprise CRUD Example

```text
POST

↓

Create Customer

↓

GET

↓

Verify Customer

↓

PATCH

↓

Update City

↓

GET

↓

Verify Update

↓

DELETE

↓

Remove Customer

↓

GET

↓

404

```

One complete API workflow.

----------

# API Test Structure

```text
Arrange

↓

Send Request

↓

Validate Response

↓

Validate Business Logic

```

Example

```typescript
const response =

await request.post(...);

expect(response.status())

.toBe(201);

const customer =

await response.json();

expect(customer.firstName)

.toBe("John");

```

----------

# Enterprise API Client

Instead of

```typescript
await request.post(...);

await request.get(...);

```

Create domain-specific methods.

```typescript
class CustomerApi{

async createCustomer(){

}

async getCustomer(){

}

async deleteCustomer(){

}

}

```

Usage

```typescript
await customerApi.createCustomer();

await customerApi.deleteCustomer();

```

----------

# Suggested Folder Structure

```text
api/

├── BaseApi.ts

├── CustomerApi.ts

├── ProductApi.ts

├── OrderApi.ts

└── AuthApi.ts

```

----------

# HTTP Method Flow

```text
Client

↓

GET

↓

Read

-----------------

POST

↓

Create

-----------------

PUT

↓

Replace

-----------------

PATCH

↓

Update

-----------------

DELETE

↓

Remove

```

----------

# Best Practices

-   Use the correct HTTP method for the intended operation.
    
-   Use `params` for query parameters instead of manually building URLs.
    
-   Validate both HTTP status codes and response content.
    
-   Keep API payloads in reusable builders or fixtures.
    
-   Wrap endpoint calls in reusable API client classes.
    

----------

# Common Mistakes

### ❌ Using POST for Updates

Use

```text
PUT

or

PATCH

```

when updating existing resources.

----------

### ❌ Building Query Strings Manually

Instead of

```typescript
"/users?page=1&size=10"

```

use

```typescript
params:{}

```

----------

### ❌ Checking Only Status Codes

Always validate:

-   Response body
    
-   Headers
    
-   Business rules
    
-   Data integrity
    

----------

### ❌ Confusing PUT and PATCH

Remember:

```text
PUT

↓

Replace Everything

-----------------

PATCH

↓

Update Only Changes

```

----------

# Interview Questions

### Q1. What is the difference between PUT and PATCH?

PUT replaces the entire resource, while PATCH updates only the specified fields.

----------

### Q2. Which HTTP method is typically used to create a new resource?

POST.

----------

### Q3. Which HTTP methods are generally considered idempotent?

GET, PUT, DELETE, HEAD, and OPTIONS are generally considered idempotent. PATCH **may or may not be idempotent** depending on the API implementation.

----------

### Q4. How do you send query parameters in Playwright?

```typescript
await request.get("/products", {

    params: {

        category: "Laptop",

        page: 1

    }

});

```

----------

### Q5. What is the purpose of the HEAD method?

HEAD retrieves only the response headers without the response body, making it useful for checking metadata such as content type, content length, or resource existence.

----------

# Summary

HTTP methods define how clients interact with REST APIs. Playwright provides first-class support for all common HTTP operations, allowing you to create, retrieve, update, delete, and inspect resources using a consistent API. Understanding the differences between GET, POST, PUT, PATCH, DELETE, HEAD, and OPTIONS is fundamental to building robust API automation and designing reusable API client libraries.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcyMjI0OTU5MF19
-->
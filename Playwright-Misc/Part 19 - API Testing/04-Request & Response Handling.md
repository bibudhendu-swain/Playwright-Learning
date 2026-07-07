This chapter is where we move from **sending API requests** to **mastering HTTP communication**.

Most beginners know how to do this:

```typescript
const response = await request.post("/users", {
    data: {
        name: "John"
    }
});

```

But enterprise API testing requires much more than that.

You need to know:

-   How to send headers
    
-   How to work with cookies
    
-   How to upload files
    
-   How to send forms
    
-   How to read binary responses
    
-   How to validate headers
    
-   How to handle compressed responses
    
-   How to build reusable request wrappers
    

This chapter covers all of those.

----------

# Part 19 – API Testing

# Chapter 4 – Request & Response Handling

----------

# Introduction

Every HTTP interaction consists of two parts:

```text
Request

↓

Server

↓

Response

```

A request contains information sent to the server.

A response contains information returned by the server.

Understanding both is essential for effective API automation.

----------

# HTTP Request Structure

A request typically contains:

```text
Request

│

├── URL

├── HTTP Method

├── Headers

├── Query Parameters

├── Cookies

└── Body

```

Example

```http
POST /users?page=1 HTTP/1.1

Authorization: Bearer xxx

Content-Type: application/json

Cookie: session=abc

{
    "name":"John"
}

```

----------

# HTTP Response Structure

A response contains:

```text
Response

│

├── Status Code

├── Headers

├── Cookies

└── Body

```

Example

```http
HTTP/1.1 200 OK

Content-Type: application/json

Set-Cookie: session=xyz

{
    "id":10
}

```

----------

# Request Headers

Headers provide metadata about the request.

Common examples:

Header

Purpose

Authorization

Authentication

Accept

Expected response type

Content-Type

Request body format

User-Agent

Client information

Accept-Language

Preferred language

----------

# Sending Headers

```typescript
const response = await request.get(

    "/users",

    {

        headers: {

            Authorization:

                `Bearer ${token}`,

            Accept:

                "application/json"

        }

    }

);

```

----------

# Default Headers

Instead of repeating headers:

```typescript
Authorization

Content-Type

Accept

```

Configure them once.

```typescript
const api =
await request.newContext({

    extraHTTPHeaders: {

        Authorization:

            `Bearer ${token}`

    }

});

```

----------

# Reading Response Headers

```typescript
const response =
await request.get("/users");

const headers =
response.headers();

console.log(headers);

```

----------

# Reading Individual Header

```typescript
expect(

response.headers()["content-type"]

)

.toContain("application/json");

```

----------

# Query Parameters

Query parameters are commonly used for:

-   Pagination
    
-   Filtering
    
-   Searching
    
-   Sorting
    

Example

```text
/users?page=2&size=20

```

----------

# Sending Query Parameters

```typescript
await request.get(

    "/users",

    {

        params: {

            page: 2,

            size: 20,

            sort: "name"

        }

    }

);

```

Generated URL

```text
/users?page=2&size=20&sort=name

```

----------

# Request Body

POST, PUT and PATCH usually contain a body.

Example

```json
{
    "name":"John",

    "city":"London"
}

```

----------

# JSON Request

```typescript
await request.post(

    "/users",

    {

        data: {

            name: "John",

            city: "London"

        }

    }

);

```

Playwright automatically:

-   Converts to JSON
    
-   Sets `Content-Type: application/json`
    

----------

# Sending Raw JSON

Sometimes you already have serialized JSON.

```typescript
await request.post(

    "/users",

    {

        data:

            JSON.stringify({

                name: "John"

            }),

        headers: {

            "Content-Type":

                "application/json"

        }

    }

);

```

Generally, prefer passing JavaScript objects unless you specifically need raw JSON.

----------

# Form Data

Many authentication APIs still use URL-encoded forms.

```typescript
await request.post(

    "/login",

    {

        form: {

            username: "admin",

            password: "secret"

        }

    }

);

```

Playwright automatically sends the appropriate `application/x-www-form-urlencoded` content type.

----------

# Multipart Form Data

Useful for:

-   Image upload
    
-   PDF upload
    
-   CSV import
    
-   Attachments
    

```typescript
import fs from "fs";

await request.post(

    "/upload",

    {

        multipart: {

            file: {

                name: "report.pdf",

                mimeType:

                    "application/pdf",

                buffer:

                    fs.readFileSync(

                        "./files/report.pdf"

                    )

            }

        }

    }

);

```

----------

# Upload Multiple Files

```typescript
await request.post("/upload", {

    multipart: {

        profile: {

            name: "profile.png",

            mimeType: "image/png",

            buffer: fs.readFileSync("./files/profile.png")

        },

        resume: {

            name: "resume.pdf",

            mimeType: "application/pdf",

            buffer: fs.readFileSync("./files/resume.pdf")

        }

    }

});

```

----------

# Cookies

Cookies often maintain authenticated sessions.

Server

```text
Login

↓

Set-Cookie

↓

Browser/API Client

```

Later requests automatically reuse cookies within the same `APIRequestContext`.

----------

# Reading Cookies

If you're sharing state with a browser context or inspecting cookies, you typically use the browser context APIs. For API requests, cookies are managed automatically by the `APIRequestContext`.

A common enterprise validation is to verify the response includes a cookie.

```typescript
const setCookie =
response.headers()["set-cookie"];

expect(setCookie)

.toContain("session");

```

----------

# Reading JSON Response

```typescript
const response =

await request.get("/users");

const users =

await response.json();

```

----------

# Reading Text Response

```typescript
const text =

await response.text();

```

Useful for:

-   HTML
    
-   XML
    
-   Plain text
    

----------

# Reading Binary Response

Suppose the API downloads:

-   PDF
    
-   ZIP
    
-   Excel
    

Use

```typescript
const buffer =

await response.body();

```

Returns

```text
Buffer

```

----------

# Saving Downloaded File

```typescript
import fs from "fs";

const buffer =
await response.body();

fs.writeFileSync(

    "./downloads/report.pdf",

    buffer

);

```

Useful for validating generated reports.

----------

# Reading Response Status

```typescript
expect(

response.status()

)

.toBe(200);

```

----------

# Checking Success

```typescript
expect(

response.ok()

)

.toBeTruthy();

```

Equivalent to checking for a successful 2xx response.

----------

# Reading Status Text

```typescript
expect(

response.statusText()

)

.toBe("OK");

```

Useful when validating HTTP responses in detail.

----------

# Response Validation

Never validate only

```typescript
status

```

Validate

```text
Status

↓

Headers

↓

Body

↓

Business Rules

```

Example

```typescript
expect(

response.status()

).toBe(200);

expect(

response.headers()["content-type"]

)

.toContain("application/json");

const body =
await response.json();

expect(body.id)

.toBe(10);

```

----------

# Compression

Most servers compress responses.

Example

```http
Content-Encoding: gzip

```

Playwright automatically decompresses responses before you call:

```typescript
await response.text();

await response.json();

```

Normally, no extra work is required.

----------

# Large Responses

Suppose

```text
100 MB JSON

```

Avoid repeatedly parsing the same response.

```typescript
const body =
await response.json();

```

Store it once.

Reuse it.

----------

# Handling Empty Responses

Some DELETE APIs return:

```http
204 No Content

```

Calling

```typescript
await response.json();

```

will fail because there is no body.

Instead

```typescript
expect(

response.status()

)

.toBe(204);

```

or

```typescript
const text =

await response.text();

expect(text)

.toBe("");

```

----------

# Enterprise Request Wrapper

Instead of

```typescript
await request.post(...);

await request.get(...);

await request.put(...);

```

Create reusable helpers.

```typescript
class BaseApi {

    constructor(

        protected api:
        APIRequestContext

    ) {}

    async get(url:string){

        return this.api.get(url);

    }

}

```

Every API class inherits it.

----------

# Enterprise Response Wrapper

Instead of

```typescript
const body =
await response.json();

```

Use

```typescript
class ApiResponse<T>{

    constructor(

        public data:T,

        public status:number

    ){}

}

```

Now API methods can return strongly typed responses.

----------

# Suggested Folder Structure

```text
api/

├── BaseApi.ts

├── ApiResponse.ts

├── UserApi.ts

├── ProductApi.ts

└── OrderApi.ts

```

----------

# Request/Response Lifecycle

```text
Build Request

↓

Headers

↓

Body

↓

Server

↓

Response

↓

Headers

↓

Body

↓

Validation

```

----------

# Best Practices

-   Configure common headers in the request context.
    
-   Use `params` instead of manually building query strings.
    
-   Prefer JavaScript objects over manually serialized JSON.
    
-   Validate response headers and body—not just the status code.
    
-   Handle empty responses safely before calling `response.json()`.
    
-   Store reusable request and response logic in shared base classes.
    

----------

# Common Mistakes

### ❌ Building Query Strings Manually

Bad

```typescript
"/users?page=1&size=10"

```

Better

```typescript
params:{

page:1,

size:10

}

```

----------

### ❌ Calling `response.json()` on 204 Responses

There is no JSON body.

Validate the status instead.

----------

### ❌ Ignoring Headers

Many APIs communicate important information through:

-   Rate limits
    
-   Pagination
    
-   Tokens
    
-   Cookies
    

Always validate headers when they are part of the contract.

----------

### ❌ Repeating Request Logic

Avoid writing

```typescript
request.get()

request.post()

request.delete()

```

inside every test.

Move them into reusable API classes.

----------

# Real-World Example – File Download API

```typescript
test("Download invoice", async ({ request }) => {

    const response = await request.get("/invoices/123/pdf");

    expect(response.status()).toBe(200);

    expect(response.headers()["content-type"])
        .toContain("application/pdf");

    const pdf = await response.body();

    expect(pdf.length).toBeGreaterThan(0);
});

```

This validates both the HTTP response and that the downloaded file isn't empty.

----------

# Real-World Example – Paginated API

```typescript
const response = await request.get("/users", {
    params: {
        page: 1,
        size: 25
    }
});

const users = await response.json();

expect(users.items.length).toBeLessThanOrEqual(25);

```

----------

# Interview Questions

### Q1. What is the difference between request headers and response headers?

-   **Request headers** are sent by the client to describe the request.
    
-   **Response headers** are returned by the server to describe the response.
    

----------

### Q2. How do you send query parameters in Playwright?

```typescript
await request.get("/users", {
    params: {
        page: 1,
        size: 20
    }
});

```

----------

### Q3. What method should be used to read binary responses?

```typescript
await response.body();

```

----------

### Q4. Why shouldn't you always call `response.json()`?

Some responses (such as **204 No Content**) have no response body. Calling `json()` in those cases will fail.

----------

### Q5. Why should enterprise frameworks wrap request and response handling?

To centralize common logic, improve maintainability, enforce consistent validation, and provide reusable, strongly typed API abstractions.

----------

# Summary

Effective API automation goes far beyond sending HTTP requests. Enterprise-quality tests require a solid understanding of request construction, headers, query parameters, payloads, cookies, file uploads, response parsing, and validation. By encapsulating this logic into reusable request and response wrappers, teams can build scalable, maintainable API frameworks that are easy to extend and reuse.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzgwNDY4NTQ2XX0=
-->
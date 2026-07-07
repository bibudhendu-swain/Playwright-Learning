 This is arguably **the most important chapter** in the API Testing section.

In almost every enterprise project, before you can call any API, you must authenticate.

Examples include:

-   Azure AD
    
-   OAuth 2.0
    
-   JWT Tokens
    
-   Bearer Tokens
    
-   API Keys
    
-   Basic Authentication
    
-   Session Cookies
    
-   CSRF Tokens
    
-   SSO
    

One of Playwright's biggest strengths is that **API authentication can also be reused for UI automation**, significantly reducing test execution time.

----------

# Part 19 – API Testing

# Chapter 5 – Authentication

----------

# Introduction

Most enterprise APIs are protected.

Without authentication:

```text
Client

↓

GET /customers

↓

401 Unauthorized

```

With authentication:

```text
Client

↓

Bearer Token

↓

GET /customers

↓

200 OK

```

Authentication proves **who you are**.

Authorization determines **what you can access**.

----------

# Authentication vs Authorization

Many engineers confuse these two concepts.

Authentication

Authorization

Who are you?

What can you access?

Login

Permissions

Identity

Access Control

Username & Password

Roles

Example

```text
Login

↓

Authenticated

↓

Role = Admin

↓

Allowed to Delete User

```

----------

# Common Authentication Types

Modern APIs commonly use:

Authentication

Used In

Basic Authentication

Legacy APIs

Bearer Token

REST APIs

JWT

Microservices

OAuth 2.0

Google, Azure, GitHub

API Key

Third-party APIs

Session Cookie

Traditional Web Apps

CSRF Token

Secure Form Submission

----------

# Authentication Flow

```text
Login

↓

Credentials

↓

Authentication Server

↓

Access Token

↓

Protected APIs

```

----------

# Basic Authentication

Basic Authentication sends username and password in every request.

Header

```http
Authorization: Basic base64(username:password)

```

----------

## Playwright Example

```typescript
const api =
await request.newContext({

    httpCredentials: {

        username: "admin",

        password: "secret"

    }

});

```

Now every request automatically includes the Basic Authentication header.

----------

# Bearer Token Authentication

The most common authentication mechanism.

Workflow

```text
Login

↓

Bearer Token

↓

Authorization Header

↓

Protected APIs

```

----------

## Header

```http
Authorization: Bearer eyJhbGciOi...

```

----------

## Playwright Example

```typescript
const token = "eyJhbGciOi...";

const api =
await request.newContext({

    extraHTTPHeaders: {

        Authorization:

            `Bearer ${token}`

    }

});

```

Every request now uses the token.

----------

# Login API

Instead of hardcoding tokens,

authenticate through the application's login endpoint.

Example

```typescript
const loginResponse =
await request.post(

    "/login",

    {

        data: {

            username: "admin",

            password: "password"

        }

    }

);

const loginBody =
await loginResponse.json();

const token =
loginBody.accessToken;

```

----------

# Reusing Token

```typescript
const api =
await request.newContext({

    extraHTTPHeaders:{

        Authorization:

        `Bearer ${token}`

    }

});

```

Now

```typescript
await api.get("/users");

await api.get("/orders");

await api.post("/products");

```

----------

# JWT Authentication

JWT stands for

```text
JSON Web Token

```

Structure

```text
Header

↓

Payload

↓

Signature

```

JWT looks like:

```text
xxxxx.yyyyy.zzzzz

```

The payload often contains:

-   User ID
    
-   Roles
    
-   Expiration
    
-   Issuer
    

----------

# Example JWT Header

```http
Authorization: Bearer eyJhbGc...

```

Playwright treats JWT like any other Bearer token.

----------

# OAuth 2.0

OAuth is used by:

-   Google
    
-   Microsoft
    
-   GitHub
    
-   Facebook
    
-   Salesforce
    

Typical flow

```text
User

↓

Login Page

↓

Authorization Server

↓

Access Token

↓

API

```

For automated testing, OAuth tokens are usually obtained through a login API, client credentials flow, or pre-generated test credentials.

----------

# Client Credentials Flow

Machine-to-machine APIs commonly use:

```text
Client ID

↓

Client Secret

↓

Token Endpoint

↓

Access Token

```

Example

```typescript
const tokenResponse =
await request.post(

    "/oauth/token",

    {

        form: {

            grant_type:

                "client_credentials",

            client_id:

                "client",

            client_secret:

                "secret"

        }

    }

);

```

----------

# API Key Authentication

Some APIs use API keys instead of tokens.

Header

```http
x-api-key: abc123

```

Example

```typescript
await request.get(

    "/weather",

    {

        headers: {

            "x-api-key":

                "abc123"

        }

    }

);

```

Some APIs instead require the key as a query parameter (for example, `?api_key=...`), depending on the provider.

----------

# Cookie Authentication

Traditional web applications often authenticate using session cookies.

Workflow

```text
Login

↓

Set-Cookie

↓

Session

↓

Protected APIs

```

Playwright automatically stores cookies within the same `APIRequestContext`.

Example

```typescript
await api.post("/login", {

    data: {

        username: "admin",

        password: "password"

    }

});

const response =
await api.get("/profile");

```

The session cookie is sent automatically.

----------

# CSRF Tokens

Many web applications protect POST requests using CSRF tokens.

Workflow

```text
Open Login Page

↓

Receive CSRF Token

↓

Send Login Request

↓

Authenticated

```

Example

```http
X-CSRF-Token: abc123

```

Playwright sends it like any other header.

```typescript
await request.post(

    "/login",

    {

        headers: {

            "X-CSRF-Token":

                csrfToken

        }

    }

);

```

----------

# Refresh Tokens

Access tokens eventually expire.

Typical workflow

```text
Access Token Expired

↓

Refresh Token

↓

New Access Token

↓

Continue

```

Example

```typescript
const refreshResponse =
await request.post(

    "/refresh",

    {

        data: {

            refreshToken

        }

    }

);

const body =
await refreshResponse.json();

const newToken =
body.accessToken;

```

Enterprise frameworks often refresh tokens automatically when needed.

----------

# Handling 401 Unauthorized

Example

```typescript
const response =
await api.get("/orders");

if (response.status() === 401) {

    // Refresh token

    // Retry request

}

```

This retry logic is commonly centralized inside an API client.

----------

# Sharing Authentication Between API and UI

One of Playwright's most powerful features.

Traditional UI

```text
Open Browser

↓

Login

↓

Navigate

↓

Run Test

```

Optimized Playwright

```text
API Login

↓

Save Authentication State

↓

Open Browser

↓

Already Logged In

```

This can reduce execution time significantly.

We'll cover `storageState` in a dedicated chapter.

----------

# Authentication Manager

Instead of

```typescript
const token = ...

const api = ...

const response = ...

```

Create a reusable manager.

```typescript
class AuthManager {

    async getToken(){

        // Login API

    }

}

```

Usage

```typescript
const token =
await authManager.getToken();

```

----------

# Enterprise API Client

```typescript
class BaseApi {

    constructor(

        protected api:
        APIRequestContext

    ) {}

}

```

Every API client shares the same authenticated context.

----------

# Suggested Folder Structure

```text
api/

├── AuthApi.ts

├── TokenManager.ts

├── BaseApi.ts

├── CustomerApi.ts

├── OrderApi.ts

└── ProductApi.ts

```

----------

# Authentication Lifecycle

```text
Login

↓

Receive Token

↓

Store Token

↓

Send Requests

↓

Refresh Token

↓

Logout

```

----------

# Authentication Types Comparison

Type

Common Usage

Basic

Legacy systems

Bearer

REST APIs

JWT

Microservices

OAuth 2.0

Enterprise SSO

API Key

Public/Partner APIs

Session Cookie

Traditional Web Apps

CSRF

Form protection

----------

# Enterprise Example – Azure AD

```text
User

↓

Azure AD

↓

Access Token

↓

Backend API

```

Playwright typically authenticates through the identity provider or uses a preconfigured test account to obtain a token before calling protected APIs.

----------

# Enterprise Example – Banking

```text
Login

↓

JWT

↓

Accounts API

↓

Transactions API

↓

Payments API

```

All requests share the same authenticated context.

----------

# Enterprise Example – E-commerce

```text
Login

↓

Session Cookie

↓

Cart

↓

Checkout

↓

Orders

```

No need to log in before every API call.

----------

# Best Practices

-   Never hardcode tokens or passwords in source code.
    
-   Retrieve tokens through authentication APIs or secure configuration.
    
-   Reuse authenticated request contexts where appropriate.
    
-   Implement centralized token refresh logic.
    
-   Store secrets securely using environment variables or secret managers.
    

----------

# Common Mistakes

### ❌ Hardcoding Tokens

Bad

```typescript
const token =
"eyJhbGc...";

```

Retrieve tokens dynamically.

----------

### ❌ Logging Sensitive Information

Avoid printing:

-   Tokens
    
-   Passwords
    
-   Client secrets
    
-   API keys
    

to test logs.

----------

### ❌ Creating New Tokens for Every Request

Authenticate once.

Reuse the request context whenever possible.

----------

### ❌ Ignoring Token Expiration

Long-running test suites may require token refresh handling.

----------

# Interview Questions

### Q1. What is the difference between Authentication and Authorization?

Authentication verifies identity ("Who are you?"), while Authorization determines permissions ("What are you allowed to do?").

----------

### Q2. How do you send a Bearer token in Playwright?

```typescript
const api = await request.newContext({

    extraHTTPHeaders: {

        Authorization:

            `Bearer ${token}`

    }

});

```

----------

### Q3. What is JWT?

JWT (JSON Web Token) is a compact, signed token format commonly used for stateless authentication. It typically contains a header, payload, and signature.

----------

### Q4. Why should authentication logic be centralized?

Centralizing authentication avoids duplicated login code, simplifies token refresh, and keeps tests focused on business scenarios.

----------

### Q5. Why is API-based login often preferred over UI login in automation?

API login is faster, more reliable, and allows UI tests to start in an authenticated state without repeatedly navigating through the login page.

----------

# Summary

Authentication is the gateway to almost every enterprise API. Playwright supports all common authentication mechanisms—including Basic Authentication, Bearer tokens, JWTs, OAuth 2.0, API keys, session cookies, and CSRF tokens—through its flexible `APIRequestContext`. By centralizing authentication, securely managing credentials, and reusing authenticated contexts, teams can build fast, maintainable, and enterprise-ready API automation frameworks.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNTc5MzY5OTldfQ==
-->
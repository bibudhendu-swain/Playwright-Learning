This is one of the biggest differences between an **intermediate Playwright framework** and an **enterprise-grade framework**.

One of the most common mistakes is putting API calls inside Page Objects.

```typescript
await loginPage.createUser();
await loginPage.deleteOrder();
await loginPage.resetDatabase();

```

This mixes **UI automation** with **backend operations**, making the framework difficult to maintain.

A mature framework introduces a **Service Layer** to isolate all backend interactions.

----------

# Part 21 – Framework Best Practices

# Chapter 5 – Service Layer Pattern

----------

# Introduction

Modern automation frameworks rarely interact with only the UI.

A complete automation suite may interact with:

-   REST APIs
    
-   GraphQL APIs
    
-   Databases
    
-   Authentication services
    
-   Message queues
    
-   Third-party services
    

The Service Layer encapsulates all these backend interactions.

Instead of calling APIs directly from tests or page objects, tests communicate with service classes.

----------

# Why Do We Need a Service Layer?

Without a Service Layer

```text
Test

↓

API Request

↓

Page Object

↓

Database

↓

Application

```

Responsibilities become mixed.

With a Service Layer

```text
Test

↓

Page

↓

Service

↓

Application

```

Each layer has a single responsibility.

----------

# Responsibilities

A Service Layer should:

-   Encapsulate API calls
    
-   Handle authentication
    
-   Create test data
    
-   Delete test data
    
-   Perform backend verification
    
-   Abstract service endpoints
    
-   Hide HTTP implementation
    

A Service Layer should **not** contain UI automation.

----------

# Enterprise Architecture

```text
Tests

├── UI Pages

├── API Services

├── Database Services

└── Message Services

```

Each layer is independent.

----------

# Folder Structure

```text
src/

├── services/

│     ├── AuthService.ts

│     ├── UserService.ts

│     ├── ProductService.ts

│     ├── OrderService.ts

│     ├── PaymentService.ts

│     ├── InventoryService.ts

│     └── NotificationService.ts

```

----------

# Base API Client

Instead of every service creating requests independently,

build one reusable API client.

```typescript
import { APIRequestContext } from '@playwright/test';

export class ApiClient {

    constructor(
        protected readonly request: APIRequestContext
    ) {}

}

```

Every service inherits or composes this client.

----------

# Authentication Service

Example

```typescript
export class AuthService {

    constructor(
        private readonly request: APIRequestContext
    ) {}

    async login(username: string, password: string) {

        return await this.request.post("/login", {

            data: {

                username,

                password

            }

        });

    }

}

```

Tests never call `/login` directly.

----------

# User Service

```typescript
export class UserService {

    async createUser(user: User){}

    async deleteUser(id: string){}

    async getUser(id: string){}

    async updateUser(user: User){}

}

```

Centralized user management.

----------

# Product Service

```typescript
export class ProductService {

    async createProduct(){}

    async deleteProduct(){}

    async updateProduct(){}

}

```

----------

# Order Service

```typescript
export class OrderService {

    async createOrder(){}

    async cancelOrder(){}

    async getOrder(){}

}

```

----------

# Payment Service

```typescript
export class PaymentService {

    async authorize(){}

    async capture(){}

    async refund(){}

}

```

Useful for payment testing.

----------

# Test Using Services

Instead of

```typescript
await request.post("/users");

```

Use

```typescript
await userService.createUser(user);

```

The test is easier to understand.

----------

# API Details Remain Hidden

Bad

```typescript
await request.post(

"/v3/api/customer/create"

);

```

Good

```typescript
await customerService.create(customer);

```

Implementation details stay inside the service.

----------

# Combining UI and API

Example

```text
UserService

↓

Create User

↓

Login Page

↓

Login

↓

Dashboard

```

UI verifies behavior,

API prepares test data.

----------

# Service Reuse

One service can be reused by:

-   UI Tests
    
-   API Tests
    
-   Integration Tests
    
-   Performance Tests
    

This avoids duplicated API logic.

----------

# Authentication Strategy

Many frameworks repeatedly log in through the UI.

A faster approach:

```text
Auth Service

↓

Token

↓

Storage State

↓

Playwright

```

This significantly reduces execution time.

----------

# API Setup Strategy

Instead of

```text
UI

↓

Create Customer

↓

5 Minutes

```

Use

```text
API

↓

Create Customer

↓

2 Seconds

```

The UI test focuses on what it actually needs to validate.

----------

# Service Composition

Services can collaborate.

Example

```text
Order Service

↓

Inventory Service

↓

Payment Service

```

Each service remains independent while supporting higher-level workflows.

----------

# Facade Service

Sometimes multiple services are combined behind a single interface.

Example

```typescript
CheckoutService

↓

UserService

↓

CartService

↓

PaymentService

```

Tests call:

```typescript
await checkoutService.prepareCheckout();

```

instead of coordinating several services themselves.

----------

# API Response Models

Avoid returning raw JSON.

Example

```typescript
const user = await userService.createUser();

```

Instead of

```typescript
const json = await response.json();

```

Return strongly typed models.

----------

# Error Handling

Handle API failures inside the service.

Example

```typescript
if (!response.ok()) {

    throw new Error("Unable to create user");

}

```

Tests receive meaningful exceptions.

----------

# Logging

Each service should log:

```text
POST /users

Status : 201

Duration : 120ms

```

Useful for debugging.

----------

# Retry Strategy

Retries should be limited.

Good candidates:

-   Temporary network failures
    
-   HTTP 429 (rate limiting)
    
-   HTTP 503 (service unavailable)
    

Avoid retrying:

-   HTTP 400
    
-   HTTP 401
    
-   HTTP 404
    

These usually indicate functional issues.

----------

# Database Service

Some projects require database verification.

```typescript
export class DatabaseService {

    async executeQuery(){}

    async cleanup(){}

}

```

Tests call the service,

not the database directly.

----------

# Message Queue Service

Enterprise applications may use queues.

```text
Application

↓

Kafka

↓

RabbitMQ

↓

Service Layer

```

The framework should isolate queue interactions.

----------

# Third-Party Services

Examples

-   Payment Gateway
    
-   Email Service
    
-   SMS Provider
    
-   Identity Provider
    

Each should have its own service class.

----------

# Dependency Injection

Services integrate well with Playwright fixtures.

Example

```typescript
test("Create User", async ({ userService }) => {

    await userService.createUser();

});

```

Tests remain clean and focused.

----------

# Enterprise Architecture

```text
Tests

↓

Pages

↓

Components

↓

Services

├── Auth

├── Users

├── Products

├── Orders

├── Payments

├── Inventory

└── Notifications

```

----------

# Recommended Folder Structure

```text
services/

├── ApiClient.ts

├── AuthService.ts

├── UserService.ts

├── ProductService.ts

├── OrderService.ts

├── InventoryService.ts

├── PaymentService.ts

├── EmailService.ts

└── DatabaseService.ts

```

----------

# Best Practices

-   Keep all HTTP logic inside services.
    
-   Never expose API endpoints directly to tests.
    
-   Return typed models instead of raw JSON whenever possible.
    
-   Separate services by business domain.
    
-   Combine UI and API strategically to speed up end-to-end scenarios.
    
-   Use services for test data creation and cleanup.
    
-   Integrate services through Playwright fixtures for better dependency management.
    

----------

# Common Mistakes

### ❌ API Calls Inside Page Objects

Bad

```typescript
loginPage.createUser();

```

Pages should represent the UI, not backend operations.

----------

### ❌ Direct HTTP Calls in Tests

Bad

```typescript
await request.post("/users");

```

Use a dedicated service instead.

----------

### ❌ One Huge Service

Avoid

```text
ApplicationService

```

containing every endpoint.

Prefer focused services like:

-   `UserService`
    
-   `OrderService`
    
-   `ProductService`
    

----------

### ❌ Returning Raw Responses Everywhere

Wrap responses in meaningful models or domain objects when appropriate.

----------

### ❌ Mixing Business Domains

Keep customer, product, payment, inventory, and notification logic in separate services.

----------

# Interview Questions

### Q1. Why introduce a Service Layer in a Playwright framework?

The Service Layer separates backend interactions from UI automation, improving maintainability, reuse, and test readability.

----------

### Q2. Should page objects perform API calls?

No. Page objects should focus on UI interactions. API calls belong in dedicated service classes.

----------

### Q3. What are the benefits of using APIs for test setup?

API-based setup is significantly faster, more reliable, and less dependent on the UI, reducing execution time for end-to-end tests.

----------

### Q4. Why use a Base API Client?

A shared API client centralizes authentication, headers, logging, retry logic, and request configuration, reducing duplication across services.

----------

### Q5. How should services be organized?

Services should be organized by business domain (for example, `UserService`, `OrderService`, `PaymentService`) rather than by technical implementation, making them easier to maintain and understand.

----------

# Summary

The Service Layer is a key architectural pattern in enterprise Playwright frameworks. By isolating backend interactions into dedicated services, teams can keep tests and page objects focused on UI behavior while reusing API logic across UI, API, and integration tests. Combined with typed models, centralized API clients, and dependency injection, a well-designed Service Layer significantly improves framework scalability, readability, and execution speed.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MTg5NTIyMzldfQ==
-->
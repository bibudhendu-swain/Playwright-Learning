This is one of the **most valuable enterprise chapters** because it addresses a common challenge: **how to manage test data efficiently**.

Many automation engineers create test data manually through the UI, which is slow and brittle. Mature automation frameworks use APIs to create, manage, and clean up data, making tests faster, independent, and suitable for parallel execution.

----------

# Part 19 – API Testing

# Chapter 7 – API Fixtures & Test Data Management

----------

# Introduction

Every automated test needs data.

For example:

```text
Login Test

↓

User

-------------------

Order Test

↓

Customer

↓

Product

↓

Order

-------------------

Payment Test

↓

Invoice

↓

Payment

```

There are two common approaches:

**UI-Based Test Data**

```text
Open Browser

↓

Create Customer

↓

Create Product

↓

Create Order

↓

Run Test

```

**API-Based Test Data**

```text
API

↓

Create Customer

↓

Create Product

↓

Create Order

↓

Run UI Test

```

The second approach is much faster and more reliable.

----------

# Why Use APIs for Test Data?

Using APIs provides several benefits:

-   Faster execution
    
-   Independent tests
    
-   Parallel execution
    
-   Cleaner setup
    
-   Easier cleanup
    
-   Less UI dependency
    

----------

# Test Data Lifecycle

A well-designed test manages its own data.

```text
Create Data

↓

Execute Test

↓

Validate

↓

Cleanup

```

Each test is responsible for the data it creates.

----------

# Using API Fixtures

Instead of creating data inside every test, create reusable fixtures.

Example

```typescript
import { test as base } from "@playwright/test";
import { CustomerApi } from "../api/CustomerApi";

export const test = base.extend<{
    customerApi: CustomerApi;
}>({
    customerApi: async ({ request }, use) => {
        const api = new CustomerApi(request);
        await use(api);
    }
});

```

Usage

```typescript
import { test, expect } from "../fixtures/apiFixture";

test("Create customer", async ({ customerApi }) => {

    const customer =
        await customerApi.createCustomer();

    expect(customer.id)
        .toBeDefined();
});

```

The fixture provides a ready-to-use API client.

----------

# Test Data Builders

Hardcoding request payloads is difficult to maintain.

Instead of

```typescript
{
    firstName: "John",
    lastName: "Doe",
    city: "London"
}

```

Create builders.

```typescript
class CustomerBuilder {

    build() {
        return {
            firstName: "John",
            lastName: "Doe",
            city: "London"
        };
    }

}

```

Usage

```typescript
const customer =
new CustomerBuilder().build();

```

----------

# Builder with Overrides

Most tests need slightly different data.

```typescript
class CustomerBuilder {

    build(overrides = {}) {

        return {

            firstName: "John",

            lastName: "Doe",

            city: "London",

            ...overrides

        };

    }

}

```

Usage

```typescript
const customer =
builder.build({

    city: "Paris"

});

```

Only the required fields change.

----------

# Dynamic Test Data

Hardcoded values often cause conflicts.

Bad

```text
john@test.com

```

Every test creates the same user.

Better

```typescript
const email =

`user_${Date.now()}@test.com`;

```

Or

```typescript
import { randomUUID } from "crypto";

const customer = {

    email:
    `${randomUUID()}@test.com`

};

```

Now every test creates unique data.

----------

# Factory Pattern

Instead of

```typescript
builder.build();

```

Create specialized factories.

```typescript
class CustomerFactory {

    static premium() {

        return {

            type: "Premium",

            city: "London"

        };

    }

    static guest() {

        return {

            type: "Guest"

        };

    }

}

```

Usage

```typescript
const customer =
CustomerFactory.premium();

```

----------

# Setup Using APIs

Example

```text
Create Customer

↓

Create Product

↓

Create Order

↓

Run UI Test

```

```typescript
test.beforeEach(async ({ customerApi }) => {

    await customerApi.createCustomer();

});

```

----------

# Cleanup Using APIs

```typescript
test.afterEach(async ({ customerApi }) => {

    await customerApi.deleteCustomer();

});

```

Now every test cleans up after itself.

----------

# Returning Created IDs

Always capture IDs for cleanup.

```typescript
const customer =

await customerApi.createCustomer();

const customerId =

customer.id;

```

Later

```typescript
await customerApi.deleteCustomer(
    customerId
);

```

----------

# Idempotent Test Data

Tests should produce the same result every time.

Good

```text
Create Customer

↓

Delete Customer

```

Bad

```text
Reuse Existing Customer

```

Avoid depending on shared or pre-existing data.

----------

# Parallel Execution

When tests run in parallel:

```text
Test 1

↓

Customer A

-----------------

Test 2

↓

Customer A

```

Conflict!

Instead

```text
Test 1

↓

Customer A123

-----------------

Test 2

↓

Customer B456

```

Unique test data prevents collisions.

----------

# Seeding Through APIs

Large test suites often seed common data.

```text
Products

↓

Categories

↓

Countries

↓

Tax Rules

```

Setup

```typescript
test.beforeAll(async ({ productApi }) => {

    await productApi.seedProducts();

});

```

----------

# Database vs API Seeding

API

Database

Uses application rules

Direct DB manipulation

Stable

May bypass validations

Slower

Faster

Preferred for functional tests

Often used for environment setup

Whenever possible, use APIs instead of direct database updates.

----------

# Enterprise Example

Order Test

```text
Create Customer

↓

Create Product

↓

Create Order

↓

Open Browser

↓

Verify Order

↓

Delete Order

↓

Delete Customer

```

Completely independent.

----------

# Reusable Test Data Service

```typescript
class TestDataService {

    constructor(

        private customerApi: CustomerApi,

        private productApi: ProductApi

    ) {}

    async createOrderData() {

        const customer =
            await this.customerApi.createCustomer();

        const product =
            await this.productApi.createProduct();

        return {

            customer,

            product

        };

    }

}

```

Usage

```typescript
const data =
await service.createOrderData();

```

----------

# API Fixture Architecture

```text
Tests

↓

Fixtures

↓

TestDataService

↓

API Clients

↓

Backend

```

Tests remain clean and focused.

----------

# Suggested Folder Structure

```text
api/

├── BaseApi.ts

├── CustomerApi.ts

├── ProductApi.ts

├── OrderApi.ts

fixtures/

├── apiFixture.ts

builders/

├── CustomerBuilder.ts

├── ProductBuilder.ts

factories/

├── CustomerFactory.ts

services/

├── TestDataService.ts

```

----------

# Enterprise Example – E-commerce

Instead of

```text
Open Browser

↓

Login

↓

Create Product

↓

Create Customer

↓

Create Cart

```

Use

```text
API

↓

Create Product

↓

Create Customer

↓

Create Cart

↓

Open Browser

↓

Checkout

```

Execution becomes much faster.

----------

# Test Data Strategy

```text
Unique Data

↓

Reusable Builders

↓

Factories

↓

API Fixtures

↓

Cleanup

↓

Parallel Safe

```

----------

# Best Practices

-   Create test data through APIs instead of the UI whenever possible.
    
-   Generate unique data to avoid collisions during parallel execution.
    
-   Use builders for flexible payload creation.
    
-   Use factories for common business scenarios.
    
-   Always clean up data created during tests.
    
-   Keep test data creation separate from test logic.
    

----------

# Common Mistakes

### ❌ Hardcoding Emails

Bad

```text
john@test.com

```

Good

```text
user_<timestamp>@test.com

```

or use a UUID.

----------

### ❌ Sharing Test Data

Multiple tests using the same customer can cause flaky failures.

Create isolated data for each test unless shared data is intentionally read-only.

----------

### ❌ Forgetting Cleanup

Leaving behind created records pollutes test environments and may affect later tests.

----------

### ❌ Creating Test Data Through the UI

UI creation is slower and more fragile.

Prefer API-based setup for speed and reliability.

----------

### ❌ Mixing Builders with Assertions

Builders should only create data.

Validation belongs in tests or validator classes.

----------

# Interview Questions

### Q1. Why should APIs be used for test data setup?

Because they are faster, more reliable, and independent of UI changes.

----------

### Q2. What is the Builder Pattern?

A design pattern used to create flexible and reusable test data objects, often allowing default values with optional overrides.

----------

### Q3. Why is unique test data important?

It prevents collisions when tests run in parallel and avoids dependencies on previously created records.

----------

### Q4. What is the difference between a Builder and a Factory?

-   A **Builder** creates customizable objects with defaults and overrides.
    
-   A **Factory** creates predefined business scenarios, such as a premium customer or a guest customer.
    

----------

### Q5. Why should cleanup be part of automated tests?

To keep environments clean, reduce test interference, and ensure repeatable execution.

----------

# Summary

Effective test data management is the foundation of stable API and UI automation. By using API fixtures, builders, factories, and dedicated test data services, teams can create isolated, repeatable, and parallel-safe tests. Combining API-based setup with automatic cleanup dramatically improves execution speed, reduces flakiness, and makes enterprise automation frameworks easier to maintain.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTk0ODg1OThdfQ==
-->
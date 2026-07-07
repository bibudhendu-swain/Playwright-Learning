This is one of the **most underestimated topics** in automation.

In my experience, more automation failures are caused by **poor test data management** than by Playwright itself.

Typical problems include:

-   Tests failing because another test deleted the data
    
-   Parallel execution causing duplicate records
    
-   Hardcoded usernames already existing
    
-   Environment pollution
    
-   Cleanup failures
    
-   Shared accounts causing flaky tests
    

A well-designed **Test Data Management (TDM)** strategy eliminates most of these issues.

----------

# Part 21 – Framework Best Practices

# Chapter 7 – Test Data Management

----------

# Introduction

Every automated test requires data.

Examples:

-   Users
    
-   Products
    
-   Orders
    
-   Payments
    
-   Customers
    
-   Addresses
    
-   Roles
    
-   Permissions
    

The way this data is created, managed, and cleaned up has a major impact on the reliability and scalability of an automation framework.

A mature framework treats **test data as a first-class architectural concern**, not as an afterthought.

----------

# Why Test Data Matters

Imagine a login test.

Without proper data management:

```text
Login Test

↓

Username already deleted

↓

Test Fails

```

Or

```text
Create Customer

↓

Customer already exists

↓

Test Fails

```

The application is working correctly.

The automation failed because of bad data management.

----------

# Characteristics of Good Test Data

Good test data should be:

-   Predictable
    
-   Independent
    
-   Repeatable
    
-   Isolated
    
-   Easy to create
    
-   Easy to clean
    
-   Parallel-safe
    
-   Environment-independent
    

----------

# Sources of Test Data

Enterprise frameworks usually obtain data from multiple sources.

```text
JSON

↓

Builder

↓

Factory

↓

API

↓

Database

↓

Generated

↓

Environment

```

Each source serves different needs.

----------

# Static Test Data

Example

```text
test-data/

├── users.json

├── products.json

├── addresses.json

```

Example

```json
{
  "username": "admin",
  "password": "Password123"
}

```

Advantages

-   Simple
    
-   Easy to maintain
    
-   Version controlled
    

Disadvantages

-   Can become stale
    
-   Limited flexibility
    

----------

# Dynamic Test Data

Generated during execution.

```text
Execution

↓

Generate User

↓

Run Test

↓

Delete User

```

Useful for:

-   Parallel execution
    
-   Independent tests
    
-   Unique data requirements
    

----------

# Hardcoded Data

Avoid

```typescript
const email = "john@test.com";

```

This quickly becomes a maintenance problem.

Prefer generated or configurable values.

----------

# Builder Pattern

Instead of

```typescript
const user = {

    firstName: "John",

    lastName: "Smith",

    email: "john@test.com",

    role: "Admin"

};

```

Use

```typescript
const user = new UserBuilder()

    .withRole("Admin")

    .build();

```

Builders make test data expressive and reusable.

----------

# Example Builder

```typescript
class UserBuilder {

    private user: User = {

        firstName: "John",

        lastName: "Doe",

        email: Random.email(),

        role: "User"

    };

    withRole(role: string){

        this.user.role = role;

        return this;

    }

    build(){

        return this.user;

    }

}

```

----------

# Factory Pattern

Factories create complete objects.

```typescript
UserFactory.admin();

UserFactory.customer();

UserFactory.manager();

```

Useful for commonly used data sets.

----------

# Builder vs Factory

Builder

Factory

Customizable

Predefined

Flexible

Consistent

Best for complex objects

Best for common objects

Many enterprise frameworks use both.

----------

# Faker Integration

Generate realistic data.

Example

```typescript
import { faker } from '@faker-js/faker';

const email = faker.internet.email();

const phone = faker.phone.number();

const company = faker.company.name();

```

Advantages

-   Unique values
    
-   Realistic test scenarios
    
-   Parallel-safe data
    

----------

# Random Data

Example

```typescript
const username =

`user-${Date.now()}`;

```

Useful for uniqueness,

but avoid making every field random if deterministic values are sufficient.

----------

# API-Based Data Creation

Instead of

```text
UI

↓

Create Customer

↓

45 Seconds

```

Use

```text
API

↓

Create Customer

↓

2 Seconds

```

This is faster and reduces UI dependency.

----------

# Database Seeding

Some applications require database setup.

```text
Database

↓

Insert Data

↓

Run Tests

```

This is often used in integration or lower-environment testing where direct database access is permitted.

----------

# Storage State

Instead of repeatedly creating users,

reuse authentication.

```text
Login Once

↓

Storage State

↓

Multiple Tests

```

This reduces execution time while keeping authentication reliable.

----------

# Environment-Specific Data

Example

```text
QA

↓

Test Customer

----------------

UAT

↓

Business Customer

----------------

Staging

↓

Production-like Customer

```

Avoid assuming that every environment contains identical data.

----------

# Test Data Cleanup

Always clean temporary data.

Workflow

```text
Create User

↓

Run Test

↓

Delete User

```

Cleanup strategies include:

-   API deletion
    
-   Database cleanup
    
-   Scheduled cleanup jobs
    

----------

# Cleanup Timing

Three common approaches.

### Before Test

```text
Delete Old Data

↓

Run Test

```

----------

### After Test

```text
Run Test

↓

Delete Data

```

----------

### Scheduled Cleanup

```text
Nightly Job

↓

Remove Test Data

```

The best choice depends on the application's constraints.

----------

# Idempotent Tests

An idempotent test can be executed repeatedly with the same outcome.

Good

```text
Create User

↓

Delete User

```

Bad

```text
Create User

↓

Never Delete

```

----------

# Parallel-Safe Data

Bad

```text
Worker 1

↓

admin@test.com

----------------

Worker 2

↓

admin@test.com

```

Good

```text
Worker 1

↓

admin-001@test.com

----------------

Worker 2

↓

admin-002@test.com

```

Each worker uses unique data.

----------

# Shared Data

Avoid sharing mutable data.

Example

```text
All Tests

↓

Same Customer

```

One test may modify or delete it.

Instead

```text
Each Test

↓

Own Customer

```

----------

# Immutable Reference Data

Some data can safely be shared.

Examples

-   Country list
    
-   Currency list
    
-   Product categories
    
-   Static configuration
    

These rarely change during tests.

----------

# Test Data Repository

A centralized repository improves reuse.

```text
test-data/

├── builders/

├── factories/

├── json/

├── templates/

└── generators/

```

----------

# Data Templates

Example

```json
{
  "firstName": "",
  "lastName": "",
  "email": "",
  "role": "Customer"
}

```

Builders populate only required fields.

----------

# Enterprise Strategy

```text
Factories

↓

Builders

↓

Faker

↓

API

↓

Application

```

This combines consistency with flexibility.

----------

# Example Workflow

```text
UserBuilder

↓

UserService

↓

Create User

↓

Login

↓

Run Test

↓

Delete User

```

The UI test never worries about how the user was created.

----------

# Test Data Ownership

Keep ownership clear.

Layer

Responsibility

Builder

Construct objects

Factory

Provide predefined objects

Service

Create/delete through APIs

Test

Use data to verify behavior

----------

# Enterprise Folder Structure

```text
test-data/

├── builders/

│     ├── UserBuilder.ts

│     ├── ProductBuilder.ts

│     └── OrderBuilder.ts

├── factories/

│     ├── UserFactory.ts

│     ├── ProductFactory.ts

│     └── OrderFactory.ts

├── json/

├── templates/

└── generators/

```

----------

# Best Practices

-   Prefer API-based setup over UI-based setup whenever possible.
    
-   Generate unique data for tests running in parallel.
    
-   Use builders for flexible object creation and factories for common scenarios.
    
-   Separate test data from test logic.
    
-   Clean up temporary data after execution.
    
-   Keep immutable reference data separate from mutable test data.
    
-   Make tests idempotent so they can be rerun safely.
    

----------

# Common Mistakes

### ❌ Hardcoding Test Accounts

Bad

```text
admin@test.com

```

used by every test.

----------

### ❌ UI-Based Data Creation

Using the UI to create all test data slows execution and increases failure points.

----------

### ❌ Never Cleaning Up

Test environments become polluted with stale data.

----------

### ❌ Random Everything

Excessive randomness makes failures difficult to reproduce.

Randomize only where uniqueness is required.

----------

### ❌ Sharing Mutable Data

Shared customers, orders, or users frequently lead to flaky parallel execution.

----------

# Interview Questions

### Q1. Why is API-based test data creation preferred over UI-based creation?

API-based creation is faster, more reliable, and avoids unnecessary UI dependencies, reducing execution time and flakiness.

----------

### Q2. What is the difference between a Builder and a Factory?

-   A **Builder** creates customizable objects step by step.
    
-   A **Factory** returns predefined object configurations for common scenarios.
    

----------

### Q3. Why should tests use unique data during parallel execution?

Unique data prevents conflicts, race conditions, and unintended interactions between concurrently running tests.

----------

### Q4. What makes a test idempotent?

An idempotent test can be executed repeatedly without leaving behind state that changes the outcome of future executions.

----------

### Q5. When is static test data appropriate?

Static data is useful for stable reference information such as countries, currencies, or predefined roles that rarely change.

----------

# Summary

Effective test data management is essential for building reliable and scalable Playwright frameworks. By combining builders, factories, dynamic data generation, API-based setup, and disciplined cleanup strategies, teams can eliminate many common sources of flaky tests and support safe parallel execution. A well-designed test data strategy allows tests to remain independent, repeatable, and maintainable as automation suites continue to grow.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NjYwMzk5MTFdfQ==
-->
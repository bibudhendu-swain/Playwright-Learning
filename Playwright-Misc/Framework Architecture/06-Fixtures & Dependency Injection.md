This chapter is arguably **Playwright's biggest architectural advantage over Selenium**.

If I had to name **one feature** that makes Playwright frameworks cleaner than most Selenium frameworks, it would be **Fixtures**.

Many Selenium frameworks depend heavily on:

-   BaseTest
    
-   Singleton Drivers
    
-   ThreadLocal
    
-   Factory Classes
    
-   Dependency Injection frameworks like Guice or Spring
    

Playwright replaces much of that complexity with a simple yet powerful fixture system.

This chapter is intentionally much deeper than the Playwright documentation because enterprise teams use fixtures extensively.

----------

# Part 21 – Framework Best Practices

# Chapter 6 – Fixtures & Dependency Injection (Enterprise Edition)

----------

# Introduction

A fixture is an object or resource that is created before a test executes and automatically cleaned up afterward.

Examples include:

-   Browser
    
-   Browser Context
    
-   Page
    
-   Logged-in User
    
-   API Client
    
-   Database Connection
    
-   Test Data
    
-   Service Classes
    

Instead of every test creating these resources manually, Playwright injects them automatically.

----------

# What is Dependency Injection?

Dependency Injection (DI) is a design principle where an object receives its dependencies from an external source instead of creating them itself.

Without DI

```text
Test

↓

new LoginPage()

↓

new UserService()

↓

new DatabaseService()

```

The test is responsible for object creation.

With DI

```text
Playwright

↓

Fixture

↓

Inject Objects

↓

Test

```

The framework manages object creation and lifecycle.

----------

# Why Fixtures?

Without fixtures

```typescript
test("Login", async ({ page }) => {

    const loginPage = new LoginPage(page);

    const dashboard = new DashboardPage(page);

    const userService = new UserService(request);

});

```

Every test repeats the same setup.

With fixtures

```typescript
test("Login", async ({

    loginPage,

    dashboardPage,

    userService

}) => {

});

```

Cleaner, shorter, and more readable.

----------

# Built-in Fixtures

Playwright provides several fixtures out of the box.

Fixture

Purpose

browser

Browser instance

context

Browser context

page

Browser page

request

API request context

browserName

Current browser

baseURL

Configured base URL

These cover most common automation needs.

----------

# Fixture Lifecycle

```text
Worker Starts

↓

Worker Fixture Created

↓

Test Starts

↓

Test Fixture Created

↓

Execute Test

↓

Test Fixture Destroyed

↓

Next Test

↓

Worker Ends

↓

Worker Fixture Destroyed

```

Understanding this lifecycle is essential for choosing the correct fixture scope.

----------

# Fixture Types

Playwright supports two primary scopes:

```text
Worker Fixture

↓

Shared Across Tests

---------------------

Test Fixture

↓

Created Per Test

```

----------

# Test Fixtures

A new instance is created for every test.

```text
Test 1

↓

New Page

----------------

Test 2

↓

New Page

```

Advantages:

-   Complete isolation
    
-   No shared state
    
-   Reduced flakiness
    

----------

# Worker Fixtures

Created once per worker.

```text
Worker

↓

Database Connection

↓

Authentication Token

↓

Configuration

↓

Multiple Tests

```

Useful for expensive resources.

----------

# Creating a Custom Fixture

```typescript
import { test as base } from '@playwright/test';

export const test = base.extend({

    loginPage: async ({ page }, use) => {

        await use(new LoginPage(page));

    }

});

```

The fixture creates the page object and injects it into tests.

----------

# Using the Fixture

```typescript
import { test } from '../fixtures/base.fixture';

test('Login', async ({ loginPage }) => {

    await loginPage.open();

});

```

No manual object creation is required.

----------

# Multiple Fixtures

```typescript
export const test = base.extend({

    loginPage: async (...) => {},

    dashboardPage: async (...) => {},

    userService: async (...) => {}

});

```

Playwright resolves dependencies automatically.

----------

# Fixture Composition

Fixtures can depend on other fixtures.

```text
page

↓

LoginPage

↓

DashboardPage

↓

OrderPage

```

Each fixture builds upon lower-level fixtures.

----------

# Service Fixtures

Instead of

```typescript
const userService = new UserService(request);

```

Use

```typescript
userService

```

The service is automatically injected.

----------

# Authentication Fixture

Example

```typescript
authenticatedPage

```

Workflow

```text
Browser

↓

Login

↓

Storage State

↓

Authenticated Page

↓

Tests

```

Every test starts in a logged-in state.

----------

# Database Fixture

```typescript
database

```

Workflow

```text
Connect

↓

Execute Tests

↓

Disconnect

```

This is often a worker-scoped fixture.

----------

# API Fixture

```typescript
apiClient

```

Shared across multiple tests.

----------

# Configuration Fixture

Instead of

```typescript
Config.getEnvironment()

```

Use

```typescript
environment

```

Injected directly into the test.

----------

# Page Fixtures

```typescript
test(

"Checkout",

async ({

loginPage,

checkoutPage,

header

}) => {

});

```

Tests become highly readable.

----------

# Worker Fixture Example

```typescript
apiToken

```

Lifecycle

```text
Worker Starts

↓

Generate Token

↓

Run 50 Tests

↓

Destroy Token

```

Much faster than logging in for every test.

----------

# Test Fixture Example

```text
Test Starts

↓

Create Customer

↓

Execute Test

↓

Delete Customer

```

Every test gets isolated data.

----------

# Fixture Dependency Graph

```text
Browser

↓

Context

↓

Page

↓

LoginPage

↓

DashboardPage

↓

CheckoutPage

```

Playwright resolves this automatically.

----------

# Lazy Initialization

Fixtures are lazy.

If a fixture isn't requested,

it isn't created.

Example

```typescript
test(

"API",

async ({

request

}) => {

});

```

The `page` fixture is never created.

This improves execution performance.

----------

# Fixture Cleanup

Always use

```typescript
await use(resource);

```

Example

```typescript
database

↓

connect()

↓

use()

↓

disconnect()

```

Cleanup runs automatically after `use()` completes.

----------

# Enterprise Fixture Structure

```text
fixtures/

├── base.fixture.ts

├── auth.fixture.ts

├── page.fixture.ts

├── api.fixture.ts

├── database.fixture.ts

├── services.fixture.ts

└── admin.fixture.ts

```

Each fixture file has a clear responsibility.

----------

# Dependency Injection Architecture

```text
Playwright

↓

Fixtures

↓

Pages

↓

Components

↓

Services

↓

Tests

```

Tests only request what they need.

----------

# Base Fixture

Many teams create one central export.

```typescript
export const test =
base.extend({

});

```

All custom fixtures are registered there.

----------

# Avoid Global Objects

Bad

```typescript
const loginPage = new LoginPage(page);

```

shared globally.

Prefer fixture injection.

----------

# Fixture vs Hooks

Hooks

```text
beforeEach()

↓

Setup

```

Fixtures

```text
Dependency

↓

Injection

```

Hooks are useful for actions.

Fixtures are useful for resources.

----------

# Fixture Naming

Good

```text
loginPage

userService

authenticatedPage

database

```

Avoid

```text
helper

common

util

```

Use names that clearly describe the injected resource.

----------

# Enterprise Example

```typescript
test(

"Checkout",

async ({

loginPage,

header,

checkoutPage,

userService,

orderService,

environment

}) => {

});

```

The test reads almost like English.

----------

# Enterprise Architecture

```text
Playwright

↓

Fixtures

├── Pages

├── Components

├── Services

├── Config

└── Database

↓

Tests

```

Everything is injected only when required.

----------

# Best Practices

-   Use fixtures for resources, not business workflows.
    
-   Prefer test-scoped fixtures unless sharing is safe and beneficial.
    
-   Use worker-scoped fixtures for expensive initialization such as authentication tokens or database connections.
    
-   Keep fixtures small and focused on a single responsibility.
    
-   Compose fixtures instead of creating large monolithic fixture files.
    
-   Take advantage of lazy initialization by injecting only the fixtures a test actually needs.
    
-   Keep cleanup logic close to resource creation using `await use(...)`.
    

----------

# Common Mistakes

### ❌ Creating Everything in `beforeEach`

Avoid initializing every page, service, and helper for every test.

Fixtures are created only when requested.

----------

### ❌ One Giant Fixture File

Bad

```text
base.fixture.ts

↓

1500 Lines

```

Split fixtures by responsibility.

----------

### ❌ Using Worker Fixtures for Mutable Test Data

Worker fixtures share state.

Don't use them for resources that individual tests modify.

----------

### ❌ Business Logic Inside Fixtures

Bad

```text
Fixture

↓

Login

↓

Create Customer

↓

Place Order

↓

Approve Payment

```

Fixtures should provide resources, not entire business scenarios.

----------

### ❌ Ignoring Cleanup

Every fixture allocating external resources should clean them up automatically.

----------

# Interview Questions

### Q1. What is the difference between a test fixture and a worker fixture?

-   A **test fixture** is created for each test and provides complete isolation.
    
-   A **worker fixture** is created once per worker and is shared across tests running in that worker.
    

----------

### Q2. Why are fixtures better than creating objects directly in tests?

Fixtures reduce duplication, improve readability, centralize setup and cleanup, and leverage Playwright's dependency injection mechanism.

----------

### Q3. What is lazy fixture initialization?

A fixture is created only if a test explicitly requests it. Unused fixtures are never initialized, improving execution performance.

----------

### Q4. When should a worker-scoped fixture be used?

When the resource is expensive to create, safe to share, and does not introduce unwanted shared mutable state—for example, authentication tokens or API clients.

----------

### Q5. Should business workflows be implemented inside fixtures?

No. Fixtures should provide reusable resources. Business workflows belong in tests or dedicated orchestration layers.

----------

# Summary

Playwright's fixture system provides a clean and powerful dependency injection mechanism that simplifies framework design. By distinguishing between test-scoped and worker-scoped resources, composing focused fixtures, and leveraging lazy initialization, teams can build automation frameworks that are easier to read, faster to execute, and significantly more maintainable than traditional setup approaches. Proper use of fixtures is one of the defining characteristics of an enterprise-grade Playwright framework.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNDM2OTQyODldfQ==
-->
Excellent. This chapter is where Playwright frameworks become **truly scalable**.

A small project can survive with a single `base.fixture.ts`.

An enterprise project with:

-   300+ Page Objects
    
-   50+ APIs
    
-   Multiple applications
    
-   Multiple teams
    

cannot.

The solution is **modular fixtures**.

This chapter explains how Microsoft recommends organizing fixtures in large projects.

----------

# 📘 Playwright Fixtures Handbook

# Part 12 – Combine Custom Fixtures from Multiple Modules (Complete Guide)

> **One fixture file is good.**
> 
> **Many focused fixture modules are much better.**

----------

# Why Split Fixtures?

Imagine a project with:

```text
loginPage

dashboardPage

checkoutPage

customerApi

orderApi

database

logger

faker

redis

kafka

mail

pdfReader

...

```

Putting everything in:

```text
base.fixture.ts

```

becomes:

```text
2500 Lines

```

Difficult to:

-   Navigate
    
-   Maintain
    
-   Review
    
-   Reuse
    

----------

# Better Architecture

Instead of

```text
base.fixture.ts

```

Split by responsibility.

```text
fixtures/

page.fixture.ts

api.fixture.ts

database.fixture.ts

logger.fixture.ts

auth.fixture.ts

common.fixture.ts

```

Each file owns one concern.

----------

# Enterprise Folder Structure

```text
fixtures/

├── page/
│      page.fixture.ts
│
├── api/
│      customer.fixture.ts
│      order.fixture.ts
│
├── auth/
│      auth.fixture.ts
│
├── database/
│      database.fixture.ts
│
├── logger/
│      logger.fixture.ts
│
└── base.fixture.ts

```

Very common.

----------

# Example – Page Fixture Module

```ts
// fixtures/page.fixture.ts

export const pageTest = base.extend({

    loginPage: async ({ page }, use) => {

        await use(new LoginPage(page));

    }

});

```

This module knows only about page objects.

----------

# API Fixture Module

```ts
// fixtures/api.fixture.ts

export const apiTest = base.extend({

    customerApi: async ({ request }, use) => {

        await use(new CustomerApi(request));

    }

});

```

Completely independent.

----------

# Database Module

```ts
database: [

async({}, use)=>{

const db=

await connect();

await use(db);

await db.close();

},

{

scope:'worker'

}

]

```

Infrastructure stays separate.

----------

# Logger Module

```ts
logger:[

async({}, use)=>{

await use(new Logger());

}

]

```

Simple.

Focused.

----------

# Problem

Now you have:

```text
pageTest

apiTest

databaseTest

loggerTest

```

How do you combine them?

----------

# Solution – `mergeTests()`

Playwright provides:

```ts
mergeTests()

```

This combines multiple extended `test` objects into one.

----------

# Example

```ts
import { mergeTests } from '@playwright/test';

import { pageTest } from './page.fixture';

import { apiTest } from './api.fixture';

export const test = mergeTests(

pageTest,

apiTest

);

```

Now one `test` contains both fixture sets.

----------

# Architecture

```text
page.fixture

↓

mergeTests

↑

api.fixture

↓

database.fixture

↓

logger.fixture

↓

Final test

```

Everything merged together.

----------

# Final Usage

```ts
import {

test,

expect

}

from '../fixtures/base.fixture';

```

Now every fixture is available.

----------

# Test Example

```ts
test('Order', async ({

loginPage,

customerApi,

database,

logger

})=>{

});

```

Those fixtures originated from four different files.

The test doesn't care.

----------

# Enterprise Example

Imagine four teams.

```text
UI Team

↓

Page Fixtures



API Team

↓

API Fixtures



Infrastructure Team

↓

Database Fixtures



Automation Team

↓

Merged Test

```

Independent development.

----------

# Module Independence

Page fixtures should not know about:

```text
Database

Kafka

Redis

```

Keep responsibilities separated.

----------

# Layered Architecture

```text
page.fixture

↓

api.fixture

↓

db.fixture

↓

logger.fixture

↓

base.fixture

```

Only:

```text
base.fixture

```

imports everything.

----------

# Real Enterprise Layout

```text
fixtures/

base.fixture.ts

page.fixture.ts

api.fixture.ts

db.fixture.ts

auth.fixture.ts

mail.fixture.ts

redis.fixture.ts

report.fixture.ts

logger.fixture.ts

```

Very maintainable.

----------

# Dependency Example

Suppose:

```text
loginPage

↓

page

```

Customer API

↓

request

Different dependency trees.

Perfectly fine.

----------

# Avoid Huge Files

Bad

```text
4000-Line

base.fixture.ts

```

Good

```text
300-Line

Modules

```

Each easy to review.

----------

# Circular Dependency Problem

Bad

```text
page.fixture

↓

imports

↓

api.fixture



api.fixture

↓

imports

↓

page.fixture

```

Circular dependency.

Avoid.

----------

# Better

```text
page.fixture

↓

base.fixture

↑

api.fixture

```

No cycles.

----------

# Shared Utilities

Need helper?

Create

```text
utils/

fixture-helper.ts

```

Both modules can use it.

----------

# Authentication Module

```text
auth.fixture

↓

loginPage

↓

storageState

↓

user

```

No API fixtures mixed in.

----------

# Worker Fixtures Module

Infrastructure

```text
database

logger

redis

config

```

Perfect candidates for a dedicated module.

----------

# Example

```text
worker.fixture.ts

```

Contains

```text
database

logger

config

```

Only.

----------

# Merge Multiple Modules

```ts
export const test = mergeTests(

pageTest,

apiTest,

databaseTest,

loggerTest,

authTest

);

```

One unified `test`.

----------

# Enterprise Diagram

```text
Page Fixtures

      │

API Fixtures

      │

Database Fixtures

      │

Logger Fixtures

      │

Auth Fixtures

      │

───────────────

mergeTests()

───────────────

      │

Final Test

```

Beautiful architecture.

----------

# Common Mistakes

## ❌ One Giant Fixture File

Avoid

```text
5000 Lines

```

Split logically.

----------

## ❌ Mixing Responsibilities

Bad

```text
Page Object

↓

Database

↓

Logger

```

One module.

----------

## ❌ Circular Imports

Never let modules import each other.

----------

## ❌ Duplicate Fixtures

Don't define:

```text
loginPage

```

twice.

One owner.

----------

## ❌ Team Ownership Confusion

Every fixture should have a clear owner.

Example:

```text
API Team

↓

API Fixtures

```

----------

# Recommended Module Ownership

Module

Owns

`page.fixture.ts`

Page Objects

`api.fixture.ts`

API clients

`database.fixture.ts`

Database connections

`auth.fixture.ts`

Authentication helpers

`logger.fixture.ts`

Logging

`worker.fixture.ts`

Worker-scoped infrastructure

`base.fixture.ts`

Merge point only

----------

# Interview Questions

### Q1. Why split fixtures into multiple modules?

To improve maintainability, readability, team ownership, and scalability in large projects.

----------

### Q2. What does `mergeTests()` do?

It combines multiple extended `test` objects into a single `test` that exposes all fixtures.

----------

### Q3. Should page fixtures and database fixtures live in the same file?

Generally no.

Keep responsibilities separated.

----------

### Q4. Can multiple teams own different fixture modules?

Yes.

That's one of the biggest advantages of modular fixtures.

----------

### Q5. Where should `mergeTests()` usually be called?

In a central file such as:

```text
fixtures/base.fixture.ts

```

which becomes the single import point for tests.

----------

### Q6. Can fixture modules depend on one another?

They can, but avoid tight coupling and especially circular dependencies. Prefer depending on shared utilities or built-in fixtures instead.

----------

# Best Practices

-   Organize fixtures by responsibility, not by object type count.
    
-   Keep one fixture module focused on one domain.
    
-   Use `mergeTests()` to compose a unified `test`.
    
-   Let `base.fixture.ts` act as the composition root.
    
-   Avoid circular imports between fixture modules.
    
-   Keep worker fixtures separate from test-scoped page fixtures.
    
-   Give each fixture a single, clear owner.
    

----------

# ⭐ Enterprise Architecture

```text
                         Tests
                           │
                           ▼
                 fixtures/base.fixture.ts
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
 page.fixture.ts     api.fixture.ts    auth.fixture.ts
        │                  │                  │
        ▼                  ▼                  ▼
   LoginPage         CustomerApi      AuthHelper
   CartPage          OrderApi         UserFactory

        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
database.fixture.ts  logger.fixture.ts worker.fixture.ts
        │                  │                  │
        ▼                  ▼                  ▼
 DatabasePool         Logger          Config / Redis / Kafka

                           │
                           ▼
                     mergeTests()
                           │
                           ▼
                  Unified Playwright Test

```

This architecture scales naturally as teams and applications grow. Each module remains independently maintainable while tests continue to import a single `test` object.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3NDA2NzI0NV19
-->
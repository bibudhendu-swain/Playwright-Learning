Excellent. This is one of the **most important advanced topics** in Playwright.

Many developers know **how** to create a worker fixture but don't understand **when** they should use one. Misusing worker-scoped fixtures can lead to flaky tests or poor performance.

By the end of this chapter, you'll know exactly when to choose **test scope** versus **worker scope**.

----------

# 📘 Playwright Fixtures Handbook

# Part 7 – Worker-Scoped Fixtures (Complete Guide)

> **Rule of thumb:**
> 
> -   **Test-scoped fixtures** → Isolation
>     
> -   **Worker-scoped fixtures** → Performance
>     

----------

# What is a Worker?

Before understanding worker fixtures, you must understand **workers**.

Suppose you run:

```bash
npx playwright test

```

Playwright doesn't execute every test one by one.

It creates one or more **worker processes**.

Example:

```text
Worker 1
│
├── login.spec.ts
├── cart.spec.ts
└── checkout.spec.ts

Worker 2
│
├── search.spec.ts
├── payment.spec.ts
└── profile.spec.ts

```

Each worker is a completely separate Node.js process.

----------

# Worker Lifecycle

```text
Worker Starts

↓

Create Worker Fixtures

↓

Run Test 1

↓

Run Test 2

↓

Run Test 3

↓

Destroy Worker Fixtures

↓

Worker Ends

```

Notice:

The fixture is created **once**.

----------

# Test Fixture vs Worker Fixture

## Test Fixture

```text
Test 1

↓

Create

↓

Run

↓

Destroy



Test 2

↓

Create

↓

Run

↓

Destroy

```

Every test gets a new instance.

----------

## Worker Fixture

```text
Worker

↓

Create

↓

Test 1

↓

Test 2

↓

Test 3

↓

Destroy

```

One instance.

Many tests.

----------

# Default Scope

When you write:

```ts
loginPage: async ({ page }, use) => {

}

```

Scope is:

```text
test

```

Even if you don't specify it.

Equivalent to:

```ts
loginPage: [

  async ({ page }, use) => {

  },

  {

    scope: 'test'

  }

]

```

----------

# Creating a Worker Fixture

Syntax

```ts
logger: [

async ({}, use) => {

    const logger = new Logger();

    await use(logger);

},

{

scope:'worker'

}

]

```

Notice the array syntax.

----------

# Why Array Syntax?

Because worker fixtures need options.

Structure

```ts
fixtureName:[

fixture,

options

]

```

Example

```ts
database:[

async(...){

},

{

scope:'worker'

}

]

```

----------

# Execution Timeline

```text
Worker Starts

↓

Create Database

↓

Test A

↓

Test B

↓

Test C

↓

Close Database

↓

Worker Ends

```

Database opens once.

----------

# Why Use Worker Fixtures?

Suppose connecting to Oracle takes:

```text
5 Seconds

```

100 tests.

Test fixture

```text
100 × 5

=

500 Seconds

```

Worker fixture

```text
5 Seconds

```

Huge improvement.

----------

# Example – Logger

Instead of

```text
Every Test

↓

Create Logger

```

Worker fixture

```text
Worker

↓

Logger

↓

100 Tests

```

----------

# Example – Database

```ts
database:[

async({}, use)=>{

const db =

await Database.connect();

await use(db);

await db.close();

},

{

scope:'worker'

}

]

```

Connection

↓

Shared

↓

Closed once.

----------

# Example – API Client

```ts
customerApi:[

async({ request }, use)=>{

const api =

new CustomerApi(request);

await use(api);

},

{

scope:'worker'

}

]

```

Useful if the API client is expensive to initialize.

----------

# Worker Fixture Can Depend On?

Important rule.

Worker fixture

↓

Can depend on

↓

Worker fixtures.

It **cannot** depend on **test-scoped fixtures** like `page` or `context`.

----------

# Why?

Suppose:

```text
Worker Fixture

↓

Depends On

↓

Page

```

Problem:

```text
Worker

↓

Many Tests

↓

Many Pages

```

Which page should it use?

Impossible.

----------

# Invalid Example

```ts
logger:[

async(

{ page },

use

)=>{

}
]

```

❌ Error.

Because:

```text
page

↓

Test Scope

```

----------

# Valid Example

```ts
browser:[

...

]

```

or

```ts
playwright

```

Worker fixtures can depend on worker-scoped fixtures.

----------

# Worker Fixture Dependency

```text
Worker Fixture A

↓

Worker Fixture B

↓

Worker Fixture C

```

Perfectly valid.

----------

# Browser Fixture

Remember:

```text
browser

```

is already:

```text
Worker Scoped

```

Playwright itself uses worker fixtures internally.

----------

# Test Isolation

Question

If worker fixture is shared...

Do tests interfere?

Answer:

Not if the shared object is **read-only** or designed for concurrent use.

Example

Logger

↓

Safe.

----------

Database Connection

↓

Usually safe.

----------

Shared Shopping Cart

↓

Not safe.

----------

# Good Worker Fixtures

✅ Logger

✅ Database Connection Pool

✅ API Client Configuration

✅ Redis Client

✅ Kafka Client

✅ Configuration Loader

✅ Secret Manager

----------

# Poor Worker Fixtures

❌ Page

❌ Login User

❌ Shopping Cart

❌ Current Customer

❌ Mutable Test Data

Those should remain test-scoped.

----------

# Enterprise Example

```text
Worker

↓

Database

↓

Redis

↓

Logger

↓

Configuration

↓

100 Tests

```

Very common.

----------

# Performance Comparison

Test Fixture

```text
Database

↓

100 Connections

```

Worker Fixture

```text
Database

↓

1 Connection

```

Much faster.

----------

# Parallel Workers

Suppose

```text
4 Workers

```

Execution

```text
Worker 1

↓

Database Connection



Worker 2

↓

Database Connection



Worker 3

↓

Database Connection



Worker 4

↓

Database Connection

```

Each worker gets its own instance.

Not global.

----------

# Worker Scope Visualization

```text
Worker 1

Database

├── Test 1

├── Test 2

└── Test 3



Worker 2

Database

├── Test 4

├── Test 5

└── Test 6

```

Two workers.

Two databases.

----------

# Lifecycle Diagram

```text
Worker Starts

↓

Worker Fixture Setup

↓

Run All Assigned Tests

↓

Worker Fixture Cleanup

↓

Worker Ends

```

Very predictable.

----------

# Enterprise Example

```ts
type WorkerFixtures = {

logger: Logger;

database: Database;

};

export const test = base.extend<WorkerFixtures>({

logger:[

async({}, use)=>{

const logger = new Logger();

await use(logger);

},

{

scope:'worker'

}

],

database:[

async({}, use)=>{

const db =

await Database.connect();

await use(db);

await db.close();

},

{

scope:'worker'

}

]

});

```

----------

# Common Mistakes

## ❌ Using `page`

Wrong

```ts
database:[

async(

{ page },

use

)=>{

}
]

```

Worker fixtures cannot depend on test fixtures.

----------

## ❌ Sharing Mutable State

Bad

```text
Current User

Shopping Cart

Current Order

```

One test can affect another.

----------

## ❌ Using Worker Scope Everywhere

Don't make every fixture worker-scoped.

Use it only when sharing is safe and beneficial.

----------

## ❌ Forgetting Parallel Workers

A worker fixture is **not global**.

If you have:

```text
4 Workers

```

You'll have:

```text
4 Database Fixtures

```

One per worker.

----------

# Test Scope vs Worker Scope

Test Scope

Worker Scope

New instance per test

One instance per worker

Maximum isolation

Better performance

Ideal for UI state

Ideal for shared infrastructure

Safe for mutable state

Best for read-only or thread-safe resources

----------

# Decision Guide

Resource

Recommended Scope

Why

`page`

Test

Every test needs a fresh tab

`context`

Test

Keeps cookies and storage isolated

`loginPage`

Test

Depends on `page`

`customerApi`

Test (usually)

Often depends on test-specific state

Logger

Worker

Stateless and reusable

Database Connection Pool

Worker

Expensive to create

Configuration Loader

Worker

Read-only

Secret Manager

Worker

Read-only and reusable

Faker Instance

Test

Avoid shared random state

----------

# Interview Questions

### Q1. What is a worker-scoped fixture?

A fixture created once per worker process and shared by all tests executed within that worker.

----------

### Q2. When should you use worker scope?

For expensive, reusable resources that are safe to share, such as loggers, configuration, or connection pools.

----------

### Q3. Can a worker fixture depend on `page`?

No.

`page` is test-scoped, while worker fixtures exist before any test-specific page is created.

----------

### Q4. Is a worker fixture shared across all workers?

No.

Each worker process gets its own instance.

----------

### Q5. What's the biggest advantage of worker fixtures?

They significantly reduce setup overhead for expensive resources while preserving test isolation where needed.

----------

### Q6. Should page objects be worker-scoped?

No.

They depend on the test-scoped `page` fixture and should therefore remain test-scoped.

----------

# Best Practices

-   Use worker scope only for resources that are expensive to initialize and safe to share.
    
-   Keep UI-related fixtures (`page`, page objects, browser contexts) test-scoped.
    
-   Avoid storing mutable business state in worker fixtures.
    
-   Remember that worker fixtures are shared **within a worker**, not across the entire test run.
    
-   Design shared resources to be thread-safe or effectively read-only.
    
-   Don't optimize prematurely—use worker scope only when it provides a measurable benefit.
    

----------

# ⭐ Enterprise Architecture

```text
                    Worker Process
                          │
          ┌───────────────┼───────────────┐
          │               │               │
       Logger        Database Pool   Config Loader
          │               │               │
          └───────────────┼───────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
      Test 1           Test 2           Test 3
        │                 │                 │
     page/context      page/context     page/context
        │                 │                 │
    loginPage        checkoutPage      customerPage

```

Notice the separation:

-   **Infrastructure resources** (logger, configuration, connection pools) are shared per worker.
    
-   **UI state** (page, context, page objects) remains isolated per test.
    

This balance gives you both **performance** and **reliability**, which is exactly why mature Playwright frameworks rely heavily on worker-scoped fixtures for infrastructure while keeping business interactions test-scoped.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyODk0NDk1NTZdfQ==
-->
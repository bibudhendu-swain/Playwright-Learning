Excellent. This is the final chapter of the Fixtures handbook, and it ties together everything we've learned.

One of the biggest misconceptions in Playwright is:

> **"Worker fixtures replace `beforeAll()`."**

That's **not entirely true**.

They overlap in many use cases, but they are **not equivalent**. Understanding this distinction is important for designing reliable frameworks.

----------

# 📘 Playwright Fixtures Handbook

# Part 16 – Adding Global `beforeAll` / `afterAll` Hooks Using Fixtures

> **`beforeAll()` is tied to a test suite.**
> 
> **Worker fixtures are tied to a worker process.**

That difference changes everything.

----------

# Traditional `beforeAll()`

Most testing frameworks use:

```ts
test.beforeAll(async () => {

    console.log("Suite Started");

});

test.afterAll(async () => {

    console.log("Suite Finished");

});

```

Execution:

```text
beforeAll

↓

Test 1

↓

Test 2

↓

Test 3

↓

afterAll

```

Simple.

----------

# Why People Replace It

Imagine:

```text
Connect Database

↓

Start Mock Server

↓

Load Configuration

↓

Initialize Logger

```

You don't want to repeat this before every test.

Naturally you think:

```text
beforeAll()

```

----------

# Worker Fixtures Solve This

Instead of:

```ts
test.beforeAll(async () => {

    const db = await connect();

});

```

Use:

```ts
database: [

async ({}, use) => {

    const db = await connect();

    await use(db);

    await db.close();

},

{

scope: 'worker'

}

]

```

The database is created once per worker.

----------

# Execution Comparison

## `beforeAll()`

```text
Suite Starts

↓

beforeAll

↓

Tests

↓

afterAll

```

----------

## Worker Fixture

```text
Worker Starts

↓

Database

↓

Tests

↓

Database Cleanup

↓

Worker Ends

```

Notice the lifecycle is different.

----------

# Important Difference

Suppose:

```text
4 Workers

```

Execution

```text
Worker 1

↓

Database



Worker 2

↓

Database



Worker 3

↓

Database



Worker 4

↓

Database

```

You now have:

```text
4 Database Connections

```

Not one.

----------

# `beforeAll()` Scope

Runs once for:

```text
Test File

```

or

```text
Describe Block

```

depending on where it's declared.

----------

# Worker Fixture Scope

Runs once per:

```text
Worker Process

```

Not per file.

----------

# Visual Comparison

## Hook

```text
login.spec.ts

↓

beforeAll

↓

Tests

↓

afterAll

```

----------

## Worker Fixture

```text
Worker

├── login.spec.ts

├── cart.spec.ts

├── checkout.spec.ts

↓

One Database

```

One fixture.

Multiple files.

----------

# Example – Logger

Traditional

```ts
test.beforeAll(async()=>{

logger=new Logger();

});

```

Worker fixture

```ts
logger:[

async({}, use)=>{

const logger=

new Logger();

await use(logger);

},

{

scope:'worker'

}

]

```

Cleaner.

Automatically cleaned up.

----------

# Example – Mock Server

```ts
mockServer:[

async({}, use)=>{

const server=

await start();

await use(server);

await server.stop();

},

{

scope:'worker'

}

]

```

Worker owns the lifecycle.

----------

# Example – Configuration

Instead of

```ts
test.beforeAll(()=>{

loadConfig();

});

```

Worker fixture

```ts
config:[

async({}, use)=>{

const cfg=

loadConfig();

await use(cfg);

},

{

scope:'worker'

}

]

```

Shared by all tests in that worker.

----------

# Database Pool

Enterprise example.

```text
Worker

↓

Database Pool

↓

500 Tests

```

Very common.

----------

# Global Authentication

Avoid

```text
beforeAll

↓

Login UI

```

Prefer

```text
Setup Project

↓

storageState

```

Even better than a worker fixture for UI authentication.

----------

# One-Time Initialization

Good candidates:

```text
Logger

Database Pool

Redis

Kafka

Secrets

Configuration

Mock Server

```

All infrastructure.

----------

# Poor Candidates

```text
Shopping Cart

Current User

Checkout

Place Order

```

These are business state.

Keep them test-scoped.

----------

# Worker Fixture vs `beforeAll()`

Worker Fixture

`beforeAll()`

Dependency injection

No dependency injection

Automatic cleanup

Manual cleanup

Shared per worker

Shared per suite

Can depend on worker fixtures

Doesn't participate in fixture graph

Reusable

Usually local to a file or describe block

----------

# But Hooks Still Matter

Suppose only one file needs:

```text
Create Special Customer

```

Use:

```ts
test.beforeAll(...)

```

No need for a framework-level worker fixture.

----------

# Rule of Thumb

Framework concern?

↓

Worker Fixture.

----------

Suite concern?

↓

Hook.

----------

# Enterprise Architecture

```text
Worker

↓

Configuration

↓

Logger

↓

Database

↓

Mock Server

↓

Spec Files

↓

Tests

```

Infrastructure lives above tests.

----------

# Lifecycle

```text
Worker Starts

↓

Setup

↓

Test 1

↓

Test 2

↓

Test 3

↓

Cleanup

↓

Worker Ends

```

Predictable.

----------

# Mixing Both

Perfectly valid.

```text
Worker Fixture

↓

beforeAll()

↓

Tests

↓

afterAll()

↓

Worker Cleanup

```

Hooks and fixtures can coexist.

----------

# Common Mistakes

## ❌ Thinking `beforeAll()` Runs Once Globally

With multiple workers,

it can execute in more than one process.

Don't assume a single global execution.

----------

## ❌ Database in `beforeAll()`

Better:

Worker fixture.

Automatic cleanup.

Better dependency management.

----------

## ❌ Business Logic in Worker Fixtures

Don't do:

```text
Create Order

Checkout

Delete Customer

```

Infrastructure only.

----------

## ❌ Huge Worker Fixtures

One fixture should not initialize your entire framework.

Split responsibilities.

----------

# Decision Matrix

Need

Best Choice

Database Pool

Worker Fixture

Logger

Worker Fixture

Configuration

Worker Fixture

Redis

Worker Fixture

Kafka

Worker Fixture

Mock Server

Worker Fixture

Suite-specific customer setup

`beforeAll()`

File-specific cleanup

`afterAll()`

----------

# Enterprise Example

Imagine:

```text
600 Tests

8 Workers

```

Execution

```text
Worker 1

↓

Database

↓

75 Tests



Worker 2

↓

Database

↓

75 Tests



...

```

Each worker initializes once.

Performance scales well.

----------

# Framework Comparison

Traditional

```text
beforeAll()

↓

Global Variables

↓

Tests

```

Playwright

```text
Worker Fixtures

↓

Dependency Injection

↓

Tests

```

More modular.

----------

# Interview Questions

### Q1. Do worker fixtures replace `beforeAll()`?

Not completely.

They solve many of the same problems but have a different lifecycle and integrate with Playwright's dependency injection system.

----------

### Q2. What's the biggest difference?

`beforeAll()` is scoped to a suite (or file/describe block).

Worker fixtures are scoped to a worker process.

----------

### Q3. Should database pools use `beforeAll()`?

Usually no.

Worker fixtures are generally a better choice because they support dependency injection and automatic teardown.

----------

### Q4. Can hooks and worker fixtures coexist?

Yes.

Many enterprise frameworks use both.

----------

### Q5. Is worker scope global?

No.

Each worker gets its own instance.

----------

### Q6. When should you still use `beforeAll()`?

For setup that is specific to a single test file or describe block and doesn't belong in the shared framework.

----------

# Best Practices

-   Use worker fixtures for reusable infrastructure.
    
-   Use `beforeAll()` and `afterAll()` for suite-specific preparation and cleanup.
    
-   Prefer `storageState` or a Setup Project over UI login in `beforeAll()`.
    
-   Keep worker fixtures focused on infrastructure, not business workflows.
    
-   Remember that multiple workers mean multiple worker fixture instances.
    
-   Don't force everything into fixtures—choose the abstraction that best matches the scope of the work.
    

----------

# ⭐ Enterprise Framework Architecture

```text
                           Test Runner
                                │
                     ┌──────────┴──────────┐
                     │                     │
                 Worker 1              Worker 2
                     │                     │
      ┌──────────────┼──────────────┐      │
      │              │              │      │
 Configuration     Logger      Database Pool
      │              │              │
      └──────────────┼──────────────┘
                     │
             ┌───────┼────────┐
             │       │        │
        login.spec  cart.spec checkout.spec
             │       │        │
        beforeAll() (optional, file-specific)
             │
          Test Cases
             │
        afterAll() (optional, file-specific)
             │
      Worker Fixture Cleanup
             │
          Worker Ends

```

This architecture demonstrates the recommended layering:

-   **Worker fixtures** provide shared infrastructure for all tests assigned to a worker.
    
-   **`beforeAll()` / `afterAll()` hooks** remain useful for setup and cleanup that is specific to an individual spec file or describe block.
    
-   **Tests** remain focused on business behavior and consume fixtures through dependency injection.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MzA2MzEzMDFdfQ==
-->
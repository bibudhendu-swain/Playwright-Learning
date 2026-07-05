Excellent. This is one of my favorite Playwright features because it often lets you **replace repetitive hooks** with reusable, composable fixtures.

Many teams continue to write everything in `beforeEach()`, even though **automatic fixtures** provide a much cleaner and more scalable solution.

----------

# 📘 Playwright Fixtures Handbook

# Part 8 – Automatic Fixtures (`auto: true`) (Complete Guide)

> **Normal fixtures run only when requested.**
> 
> **Automatic fixtures run for every test (or every worker) without being requested.**

This makes them ideal for:

-   Logging
    
-   Reporting
    
-   Test auditing
    
-   Automatic screenshots
    
-   Environment preparation
    
-   Global cleanup
    

----------

# What is an Automatic Fixture?

Normally, a fixture runs only if you request it.

Example:

```ts
test('Login', async ({ loginPage }) => {

});

```

Only:

```text
loginPage

```

is created.

----------

Automatic fixtures are different.

Even if your test looks like this:

```ts
test('Login', async ({ page }) => {

});

```

the automatic fixture still runs.

----------

# How to Create One

Add:

```ts
auto: true

```

Example

```ts
logger: [

async ({}, use) => {

    console.log('Start Test');

    await use();

    console.log('End Test');

},

{

auto: true

}

]

```

Notice:

The test never requests:

```text
logger

```

Yet it still executes.

----------

# Execution Flow

```text
Automatic Fixture

↓

Setup

↓

Run Test

↓

Cleanup

```

Every test.

----------

# Syntax

Automatic fixtures use the array syntax because they require options.

```ts
fixtureName:[

async(...){

},

{

auto:true

}

]

```

----------

# Example – Logging Every Test

```ts
logger:[

async({}, use)=>{

console.log('===== TEST START =====');

await use();

console.log('===== TEST END =====');

},

{

auto:true

}

]

```

Every test automatically gets logging.

----------

# Test

```ts
test('Login', async ({ page }) => {

});

```

Output

```text
===== TEST START =====

Run Test

===== TEST END =====

```

No fixture requested.

----------

# Example – Measuring Execution Time

```ts
timer:[

async({}, use)=>{

const start = Date.now();

await use();

console.log(

Date.now()-start

);

},

{

auto:true

}

]

```

Every test duration logged automatically.

----------

# Example – Test Audit

```ts
audit:[

async({}, use)=>{

console.log(

'Audit Started'

);

await use();

console.log(

'Audit Finished'

);

},

{

auto:true

}

]

```

No test changes required.

----------

# Example – Capture Logs

```ts
logger:[

async({}, use)=>{

const logs=[];

await use();

console.log(logs);

},

{

auto:true

}

]

```

Useful for reporting.

----------

# Example – Database Cleanup

```ts
cleanup:[

async({}, use)=>{

await use();

await clearDatabase();

},

{

auto:true

}

]

```

Every test automatically cleans up afterward.

----------

# Replacing `beforeEach`

Traditional

```ts
test.beforeEach(async()=>{

await login();

});

```

Automatic Fixture

```ts
login:[

async({}, use)=>{

await login();

await use();

},

{

auto:true

}

]

```

More reusable.

----------

# Automatic Worker Fixture

Combine:

```text
scope:'worker'

+

auto:true

```

Example

```ts
logger:[

async({}, use)=>{

const logger=

new Logger();

await use(logger);

},

{

scope:'worker',

auto:true

}

]

```

Execution

```text
Worker Starts

↓

Logger

↓

All Tests

↓

Logger Destroyed

```

----------

# Auto + Test Scope

```text
Test Starts

↓

Automatic Fixture

↓

Test

↓

Cleanup

```

Every test.

----------

# Auto + Worker Scope

```text
Worker Starts

↓

Automatic Fixture

↓

100 Tests

↓

Cleanup

```

Only once per worker.

----------

# Difference

Normal Fixture

```text
Requested?

↓

Yes

↓

Execute

```

----------

Automatic Fixture

```text
Execute

↓

Always

```

----------

# Enterprise Logging

```ts
logger:[

async({}, use)=>{

console.log(

'Executing',

test.info().title

);

await use();

},

{

auto:true

}

]

```

Every test logged.

----------

# Automatic Screenshots

```ts
screenshot:[

async({ page }, use)=>{

await use();

if(

test.info().status!==

test.info().expectedStatus

){

await page.screenshot();

}

},

{

auto:true

}

]

```

Every failed test automatically gets a screenshot.

> **Note:** In a real implementation, you'll typically use the `testInfo` fixture (covered later) to access status and attach artifacts cleanly.

----------

# Automatic Attachments

```ts
attachment:[

async({}, use)=>{

await use();

// attach logs

},

{

auto:true

}

]

```

Useful for reports.

----------

# Automatic Environment Validation

Before every test

```text
Verify API

↓

Verify Database

↓

Run Test

```

If environment isn't healthy:

Fail early.

----------

# Automatic Mock Setup

```ts
mockApi:[

async({ page }, use)=>{

await page.route(...);

await use();

},

{

auto:true

}

]

```

Every test uses the mock automatically.

----------

# Enterprise Example

```text
Automatic Logger

↓

Automatic Screenshot

↓

Automatic Audit

↓

Automatic Cleanup

↓

Test

```

Test stays tiny.

----------

# Multiple Automatic Fixtures

```ts
logger

audit

timer

cleanup

```

All

↓

Run automatically.

----------

# Lifecycle

```text
Logger Setup

↓

Audit Setup

↓

Timer Setup

↓

Test

↓

Timer Cleanup

↓

Audit Cleanup

↓

Logger Cleanup

```

Reverse cleanup order.

----------

# Good Uses

✅ Logging

✅ Timing

✅ Cleanup

✅ Reporting

✅ Environment Validation

✅ API Mocking

✅ Attachments

----------

# Poor Uses

❌ Place Order

❌ Delete Customer

❌ Login Through UI (usually)

❌ Checkout

Business logic should remain explicit in tests.

----------

# Auto Fixture vs Hook

Auto Fixture

Hook

Can depend on other fixtures

Cannot be injected like fixtures

Reusable across modules

Usually tied to a test file or describe block

Supports dependency injection

Manual setup

Has setup and teardown

Has setup and teardown

Composable

Less composable

----------

# Common Mistakes

## ❌ Making Everything Automatic

Not every fixture should use:

```text
auto:true

```

Only cross-cutting concerns.

----------

## ❌ Hiding Business Logic

Bad

```text
Login

Checkout

Create Customer

```

The test should clearly show its business steps.

----------

## ❌ Long Automatic Fixtures

Automatic fixtures should stay focused.

Example:

```text
Logger

Cleanup

Audit

```

Not 500 lines.

----------

## ❌ Using Auto Instead of Worker Scope

If initialization is expensive but doesn't need to run for every test:

Prefer:

```text
scope:'worker'

```

----------

# Decision Guide

Requirement

Recommendation

Run before every test

`auto: true` (test scope)

Run once per worker

`scope: 'worker'` + `auto: true`

Only when requested

Standard fixture

UI business flow

Explicit fixture or test code

Logging/Reporting

Automatic fixture

----------

# Interview Questions

### Q1. What is an automatic fixture?

A fixture configured with:

```ts
auto:true

```

It executes automatically without being requested by the test.

----------

### Q2. Does a test need to request an automatic fixture?

No.

Playwright runs it automatically.

----------

### Q3. Can an automatic fixture also be worker-scoped?

Yes.

Example:

```ts
{

scope:'worker',

auto:true

}

```

It runs once per worker.

----------

### Q4. When should you use automatic fixtures?

For cross-cutting concerns such as:

-   Logging
    
-   Reporting
    
-   Cleanup
    
-   Environment validation
    
-   Timing
    
-   API mocking
    

----------

### Q5. Should business logic go inside automatic fixtures?

No.

Business scenarios should remain visible in the test itself.

----------

### Q6. What's the difference between an automatic fixture and `beforeEach()`?

Automatic fixtures participate in Playwright's dependency injection system, can depend on other fixtures, and are reusable across modules. `beforeEach()` is simply a lifecycle hook.

----------

# Best Practices

-   Use `auto: true` for cross-cutting infrastructure concerns, not business workflows.
    
-   Keep automatic fixtures small and focused.
    
-   Combine `scope: 'worker'` with `auto: true` for expensive initialization shared by all tests in a worker.
    
-   Prefer fixtures over hooks when you need dependency injection.
    
-   Keep business actions (login, checkout, order placement) explicit unless you're intentionally preparing the environment.
    
-   If an automatic fixture needs test metadata or attachments, use the built-in `testInfo` fixture rather than relying on global state.
    

----------

# ⭐ Enterprise Architecture

A mature framework often has several automatic fixtures layered together:

```text
                    Test Starts
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
 Automatic Logger   Environment Check   Timer
     │                   │                   │
     └───────────────────┼───────────────────┘
                         │
                  Execute Test
                         │
     ┌───────────────────┼───────────────────┐
     │                   │                   │
 Screenshot on Fail   Attach Logs     Database Cleanup
     │                   │                   │
     └───────────────────┼───────────────────┘
                         │
                    Test Ends

```

The test itself stays clean:

```ts
test('Place Order', async ({
  loginPage,
  checkoutPage
}) => {

  await loginPage.login();

  await checkoutPage.placeOrder();

});

```

All the infrastructure—logging, timing, reporting, cleanup, and diagnostics—happens automatically around the test.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM5Nzc2MDU0Nl19
-->
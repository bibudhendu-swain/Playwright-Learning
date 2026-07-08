Excellent. This is one of the most practical chapters in the entire handbook.

If you're coming from **Selenium + TestNG**, **JUnit**, or **Cypress**, you're probably used to putting everything inside:

-   `beforeEach()`
    
-   `afterEach()`
    

In Playwright, **fixtures are often a better choice** because they are composable, dependency-aware, and reusable.

> **Important:** This chapter is not about replacing hooks completely. Hooks still have valid use cases. The goal is to understand **when fixtures are the better abstraction**.

----------

# 📘 Playwright Fixtures Handbook

# Part 15 – Adding Global `beforeEach` / `afterEach` Hooks Using Fixtures

----------

# Traditional Hook Approach

Most automation frameworks start like this.

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/');
  await login();
});

test.afterEach(async ({ page }) => {
  await page.close();
});

```

Works perfectly.

But imagine:

```
200 Spec Files

```

Every file contains:

```
beforeEach()

afterEach()

```

Now maintenance becomes difficult.

----------

# The Problem

Suppose tomorrow your login flow changes.

You now need to update:

```
beforeEach()

↓

200 Files

```

Not ideal.

----------

# Better Architecture

Instead of:

```
Spec File

↓

beforeEach()

```

Use:

```
Automatic Fixture

↓

Every Test

```

One place.

One implementation.

----------

# Why Fixtures Are Better

Fixtures provide:

-   Dependency Injection
    
-   Automatic Cleanup
    
-   Reusability
    
-   Composition
    
-   Test Isolation
    

Hooks don't provide dependency injection.

----------

# Converting `beforeEach`

Traditional

```ts
test.beforeEach(async ({ page }) => {
  await page.goto('/');
});

```

Fixture

```ts
page: [
  async ({ page }, use) => {

    await page.goto('/');

    await use(page);

  },
  {
    auto: true
  }
]

```

Every test now starts on the home page.

----------

# Execution Timeline

```
Fixture Setup

↓

Navigate Home

↓

Run Test

↓

Cleanup

```

Exactly like `beforeEach()`.

----------

# Converting `afterEach`

Traditional

```ts
test.afterEach(async () => {

    console.log('Finished');

});

```

Fixture

```ts
logger: [

async({}, use)=>{

await use();

console.log('Finished');

},

{

auto:true

}

]

```

Everything after:

```ts
await use()

```

acts like:

```
afterEach()

```

----------

# Think of `await use()` as the Divider

Everything before:

```ts
await use()

```

↓

Equivalent to

```
beforeEach()

```

Everything after:

```ts
await use()

```

↓

Equivalent to

```
afterEach()

```

This is one of the most important concepts in Playwright fixtures.

----------

# Visual Representation

```
Fixture Starts

↓

Setup

↓

await use()

↓

Test Executes

↓

Cleanup

↓

Fixture Ends

```

----------

# Global Logging

Instead of

```ts
test.beforeEach(() => {

console.log('Start');

});

test.afterEach(() => {

console.log('End');

});

```

Use

```ts
logger:[

async({}, use)=>{

console.log('Start');

await use();

console.log('End');

},

{

auto:true

}

]

```

Cleaner.

----------

# Global Screenshot

Traditional

```ts
test.afterEach(async ({ page }) => {

await page.screenshot();

});

```

Fixture

```ts
screenshot:[

async({ page }, use)=>{

await use();

await page.screenshot();

},

{

auto:true

}

]

```

Every test automatically captures a screenshot after execution.

> In a real project, you'd typically take the screenshot only on failure by using the `testInfo` fixture.

----------

# Global Timing

```ts
timer:[

async({}, use)=>{

const start=Date.now();

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

Every test gets timing information.

----------

# Global Cleanup

Instead of

```ts
test.afterEach(async()=>{

await deleteCustomer();

});

```

Fixture

```ts
cleanup:[

async({}, use)=>{

await use();

await deleteCustomer();

},

{

auto:true

}

]

```

No repeated hooks.

----------

# Environment Validation

Before every test

```
Ping API

↓

Check Database

↓

Verify Feature Flags

↓

Run Test

```

Perfect use case for an automatic fixture.

----------

# Authentication

You could write

```ts
test.beforeEach(async()=>{

await login();

});

```

But in modern Playwright, prefer:

-   `storageState`
    
-   Setup Project
    
-   Authentication fixture (when appropriate)
    

instead of UI login before every test.

----------

# Multiple Cross-Cutting Concerns

Suppose every test needs:

-   Logging
    
-   Timing
    
-   Cleanup
    
-   Reporting
    

Instead of four hooks:

```
beforeEach()

afterEach()

beforeEach()

afterEach()

```

Create four focused automatic fixtures.

Much easier to maintain.

----------

# Enterprise Architecture

```
Logger

↓

Timer

↓

Environment Check

↓

Test

↓

Cleanup

↓

Reporting

```

Each concern is isolated.

----------

# Hooks vs Fixtures

## Hooks

```
Lifecycle

↓

Code

```

## Fixtures

```
Dependency

↓

Lifecycle

↓

Code

```

Fixtures understand dependencies.

Hooks don't.

----------

# Example

Suppose Logger depends on Configuration.

Fixtures

```
Configuration

↓

Logger

↓

Test

```

Playwright resolves it automatically.

With hooks, you'd manage this relationship manually.

----------

# Execution Order

Automatic Fixtures

↓

Setup

↓

Test

↓

Cleanup

Matches:

```
beforeEach()

↓

Test

↓

afterEach()

```

----------

# Should Hooks Be Avoided?

No.

Hooks are still appropriate for:

-   Test-file-specific setup
    
-   Small one-off scenarios
    
-   Local initialization
    
-   Readability within a single suite
    

Fixtures are generally a better fit for:

-   Shared framework behavior
    
-   Cross-cutting concerns
    
-   Reusable infrastructure
    

----------

# Common Enterprise Pattern

Automatic Fixtures

```
Logger

Timer

Audit

Environment Check

```

Hooks

```
Specific data preparation

Specific cleanup

Specific assertions

```

They complement each other.

----------

# Common Mistakes

## ❌ Moving Every Hook Into Fixtures

Not everything belongs in fixtures.

Example:

```ts
test.beforeEach(async () => {

createCustomer();

});

```

If only one test file needs it,

keep it local.

----------

## ❌ UI Login in Global Fixtures

Avoid

```
Login

↓

Every Test

```

Prefer:

```
storageState

```

or

```
Setup Project

```

----------

## ❌ One Huge Automatic Fixture

Bad

```
Logger

↓

Database

↓

API

↓

Cleanup

↓

Reporting

↓

Screenshots

```

One file.

Hard to maintain.

Split responsibilities.

----------

## ❌ Hiding Business Logic

Fixtures should prepare the environment.

Business steps should remain visible in tests.

----------

# Comparison

Traditional Hook

Automatic Fixture

File-level

Framework-level

Limited reuse

Highly reusable

No dependency injection

Supports dependency injection

Manual organization

Composable

Good for local setup

Good for shared infrastructure

----------

# Interview Questions

### Q1. Can fixtures replace `beforeEach()`?

Often yes, especially for reusable framework-level setup.

----------

### Q2. What part of a fixture acts like `beforeEach()`?

Everything before:

```ts
await use()

```

----------

### Q3. What part acts like `afterEach()`?

Everything after:

```ts
await use()

```

----------

### Q4. Should hooks disappear completely?

No.

Hooks remain useful for suite-specific or local setup and teardown.

----------

### Q5. Why are fixtures preferred for global setup?

Because they support dependency injection, composition, lifecycle management, and reuse across the entire framework.

----------

### Q6. Which is better for logging every test?

An automatic fixture is usually preferable because it centralizes the behavior and integrates with Playwright's fixture system.

----------

# Best Practices

-   Use automatic fixtures for framework-wide concerns.
    
-   Use hooks for test-file-specific behavior.
    
-   Keep one responsibility per fixture.
    
-   Keep business workflows explicit in tests.
    
-   Use `storageState` for authentication rather than logging in before every test.
    
-   Remember that everything before `await use()` is setup, and everything after is teardown.
    

----------

# ⭐ Enterprise Example

A mature framework might organize global behavior like this:

```text
                 Test Starts
                      │
     ┌────────────────┼────────────────┐
     │                │                │
 Environment      Execution        Timer
   Check           Logger
     │                │                │
     └────────────────┼────────────────┘
                      │
                 Execute Test
                      │
     ┌────────────────┼────────────────┐
     │                │                │
 Screenshot      Audit Log       Cleanup
  (if failed)                     Tasks
     │                │                │
     └────────────────┼────────────────┘
                      │
                  Test Ends

```

The test itself remains focused:

```ts
test('Complete Checkout', async ({
  checkoutPage,
  customerApi
}) => {

  await customerApi.createCustomer();

  await checkoutPage.placeOrder();

});

```

No logging code.

No timing code.

No cleanup code.

No screenshot logic.

All framework-level behavior runs automatically around the test.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExMDI4MDQ5MV19
-->
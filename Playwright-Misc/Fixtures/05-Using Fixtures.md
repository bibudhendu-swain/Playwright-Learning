Excellent. This is where developers usually go from _"I know how to create a fixture"_ to _"I know how to build a framework."_

Creating fixtures is only half the story. The real power comes from **using them together** to build reusable, maintainable test architecture.

----------

# 📘 Playwright Fixtures Handbook

# Part 5 – Using Fixtures (Complete Guide)

> **Creating a fixture is easy.**
> 
> **Designing and using fixtures effectively is what separates a beginner Playwright framework from an enterprise-grade one.**

----------

# What Does "Using a Fixture" Mean?

Once you've created a fixture using `test.extend()`, using it is incredibly simple.

Suppose you have:

```ts
loginPage

```

Instead of:

```ts
const loginPage = new LoginPage(page);

```

You simply request it.

```ts
test('Login', async ({ loginPage }) => {

});

```

Playwright injects it automatically.

----------

# Step 1 – Import Your Custom Test

Instead of importing Playwright's built-in `test`:

```ts
import { test, expect } from '@playwright/test';

```

Import your extended version:

```ts
import { test, expect } from '../fixtures/base.fixture';

```

This is one of the most common mistakes beginners make.

----------

# Example Fixture

Suppose:

```ts
export const test = base.extend({

    loginPage: async ({ page }, use) => {

        await use(new LoginPage(page));

    }

});

```

----------

# Using It

```ts
test('Login', async ({ loginPage }) => {

    await loginPage.login();

});

```

Notice:

No object creation.

----------

# Multiple Fixtures

Suppose framework provides:

```text
loginPage

dashboardPage

customerApi

```

Use them together.

```ts
test('Customer', async ({

loginPage,

dashboardPage,

customerApi

}) => {

});

```

Playwright injects all three.

----------

# Dependency Resolution

Suppose:

```text
dashboardPage

↓

depends on

↓

page

```

You request:

```text
dashboardPage

```

Playwright automatically creates:

```text
Browser

↓

Context

↓

Page

↓

DashboardPage

```

No manual setup.

----------

# Request Only What You Need

Test 1

```ts
test('Login', async ({

loginPage

}) => {

});

```

Only creates:

```text
LoginPage

```

----------

Test 2

```ts
test('Checkout', async ({

checkoutPage,

customerApi

}) => {

});

```

Creates:

```text
CheckoutPage

CustomerApi

```

Nothing else.

Lazy initialization in action.

----------

# Combining Fixtures

Example

```ts
test('Order', async ({

loginPage,

productPage,

cartPage,

checkoutPage

}) => {

});

```

Business flow becomes obvious.

----------

# Real Enterprise Example

```ts
test('Place Order', async ({

loginPage,

productPage,

cartPage,

checkoutPage

}) => {

    await loginPage.login();

    await productPage.search('Laptop');

    await cartPage.add();

    await checkoutPage.placeOrder();

});

```

No setup code.

Pure business logic.

----------

# API + UI Together

Suppose:

```ts
test('Create Customer', async ({

customerApi,

customerPage

}) => {

});

```

Flow

```text
API

↓

Create Customer

↓

UI

↓

Verify Customer

```

Fast.

Reliable.

----------

# Database + UI

```ts
test('Verify Customer', async ({

database,

customerPage

}) => {

});

```

Flow

```text
Database

↓

Insert Data

↓

UI

↓

Validate

```

----------

# Logger Fixture

```ts
test('Checkout', async ({

logger,

checkoutPage

}) => {

logger.info('Checkout Started');

});

```

No logger creation.

----------

# Faker Fixture

Instead of

```ts
const faker = new Faker();

```

Use

```ts
test('Register', async ({

faker

}) => {

});

```

----------

# Environment Fixture

Instead of

```ts
process.env.BASE_URL

```

everywhere.

Create

```text
environment

```

fixture.

Usage

```ts
test('Example', async ({

environment

}) => {

});

```

Much cleaner.

----------

# Multiple Page Objects

```ts
test('Order', async ({

homePage,

productPage,

cartPage,

checkoutPage,

confirmationPage

}) => {

});

```

Immediately obvious what the test uses.

----------

# Composition Instead of Inheritance

Traditional

```text
BaseTest

↓

Everything

```

Playwright

```text
Request

Only

Needed

Fixtures

```

Much cleaner.

----------

# Sharing Fixtures

Framework

```text
fixtures/

base.fixture.ts

```

All tests

↓

Import

↓

Same fixtures.

----------

# Fixture File

Example

```ts
export {

test,

expect

};

```

Every test imports from here.

----------

# Folder Structure

```text
fixtures/

base.fixture.ts

api.fixture.ts

page.fixture.ts

database.fixture.ts

logger.fixture.ts

```

We'll combine these later.

----------

# Readability Comparison

Without Fixtures

```ts
const login = ...

const dashboard = ...

const api = ...

const db = ...

```

Business logic starts on line 20.

----------

With Fixtures

```ts
test('Order', async ({

checkoutPage,

customerApi

}) => {

});

```

Business logic starts immediately.

----------

# One Test

```ts
test('Search', async ({

searchPage

}) => {

});

```

Another

```ts
test('Payment', async ({

paymentPage,

customerApi,

logger

}) => {

});

```

Each test requests different fixtures.

----------

# Dependency Graph

```text
checkoutPage

↓

cartPage

↓

productPage

↓

page

↓

context

↓

browser

```

One request.

Entire chain resolved.

----------

# Enterprise Example

```ts
test('Complete Purchase', async ({

loginPage,

productPage,

cartPage,

checkoutPage,

customerApi,

logger

}) => {

logger.info('Starting Purchase');

await loginPage.login();

await customerApi.prepareTestData();

await productPage.open();

await cartPage.checkout();

});

```

Everything injected.

----------

# Test Signature as Documentation

One underrated benefit.

Looking at:

```ts
test('Refund', async ({

refundPage,

paymentApi,

database

}) => {

});

```

You instantly know:

This test uses:

-   Refund Page
    
-   Payment API
    
-   Database
    

No scrolling.

----------

# Fixtures Can Be Mixed

Built-in

```ts
page

```

Custom

```ts
loginPage

```

Together

```ts
test('Example', async ({

page,

loginPage

}) => {

});

```

Perfectly valid.

----------

# Common Mistakes

## ❌ Importing the Wrong `test`

Wrong

```ts
import {

test

}

from '@playwright/test';

```

Custom fixtures won't be available.

Correct

```ts
import {

test

}

from '../fixtures/base.fixture';

```

----------

## ❌ Creating Page Objects Again

Wrong

```ts
const login = new LoginPage(page);

```

If a `loginPage` fixture already exists, use it instead.

----------

## ❌ Requesting Unused Fixtures

Bad

```ts
test('Login', async ({

loginPage,

checkoutPage,

logger,

database

}) => {

});

```

Only request what you actually use. Although Playwright lazily initializes fixtures, requesting unnecessary fixtures can still increase setup work and make tests harder to read.

----------

## ❌ Huge Fixture Lists

Bad

```text
15 Fixtures

```

Usually indicates:

Test doing too much.

----------

# Interview Questions

### Q1. How do you use a custom fixture?

Import the extended `test` object and request the fixture as a parameter in the test function.

----------

### Q2. Can you use built-in and custom fixtures together?

Yes.

Example:

```ts
test('Example', async ({ page, loginPage }) => {

});

```

----------

### Q3. Do you create page objects manually after creating fixtures?

No.

The fixture should provide the page object.

----------

### Q4. Can one test use multiple fixtures?

Absolutely.

Playwright resolves all requested dependencies automatically.

----------

### Q5. Why should you import your custom `test`?

Because Playwright's default `test` knows only about built-in fixtures. Your extended `test` includes your custom fixtures.

----------

# Best Practices

-   Create a single `base.fixture.ts` (or similarly named entry point) that exports your extended `test` and `expect`.
    
-   Import that custom `test` consistently throughout your project.
    
-   Request only the fixtures a test genuinely needs.
    
-   Keep test signatures readable; if you regularly need 10–15 fixtures, reconsider the test design.
    
-   Let fixtures handle object creation so tests focus on business behavior.
    
-   Combine built-in and custom fixtures naturally—there's no need to replace built-in fixtures unless you have a good reason.
    

----------

# ⭐ Enterprise Fixture Architecture

A common enterprise structure looks like this:

```text
                 Test
                   │
     ┌─────────────┼─────────────┐
     │             │             │
 loginPage    customerApi     logger
     │             │
     ▼             ▼
    page        request
       \         /
        ▼       ▼
      Browser Context
             │
         Browser

```

The test becomes extremely clean:

```ts
import { test, expect } from '../fixtures/base.fixture';

test('Complete Checkout', async ({
  loginPage,
  productPage,
  cartPage,
  checkoutPage,
  customerApi
}) => {

  await customerApi.createCustomer();

  await loginPage.login();

  await productPage.addProductToCart();

  await checkoutPage.placeOrder();

});

```

Notice what's missing:

-   No `new LoginPage(...)`
    
-   No `new CustomerApi(...)`
    
-   No browser setup
    
-   No cleanup code
    

Only the business workflow remains.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNjI1MjcxNDQ2XX0=
-->
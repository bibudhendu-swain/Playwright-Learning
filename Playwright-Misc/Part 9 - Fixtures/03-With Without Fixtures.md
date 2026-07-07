Many developers say, **"I understand fixtures."** But they're still writing Playwright code exactly like Selenium—creating page objects manually, using BaseTest classes, and repeating setup code.

This chapter explains **why fixtures exist** by comparing three approaches.

----------

# 📘 Playwright Fixtures Handbook

# Part 3 – Without Fixtures vs With Fixtures (Complete Guide)

> **This chapter answers one question:**
> 
> **"Why should I use fixtures instead of simply creating objects manually?"**

We'll compare:

1.  Selenium/TestNG approach
    
2.  Plain Playwright (without custom fixtures)
    
3.  Playwright with custom fixtures (recommended)
    

----------

# Scenario

Let's automate a login flow.

We'll use:

-   LoginPage
    
-   DashboardPage
    
-   CustomerApi
    
-   Logger
    

----------

# Option 1 – Traditional Selenium/TestNG

Most Selenium frameworks look like this.

```java
public class LoginTest extends BaseTest {

    @Test
    public void login() {

        LoginPage login = new LoginPage(driver);

        DashboardPage dashboard =
            new DashboardPage(driver);

        CustomerApi api = new CustomerApi();

        Logger logger = new Logger();

    }

}

```

Every test repeats:

-   Page Object creation
    
-   API creation
    
-   Logger creation
    

----------

# Architecture

```text
BaseTest

↓

Driver

↓

Page Objects

↓

Test

```

Everything depends on the BaseTest.

----------

# Problems

Imagine:

```text
500 Tests

```

Each test creates:

```text
LoginPage

DashboardPage

ProductPage

CustomerApi

Logger

```

That's a lot of repeated code.

----------

# Option 2 – Plain Playwright (Without Custom Fixtures)

Many beginners migrate from Selenium and write this:

```ts
test('Login', async ({ page }) => {

    const loginPage = new LoginPage(page);

    const dashboardPage =
        new DashboardPage(page);

    const api = new CustomerApi();

    const logger = new Logger();

});

```

This works.

But it doesn't scale well.

----------

# What's Wrong?

Nothing is technically incorrect.

The problem is repetition.

Every test creates:

```text
new LoginPage()

new DashboardPage()

new CustomerApi()

new Logger()

```

Again.

And again.

----------

# Example

Test 1

```ts
const login = new LoginPage(page);

```

Test 2

```ts
const login = new LoginPage(page);

```

Test 300

```ts
const login = new LoginPage(page);

```

The same code appears hundreds of times.

----------

# Another Problem

Suppose constructor changes.

Old

```ts
new LoginPage(page)

```

New

```ts
new LoginPage(page, logger)

```

Now every test must change.

Potentially:

```text
500 Files

```

----------

# Another Example

Suppose logger changes.

Old

```ts
new Logger()

```

New

```ts
new Logger(reporter)

```

Again:

Hundreds of files.

----------

# Maintenance Cost

```text
Constructor Change

↓

500 Tests

↓

500 Updates

```

Not ideal.

----------

# Option 3 – Custom Fixtures

Now look at the Playwright way.

```ts
test('Login', async ({

loginPage,

dashboardPage,

customerApi,

logger

}) => {

});

```

No object creation.

Everything injected.

----------

# Compare Side by Side

Without Fixtures

```ts
const login = new LoginPage(page);

const dashboard =
    new DashboardPage(page);

const api = new CustomerApi();

```

----------

With Fixtures

```ts
test('Login', async ({

loginPage,

dashboardPage,

customerApi

}) => {

});

```

Huge difference.

----------

# Boilerplate Comparison

Without Fixtures

```text
Test

↓

Create LoginPage

↓

Create DashboardPage

↓

Create Logger

↓

Create API

```

----------

With Fixtures

```text
Test

↓

Use Fixtures

```

----------

# Constructor Changes

Suppose LoginPage changes.

Old

```ts
new LoginPage(page)

```

New

```ts
new LoginPage(page, logger)

```

Without Fixtures

```text
500 Tests Updated

```

----------

With Fixtures

Only

```text
login.fixture.ts

```

changes.

Tests remain untouched.

----------

# Dependency Injection

Instead of

```ts
const loginPage =
    new LoginPage(page);

```

Playwright performs:

```text
Create LoginPage

↓

Inject LoginPage

↓

Test

```

Exactly like Spring Boot.

----------

# Another Example

Test

```ts
test('Checkout', async ({

checkoutPage

}) => {

});

```

CheckoutPage depends on:

```text
Page

```

Page depends on:

```text
Context

```

Context depends on:

```text
Browser

```

Playwright resolves all dependencies automatically.

----------

# Manual Dependency Chain

Without fixtures:

```text
Browser

↓

Context

↓

Page

↓

LoginPage

↓

CheckoutPage

```

Everything created manually.

----------

With fixtures:

```text
CheckoutPage Requested

↓

Playwright Resolves

↓

Browser

↓

Context

↓

Page

↓

CheckoutPage

```

One request.

Everything handled automatically.

----------

# Test Readability

Without

```ts
const loginPage = ...

const dashboard = ...

const logger = ...

const api = ...

```

Business logic starts much later.

----------

With

```ts
test('Checkout', async ({

checkoutPage

}) => {

});

```

Business logic starts immediately.

----------

# Large Project Example

Suppose:

```text
30 Page Objects

20 APIs

10 Utilities

```

Without fixtures:

Every test creates them manually.

----------

With fixtures:

Request only what you need.

----------

# Lazy Loading

Suppose test needs:

```text
CheckoutPage

```

Playwright creates:

```text
CheckoutPage

```

Only.

It doesn't create:

```text
LoginPage

AdminPage

ReportPage

```

unless requested.

----------

# Reusability

One fixture.

```text
LoginPage

```

Used by:

```text
1000 Tests

```

----------

# Cleaner Tests

Instead of

```ts
const dashboard = ...

const login = ...

const api = ...

```

You immediately see:

```ts
test('Checkout', async ({

checkoutPage,

customerApi

}) => {

});

```

The dependencies are explicit in the function signature.

----------

# Better IDE Experience

Typing:

```text
loginPage.

```

Shows IntelliSense immediately.

No scrolling through object creation code.

----------

# Easier Refactoring

Suppose LoginPage constructor changes.

Only:

```text
Fixture

```

changes.

Tests stay the same.

----------

# Dependency Graph

Without

```text
Test

↓

Creates

↓

Everything

```

----------

With

```text
Test

↓

Requests

↓

Dependencies

```

Huge architectural difference.

----------

# Framework Architecture

Traditional

```text
BaseTest

↓

Driver

↓

Page Objects

↓

Test

```

----------

Playwright

```text
Fixtures

↓

Dependency Injection

↓

Test

```

No inheritance required.

----------

# Enterprise Example

Without Fixtures

```ts
const loginPage = new LoginPage(page);

const dashboardPage =
    new DashboardPage(page);

const customerApi =
    new CustomerApi(request);

const logger =
    new Logger();

```

----------

With Fixtures

```ts
test('Order', async ({

loginPage,

dashboardPage,

customerApi,

logger

}) => {

});

```

Business logic only.

----------

# Code Size Comparison

Without Fixtures

```text
20 lines

↓

Object Creation

```

Business logic:

```text
5 lines

```

----------

With Fixtures

Business logic starts immediately.

----------

# Real Enterprise Pattern

Instead of

```text
new LoginPage()

new ProductPage()

new CartPage()

new CheckoutPage()

new CustomerApi()

```

Framework exposes:

```text
loginPage

productPage

cartPage

checkoutPage

customerApi

```

All injected.

----------

# Comparison Table

Without Fixtures

With Fixtures

Manual object creation

Automatic dependency injection

Boilerplate in every test

Clean test signatures

Constructor changes affect many tests

Constructor changes usually affect only the fixture

More setup code

Business logic first

Manual lifecycle management

Automatic lifecycle management

Harder to refactor

Easier to refactor

Similar to Selenium

Idiomatic Playwright

----------

# Common Mistakes

## ❌ Creating Page Objects in Every Test

```ts
const login = new LoginPage(page);

```

Repeated hundreds of times.

Better:

Fixture.

----------

## ❌ Creating Utility Objects Manually

```ts
new CustomerApi()

new Logger()

```

Move them into fixtures when they are shared dependencies.

----------

## ❌ Keeping Selenium Style

```text
BaseTest

↓

new PageObject()

```

Playwright encourages dependency injection instead.

----------

## ❌ Massive Constructors in Tests

When every test starts with 10–20 lines of setup, the intent of the test becomes harder to read.

----------

# Interview Questions

### Q1. Why use fixtures instead of creating page objects manually?

Because fixtures centralize object creation, reduce boilerplate, improve maintainability, and let Playwright manage lifecycle automatically.

----------

### Q2. What is the biggest advantage of fixtures?

Dependency Injection.

----------

### Q3. If the `LoginPage` constructor changes, how many tests change?

Without fixtures:

Potentially hundreds.

With fixtures:

Usually only the fixture definition.

----------

### Q4. Do fixtures improve readability?

Yes.

The test immediately shows its dependencies in the parameter list, making the business logic easier to understand.

----------

### Q5. Are fixtures only for Page Objects?

No.

Fixtures can provide:

-   Page Objects
    
-   API clients
    
-   Database connections
    
-   Loggers
    
-   Test data generators
    
-   Authentication helpers
    
-   Utility classes
    

----------

# Best Practices

-   Avoid creating shared objects manually in every test.
    
-   Keep tests focused on business logic, not object construction.
    
-   Use fixtures to inject reusable dependencies.
    
-   Let constructor changes be isolated to fixture definitions.
    
-   Prefer composition (fixtures) over inheritance (`BaseTest`).
    
-   Request only the fixtures you actually need to benefit from lazy initialization.
    

----------

# Enterprise Refactoring Example

### Before

```text
Test

↓

Create Page Objects

↓

Create APIs

↓

Create Logger

↓

Business Logic

```

### After

```text
Test

↓

Injected Fixtures

↓

Business Logic

```

The result is cleaner, easier to maintain, and more aligned with Playwright's design philosophy.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzg2MjA2NjU2XX0=
-->
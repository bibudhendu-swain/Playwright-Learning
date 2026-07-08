# 📘 Playwright Fixtures Handbook

# Part 1 – Introduction to Fixtures (Complete Guide)

> **If you understand Fixtures, you understand Playwright.**
> 
> Almost every advanced Playwright framework—whether developed by Microsoft, enterprise companies, or open-source teams—relies heavily on fixtures.

----------

# What is a Fixture?

The official documentation says:

> A fixture is an object that Playwright sets up before your test and tears down after your test.

While correct, this definition doesn't really explain _why_ fixtures exist.

A better way to think about it is:

> **A fixture is Playwright's Dependency Injection (DI) mechanism.**

If you've worked with:

-   Spring Boot → Dependency Injection
    
-   .NET → Services
    
-   Angular → Dependency Injection
    
-   NestJS → Providers
    

then Playwright fixtures are conceptually very similar.

Instead of creating objects yourself:

```ts
const loginPage = new LoginPage(page);

const api = new CustomerApi();

const db = new Database();

```

Playwright can automatically create them, manage their lifecycle, inject them into your test, and clean them up afterward.

----------

# Why Do Fixtures Exist?

Imagine a project with 500 test cases.

Without fixtures:

```text
Test 1

↓

Launch Browser

↓

Create Context

↓

Create Page

↓

Login

↓

Create Page Objects

↓

Run Test

```

Test 2:

```text
Launch Browser

↓

Create Context

↓

Create Page

↓

Login

↓

Create Page Objects

```

Test 500:

Same process again.

That's a lot of repeated setup logic.

----------

# The Problem Fixtures Solve

Suppose every test needs:

-   Browser
    
-   Page
    
-   Login
    
-   HomePage
    
-   ProductPage
    
-   API client
    
-   Logger
    
-   Database connection
    

Without fixtures:

Every test creates everything manually.

With fixtures:

Playwright injects only what the test requests.

----------

# Think of Fixtures Like a Hotel

Imagine checking into a hotel.

You don't build:

-   the room
    
-   the bed
    
-   the bathroom
    
-   the Wi-Fi
    

The hotel prepares everything before you arrive.

You simply use it.

When you leave:

The hotel cleans everything up.

Fixtures work the same way.

----------

# Lifecycle

Every fixture follows the same lifecycle.

```text
Create

↓

Initialize

↓

Use

↓

Cleanup

```

Internally:

```text
Setup

↓

Test Executes

↓

Teardown

```

This lifecycle is automatic.

----------

# What Can Be a Fixture?

Almost anything.

Examples:

```text
Browser

Browser Context

Page

Login User

API Client

Database Connection

Page Objects

Environment

Logger

Random Test Data

Authentication Token

Email Client

Kafka Client

Redis

GraphQL Client

```

If your test needs it, it can probably be a fixture.

----------

# Built-in vs Custom Fixtures

Two types exist.

## Built-in

Provided by Playwright.

Examples:

```text
browser

context

page

request

browserName

```

----------

## Custom

Created by you.

Examples:

```text
loginPage

dashboardPage

adminUser

customerApi

database

logger

```

----------

# How Fixtures Are Injected

Look at this test:

```ts
test('Login', async ({ page }) => {

});

```

Where did `page` come from?

You never wrote:

```ts
const page = ...

```

Playwright injected it.

This is dependency injection.

----------

# Another Example

```ts
test('Checkout', async ({ page, request }) => {

});

```

Playwright creates:

```text
Browser

↓

Context

↓

Page

↓

API Request Context

```

Then injects both into the test.

----------

# The Magic of Lazy Initialization

One of the smartest parts of Playwright.

Suppose:

```ts
test('Example', async ({ page }) => {

});

```

Only `page` is requested.

Playwright creates:

```text
Browser

↓

Context

↓

Page

```

Nothing else.

----------

Another test:

```ts
test('API', async ({ request }) => {

});

```

Playwright creates only:

```text
API Request Context

```

No browser.

No page.

This is called **lazy initialization**.

Only requested fixtures are created.

----------

# Dependency Graph

Fixtures can depend on other fixtures.

Example:

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

Playwright automatically resolves the dependency graph.

----------

# Example

Suppose:

```text
CheckoutPage

depends on

↓

Page

depends on

↓

Context

depends on

↓

Browser

```

You request:

```ts
test('Checkout', async ({ checkoutPage }) => {

});

```

Playwright automatically creates:

```text
Browser

↓

Context

↓

Page

↓

CheckoutPage

```

You only requested one object.

Playwright handled the rest.

----------

# Manual Approach

Traditional Selenium:

```java
WebDriver driver = new ChromeDriver();

LoginPage login = new LoginPage(driver);

DashboardPage dashboard = new DashboardPage(driver);

CustomerApi api = new CustomerApi();

```

Every test repeats this.

----------

# Fixture Approach

```ts
test('Checkout', async ({

checkoutPage,

customerApi

}) => {

});

```

Everything is already available.

----------

# Fixture Responsibility

A fixture should have **one responsibility**.

Good examples:

```text
loginPage

cartPage

customerApi

database

logger

```

Bad example:

```text
everythingFixture

```

One fixture should not create half of your framework.

----------

# Fixtures Are Not Variables

Many beginners think:

```ts
page

```

is just a variable.

It is not.

It's a managed object.

Playwright controls:

-   Creation
    
-   Lifetime
    
-   Cleanup
    

----------

# Setup and Cleanup

Internally every fixture looks like this:

```text
Setup

↓

Yield Object

↓

Test

↓

Cleanup

```

This becomes important when we start writing custom fixtures.

----------

# Test Isolation

Every test gets fresh fixtures.

Example:

```text
Test 1

↓

New Context

↓

New Page

```

Test 2

```text
New Context

↓

New Page

```

Nothing is shared unless you explicitly choose to share it (using worker-scoped fixtures, which we'll cover later).

----------

# Common Enterprise Fixtures

Real enterprise frameworks commonly include:

```text
loginPage

homePage

checkoutPage

orderPage

customerApi

productApi

database

logger

faker

mailbox

testData

```

Tests become much cleaner.

----------

# Traditional Framework vs Playwright

Traditional framework:

```text
Test

↓

BaseTest

↓

Driver

↓

Page Object

```

Playwright:

```text
Fixture

↓

Fixture

↓

Fixture

↓

Test

```

The framework becomes modular instead of relying on inheritance.

----------

# Why Microsoft Chose Fixtures

Instead of:

```text
Inheritance

↓

Base Classes

↓

Singletons

```

Microsoft chose:

```text
Dependency Injection

↓

Fixtures

↓

Composition

```

This scales much better.

----------

# Internal Architecture

Conceptually, Playwright works like this:

```text
Test Starts

↓

Read Requested Fixtures

↓

Resolve Dependencies

↓

Create Fixtures

↓

Execute Test

↓

Destroy Fixtures

```

You never manually create or destroy them.

----------

# Benefits

## Cleaner Tests

Instead of:

```ts
const login = new LoginPage(page);

const dashboard = new DashboardPage(page);

const api = new CustomerApi();

```

Simply write:

```ts
test('Checkout', async ({

loginPage,

dashboardPage,

customerApi

}) => {

});

```

----------

## Less Boilerplate

No repeated setup code.

----------

## Better Reusability

One fixture.

Hundreds of tests.

----------

## Automatic Cleanup

No need for:

```ts
driver.quit();

```

Playwright handles cleanup automatically.

----------

## Better Parallel Execution

Each test gets isolated fixtures, making parallel execution safer and reducing interference between tests.

----------

# Common Misconceptions

## ❌ Fixtures are Page Objects

No.

A Page Object can be a fixture.

A fixture is broader—it can represent any reusable dependency.

----------

## ❌ Fixtures replace Hooks

No.

Fixtures and hooks solve different problems.

Hooks:

```text
beforeEach

afterEach

```

Fixtures:

```text
Dependency Injection

```

We'll compare them in later chapters.

----------

## ❌ Fixtures are Global Variables

No.

Each test receives its own fixture instances unless they're explicitly defined as worker-scoped.

----------

# Interview Questions

### Q1. What is a fixture?

A fixture is a managed dependency that Playwright creates before a test, injects into the test, and cleans up afterward.

----------

### Q2. Why are fixtures used?

To eliminate repetitive setup code, provide dependency injection, improve test isolation, and manage resource lifecycles automatically.

----------

### Q3. What is the biggest advantage of fixtures?

Automatic lifecycle management combined with dependency injection.

----------

### Q4. Can fixtures depend on other fixtures?

Yes.

Playwright automatically resolves fixture dependencies.

Example:

```text
Browser

↓

Context

↓

Page

↓

LoginPage

```

----------

### Q5. Are fixtures created for every test?

Test-scoped fixtures are.

Worker-scoped fixtures are shared across tests running in the same worker. We'll cover those in a later chapter.

----------

### Q6. Do unused fixtures get created?

No.

Playwright uses lazy initialization and creates only the fixtures that are actually requested.

----------

# Best Practices

-   Keep each fixture focused on a single responsibility.
    
-   Prefer composition (fixtures) over inheritance (`BaseTest` patterns).
    
-   Don't create objects manually if they can be injected as fixtures.
    
-   Let Playwright manage setup and teardown.
    
-   Avoid storing mutable shared state in test-scoped fixtures.
    
-   Name fixtures clearly (`loginPage`, `cartPage`, `customerApi`) so their purpose is obvious.
    

----------

# Enterprise Example

A mature Playwright framework might expose fixtures like this:

```text
                Test
                  │
      ┌───────────┼───────────┐
      │           │           │
 loginPage   customerApi   logger
      │           │
      ▼           ▼
     page      request
        \       /
         ▼     ▼
     Browser Context
             │
         Browser

```

The test only requests what it needs:

```ts
test('Place Order', async ({ loginPage, customerApi }) => {
  // Business logic only
});

```

There are no manual object creations, no driver management, and no explicit cleanup.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIyODg5NjgxNV19
-->
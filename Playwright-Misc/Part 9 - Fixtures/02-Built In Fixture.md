Excellent. This is one of the most important chapters because **every custom fixture is built on top of these built-in fixtures**. If you don't understand these, the later chapters (custom fixtures, worker fixtures, fixture execution order) become much harder.

----------

# 📘 Playwright Fixtures Handbook

# Part 2 – Built-in Fixtures (Complete Guide)

> **Everything in Playwright starts with built-in fixtures.**
> 
> Every custom fixture you create will ultimately depend on one or more of these.

----------

# What are Built-in Fixtures?

Playwright automatically provides several fixtures that you can use without creating them yourself.

For example:

```ts
import { test } from '@playwright/test';

test('Example', async ({ page }) => {
  await page.goto('https://playwright.dev');
});

```

Where did `page` come from?

You didn't create it.

Playwright injected it.

----------

# Built-in Fixtures Overview

Playwright provides both **test-scoped** and **worker-scoped** fixtures.

## Test-Scoped Fixtures

Created for every test.

Fixture

Type

Scope

`page`

Page

Test

`context`

BrowserContext

Test

`request`

APIRequestContext

Test

----------

## Worker-Scoped Fixtures

Created once per worker.

Fixture

Type

Scope

`browser`

Browser

Worker

`browserName`

string

Worker

`playwright`

Playwright

Worker

We'll learn later how to create our own worker-scoped fixtures.

----------

# Relationship Between Fixtures

This is one of the most important diagrams.

```text
Browser (Worker)

        │
        ▼

Browser Context (Test)

        │
        ▼

Page (Test)

```

Notice:

-   Browser is shared inside a worker.
    
-   Context is isolated per test.
    
-   Page is isolated per test.
    

----------

# Lifecycle Diagram

Suppose we have two tests.

```text
Worker Starts

↓

Browser Created

↓

Test 1

↓

Context 1

↓

Page 1

↓

Destroy Page

↓

Destroy Context

↓

Test 2

↓

Context 2

↓

Page 2

↓

Destroy Page

↓

Destroy Context

↓

Destroy Browser

```

The browser survives across tests.

The page does not.

----------

# 1. `page`

Probably the most used fixture.

Example

```ts
test('Login', async ({ page }) => {

  await page.goto('/login');

});

```

Type

```ts
Page

```

Represents:

```text
Browser Tab

```

Everything UI-related happens through `page`.

----------

## Common Methods

```ts
await page.goto('/');

await page.click();

await page.fill();

await page.locator();

await page.screenshot();

```

----------

## Lifetime

```text
One Test

↓

One Page

↓

Destroyed

```

Every test receives a new page.

----------

# Why One Page Per Test?

Imagine:

```text
Test A

↓

Logged In

↓

Adds Item

```

If Test B reused the same page:

```text
Shopping Cart Already Filled

```

Tests would interfere.

Playwright avoids this.

----------

# 2. `context`

Type

```ts
BrowserContext

```

Represents:

```text
Incognito Browser Window

```

Each context has:

-   Cookies
    
-   Local Storage
    
-   Session Storage
    
-   Permissions
    
-   Cache
    

Everything isolated.

----------

Example

```ts
test('Example', async ({ context }) => {

});

```

----------

Context Creates Pages

```text
Browser

↓

Context

↓

Page 1

↓

Page 2

↓

Page 3

```

----------

Multiple Tabs

```ts
const secondPage = await context.newPage();

```

Useful for:

-   Payment windows
    
-   Admin portals
    
-   Email verification
    
-   Multi-user scenarios
    

----------

# Why Context Instead of Browser?

Creating a browser is expensive.

Creating a context is cheap.

Instead of:

```text
Chrome

Chrome

Chrome

```

Playwright creates:

```text
Chrome

↓

Context A

↓

Context B

↓

Context C

```

Much faster.

----------

# 3. `browser`

Type

```ts
Browser

```

Worker-scoped.

Example

```ts
test('Example', async ({ browser }) => {

});

```

----------

Normally

You don't need it.

Playwright already provides:

```text
context

page

```

----------

When Do You Need Browser?

Example:

```ts
const secondContext = await browser.newContext();

```

Useful for:

-   Multi-user testing
    
-   Buyer vs Seller
    
-   Admin vs Customer
    
-   Chat applications
    

----------

Example

```text
Browser

├── Context A

│      User 1

│

└── Context B

       User 2

```

Both users interact simultaneously.

----------

# 4. `request`

Type

```ts
APIRequestContext

```

Example

```ts
test('API', async ({ request }) => {

});

```

----------

API Call

```ts
const response = await request.get('/users');

```

----------

Useful for

-   Login
    
-   Database setup
    
-   Test data creation
    
-   Cleanup
    
-   API validation
    

----------

Example

```text
Create Customer

↓

API

↓

Open UI

↓

Validate Customer

```

Much faster than creating everything through the UI.

----------

# UI + API Together

```ts
test('Checkout', async ({

page,

request

}) => {

});

```

Common enterprise flow:

```text
API

↓

Create Test Data

↓

UI

↓

Verify

```

----------

# 5. `browserName`

Type

```ts
string

```

Example

```ts
test('Example', async ({ browserName }) => {

});

```

Output

```text
chromium

```

or

```text
firefox

```

or

```text
webkit

```

----------

Conditional Logic

```ts
if(browserName === 'firefox'){

// Skip

}

```

Useful for browser-specific behavior, though it should be used sparingly.

----------

# 6. `playwright`

Type

```ts
Playwright

```

Example

```ts
test('Example', async ({ playwright }) => {

});

```

Gives access to:

```text
chromium

firefox

webkit

devices

```

----------

Example

```ts
const browser = await playwright.chromium.launch();

```

Rarely used.

Mostly useful for advanced scenarios.

----------

# Fixture Dependency Tree

```text
playwright

↓

browser

↓

context

↓

page

```

Every custom fixture eventually depends on these.

----------

# Lazy Initialization

Suppose

```ts
test('API', async ({ request }) => {

});

```

Playwright creates:

```text
APIRequestContext

```

Only.

No browser.

----------

Another

```ts
test('UI', async ({ page }) => {

});

```

Creates

```text
Browser

↓

Context

↓

Page

```

Only what is required.

----------

# Multiple Built-in Fixtures

```ts
test('Example', async ({

page,

request,

browserName

}) => {

});

```

Playwright resolves them automatically.

----------

# Enterprise Example

```ts
test('Place Order', async ({

page,

request,

browserName

}) => {

  // Create order through API

  // Open UI

  // Verify UI

});

```

Very common.

----------

# Selenium vs Playwright

Selenium

```java
WebDriver driver

↓

ChromeDriver

↓

Browser

```

Everything uses the same driver.

----------

Playwright

```text
Browser

↓

Context

↓

Page

```

Better isolation.

----------

# Browser Context Is the Secret

Instead of launching:

```text
Chrome

Chrome

Chrome

Chrome

```

Playwright launches:

```text
Chrome

↓

Context

↓

Context

↓

Context

```

Much faster.

----------

# Common Mistakes

## ❌ Using Browser Instead of Context

Bad

```ts
await browser.newPage();

```

Better

```ts
await context.newPage();

```

Unless you intentionally need another isolated browser context.

----------

## ❌ Launching Browser Yourself

Bad

```ts
chromium.launch()

```

inside tests.

Use the injected fixtures instead.

----------

## ❌ Sharing Page

Never store:

```ts
global.page

```

Each test should receive its own page fixture.

----------

## ❌ Using Request Through Page

Don't call APIs via browser scripts if you can use:

```ts
request

```

It is faster and cleaner.

----------

# Which Fixture Should You Use?

Need

Fixture

UI Automation

`page`

Multiple Tabs

`context`

Multiple Users

`browser`

API Testing

`request`

Browser-Specific Logic

`browserName`

Advanced Browser Control

`playwright`

----------

# Interview Questions

### Q1. What is the difference between `page` and `context`?

-   `page` represents a browser tab.
    
-   `context` represents an isolated browser session that can contain one or more pages.
    

----------

### Q2. Why is `browser` worker-scoped?

Launching a browser is expensive.

Playwright reuses the browser within a worker to improve performance.

----------

### Q3. Can one context contain multiple pages?

Yes.

```ts
const secondPage = await context.newPage();

```

----------

### Q4. Can one browser contain multiple contexts?

Yes.

This is the foundation of Playwright's test isolation.

----------

### Q5. Which fixture is used for API testing?

```text
request

```

----------

### Q6. Why doesn't Playwright launch a browser for API-only tests?

Because of **lazy initialization**.

Only requested fixtures are created.

----------

### Q7. Which fixture is used most often?

```text
page

```

----------

# Best Practices

-   Prefer `page` for standard UI interactions.
    
-   Use `context` when you need additional tabs or browser-level state.
    
-   Use `browser` only for advanced scenarios like multi-user workflows.
    
-   Use `request` to create or clean up test data instead of driving the UI.
    
-   Let Playwright inject built-in fixtures; avoid launching browsers manually inside tests.
    
-   Take advantage of lazy initialization by requesting only the fixtures your test actually needs.
    

----------

# Enterprise Usage Pattern

```text
                Test
                  │
     ┌────────────┼────────────┐
     │            │            │
    page       request    browserName
     │            │
     ▼            ▼
  context     APIRequestContext
     │
     ▼
  browser
     │
     ▼
 playwright

```

Every custom fixture you'll build later—such as `loginPage`, `checkoutPage`, `customerApi`, or `database`—will ultimately be composed from these built-in fixtures.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIzNDcxMDk5Nl19
-->
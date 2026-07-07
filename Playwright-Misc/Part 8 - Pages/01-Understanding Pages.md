The next logical section is **Pages**.

Many beginners think a **Page** is just a browser tab. While that's true at a high level, understanding how **Browser → BrowserContext → Page** works is essential for writing scalable Playwright tests and is one of the most common Playwright interview topics.

----------

# Part 8 – Pages

# Chapter 1 – Understanding Pages

----------

# Introduction

Every Playwright test interacts with a **Page**.

Whenever you write:

```typescript
await page.goto('https://example.com');

```

or

```typescript
await page.locator('#username').fill('admin');

```

you are interacting with a **Page** object.

A **Page** represents a **single browser tab or window**.

----------

# What is a Page?

Think of a browser like this:

```text
Browser
│
├── Tab 1 (Page)
├── Tab 2 (Page)
├── Tab 3 (Page)
└── Tab 4 (Page)

```

Each browser tab is represented by one **Page** object.

Example

```typescript
const page = await browser.newPage();

```

Now `page` represents one browser tab.

----------

# Browser → BrowserContext → Page

One of Playwright's biggest strengths is the **BrowserContext**.

The hierarchy looks like:

```text
Browser
    │
    ├── BrowserContext
    │      │
    │      ├── Page
    │      ├── Page
    │      └── Page
    │
    └── BrowserContext
           │
           ├── Page
           └── Page

```

This hierarchy is extremely important.

----------

# Browser

Represents the browser process.

Example

```typescript
const browser = await chromium.launch();

```

There is usually **one browser instance**.

----------

# BrowserContext

Represents an isolated browser session.

Each context has its own:

-   Cookies
    
-   Local Storage
    
-   Session Storage
    
-   Permissions
    
-   Cache
    

Example

```typescript
const context =
await browser.newContext();

```

----------

# Page

Represents a browser tab.

Example

```typescript
const page =
await context.newPage();

```

----------

# Why BrowserContext Exists

Suppose two users log in simultaneously.

Without BrowserContext:

```text
Chrome

↓

User A Login

↓

User B Login

↓

Cookies Shared

```

Problem:

Both users share the same session.

----------

With BrowserContext:

```text
Browser

↓

Context A

↓

User A

----------------

Context B

↓

User B

```

Sessions remain completely isolated.

----------

# Creating a New Page

## Using Playwright Test

Most tests receive `page` automatically.

```typescript
test('Login', async ({ page }) => {

    await page.goto('/login');

});

```

No setup required.

----------

## Using BrowserContext

```typescript
const browser =
await chromium.launch();

const context =
await browser.newContext();

const page =
await context.newPage();

```

----------

# Opening a Website

```typescript
await page.goto(
'https://playwright.dev'
);

```

----------

Another example

```typescript
await page.goto(
'https://github.com'
);

```

Playwright waits for navigation automatically.

----------

# Getting the Current URL

```typescript
console.log(
page.url()
);

```

----------

Validation

```typescript
await expect(page)
.toHaveURL(
/dashboard/
);

```

Preferred over

```typescript
expect(
page.url()
)
.toContain(
'dashboard'
);

```

because the assertion automatically retries.

----------

# Getting the Page Title

```typescript
const title =
await page.title();

console.log(title);

```

----------

Better validation

```typescript
await expect(page)
.toHaveTitle(
'Dashboard'
);

```

----------

# Reloading the Page

```typescript
await page.reload();

```

----------

Real Example

```typescript
await page.reload();

await expect(
page.getByText(
'Welcome'
)
).toBeVisible();

```

----------

# Going Back

Equivalent to browser Back.

```typescript
await page.goBack();

```

----------

Example

```typescript
await page.goto('/products');

await page.goto('/cart');

await page.goBack();

await expect(page)
.toHaveURL(
/products/
);

```

----------

# Going Forward

```typescript
await page.goForward();

```

Equivalent to browser Forward.

----------

# Refresh vs Reload

Many people ask this in interviews.

There is no separate refresh API.

Browser refresh is:

```typescript
await page.reload();

```

----------

# Closing a Page

```typescript
await page.close();

```

The browser remains open.

Only the page closes.

----------

# Closing Browser vs Closing Page

```typescript
await page.close();

```

↓

```text
Tab Closed

```

----------

```typescript
await browser.close();

```

↓

```text
Entire Browser Closed

```

----------

# Waiting for Navigation

Example

```typescript
await page.goto('/login');

await page.getByRole('button',{
name:'Login'
})
.click();

await expect(page)
.toHaveURL(/dashboard/);

```

Playwright automatically waits for navigation when appropriate, and `toHaveURL()` retries until the expected URL is reached.

----------

# Screenshot of Page

```typescript
await page.screenshot({
path:'page.png'
});

```

----------

Full Page Screenshot

```typescript
await page.screenshot({

path:'full.png',

fullPage:true

});

```

----------

# PDF Generation

Chromium only.

```typescript
await page.pdf({

path:'report.pdf'

});

```

Useful for

-   Reports
    
-   Invoices
    
-   Statements
    

----------

# Viewport Size

```typescript
await page.setViewportSize({

width:1280,

height:720

});

```

Useful for responsive testing.

----------

# Getting Viewport

```typescript
page.viewportSize();

```

----------

# Bring Page to Front

Useful with multiple tabs.

```typescript
await page.bringToFront();

```

----------

# Example

Suppose

Tab A

Tab B

Tab C

```typescript
await page.bringToFront();

```

Now

Tab C becomes active.

----------

# Real-world Example

```typescript
test('Login Test', async ({ page }) => {

await page.goto('/login');

await page.getByLabel('Username')
.fill('admin');

await page.getByLabel('Password')
.fill('password');

await page.getByRole('button',{
name:'Login'
})
.click();

await expect(page)
.toHaveURL(/dashboard/);

await expect(
page.getByRole('heading',{
name:'Dashboard'
})
).toBeVisible();

});

```

Everything happens inside one Page.

----------

# Common Page APIs

API

Purpose

`goto()`

Navigate

`reload()`

Refresh page

`goBack()`

Browser Back

`goForward()`

Browser Forward

`url()`

Current URL

`title()`

Page title

`close()`

Close page

`bringToFront()`

Activate tab

`screenshot()`

Capture screenshot

`pdf()`

Generate PDF

`setViewportSize()`

Resize viewport

----------

# Best Practices

✅ Use the `page` fixture provided by Playwright Test whenever possible.

✅ Prefer `expect(page).toHaveURL()` over checking `page.url()` directly.

✅ Prefer `expect(page).toHaveTitle()` over reading the title manually.

✅ Close pages you create manually to avoid resource leaks.

✅ Create new `BrowserContext` objects—not just new pages—when simulating different users.

----------

# Common Mistakes

## ❌ Confusing Browser with Page

```text
Browser ≠ Page

```

Browser

↓

Many Pages

----------

## ❌ Using Multiple Pages Instead of Multiple Contexts

Bad

```text
Page A

Page B

Same Login Session

```

Better

```text
Context A

↓

Page

----------------

Context B

↓

Page

```

----------

## ❌ Manual URL Validation

Bad

```typescript
expect(
page.url()
).toContain(
'dashboard'
);

```

Better

```typescript
await expect(page)
.toHaveURL(/dashboard/);

```

----------

# Interview Questions

### Q1. What is a Page in Playwright?

A Page represents a single browser tab or window where your test interacts with web content.

----------

### Q2. What is the relationship between Browser, BrowserContext, and Page?

```
Browser
    ↓
BrowserContext
    ↓
Page

```

A browser can contain multiple contexts, and each context can contain multiple pages.

----------

### Q3. Why should multiple users use different BrowserContexts instead of different Pages?

Pages within the same BrowserContext share cookies, storage, and session data. Separate BrowserContexts provide isolated sessions, allowing tests to simulate independent users.

----------

### Q4. What is the difference between `page.reload()` and `page.goto()`?

-   `page.goto()` navigates to a specified URL.
    
-   `page.reload()` refreshes the current page.
    

----------

### Q5. How do you verify that navigation has completed?

Prefer:

```typescript
await expect(page).toHaveURL(/dashboard/);

```

This automatically retries until the expected URL is reached.

----------

# Summary

The `Page` object is the primary interface for interacting with a browser tab in Playwright. Understanding the hierarchy of **Browser → BrowserContext → Page** is fundamental to building scalable automation. While most tests use a single page provided by the Playwright Test runner, knowing how to create, manage, and navigate pages manually becomes essential for advanced scenarios such as multi-tab workflows and multi-user testing.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAwNzE3Mjk2Nl19
-->
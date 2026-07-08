This chapter covers one of the most common real-world automation scenarios. Almost every enterprise application opens additional tabs or windows for workflows such as:

-   OAuth authentication (Google, Microsoft, GitHub)
    
-   Payment gateways
    
-   PDF previews
    
-   Reports
    
-   External documentation
    
-   Terms & Conditions
    
-   Help Center
    
-   Print Preview
    

Many beginners struggle with multi-page automation because they treat tabs like Selenium window handles. Playwright provides a much cleaner model.

----------

# Chapter: Pages (Part 2) – Multiple Pages & Popups

----------

# Introduction

A browser rarely contains just one page.

Example:

```text
Browser
│
├── Dashboard
├── Reports
├── Payment
└── Documentation

```

Each tab is represented by a separate **Page** object.

Playwright makes switching between them straightforward.

----------

# Creating Multiple Pages

A `BrowserContext` can contain multiple pages.

```typescript
const page1 = await context.newPage();

const page2 = await context.newPage();

```

Now:

```text
BrowserContext

↓

Page 1

↓

Page 2

```

Both pages:

-   Share cookies
    
-   Share Local Storage
    
-   Share Session Storage
    

because they belong to the same BrowserContext.

----------

# Example

```typescript
const browser = await chromium.launch();

const context = await browser.newContext();

const home = await context.newPage();

const reports = await context.newPage();

await home.goto('/');

await reports.goto('/reports');

```

Each page works independently.

----------

# Accessing All Pages

Playwright provides:

```typescript
context.pages()

```

Example

```typescript
const pages = context.pages();

console.log(pages.length);

```

Suppose:

```text
Dashboard

Reports

Orders

```

Output

```text
3

```

----------

# Access Individual Pages

```typescript
const first = context.pages()[0];

const second = context.pages()[1];

```

Useful when multiple tabs already exist.

----------

# Switching Between Pages

Unlike Selenium,

there is **no window handle switching**.

Simply interact with the Page object.

Example

```typescript
await reports.bringToFront();

await reports.getByRole('button', {
    name: 'Generate'
}).click();

```

Then

```typescript
await home.bringToFront();

```

Very clean.

----------

# Opening a New Tab

Suppose clicking

```text
View Invoice

```

opens another tab.

The correct pattern is:

```typescript
const newPagePromise =
    context.waitForEvent('page');

await page.getByRole('link', {
    name: 'View Invoice'
}).click();

const invoicePage =
    await newPagePromise;

```

----------

## Why Register the Event First?

Wrong

```typescript
await page.click('#invoice');

const invoice =
await context.waitForEvent('page');

```

The tab may already have opened.

Correct

```typescript
const pagePromise =
context.waitForEvent('page');

await page.click('#invoice');

const newPage =
await pagePromise;

```

Always register the listener before triggering the event.

----------

# Waiting for the New Page to Load

After obtaining the page:

```typescript
await newPage.waitForLoadState();

```

Then interact normally.

```typescript
await expect(newPage)
.toHaveTitle(/Invoice/);

```

----------

# Example – Report Tab

```typescript
const reportPromise =
context.waitForEvent('page');

await page.getByRole('button', {
    name: 'Reports'
}).click();

const reportPage =
await reportPromise;

await reportPage.waitForLoadState();

await expect(reportPage)
.toHaveURL(/reports/);

```

----------

# Handling Popups

Many applications use

```javascript
window.open()

```

instead of opening a normal browser tab.

Playwright calls this a **popup**.

----------

## Waiting for Popup

```typescript
const popupPromise =
page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Help'
}).click();

const popup =
await popupPromise;

```

Notice

```typescript
page.waitForEvent('popup')

```

instead of

```typescript
context.waitForEvent('page')

```

----------

# Page vs Popup

Opens From

Wait API

Browser Context

`context.waitForEvent('page')`

Existing Page

`page.waitForEvent('popup')`

Both return a Page object.

----------

# Real-World Example – OAuth Login

Application

↓

Click

```text
Login with Google

```

↓

Google opens in another window.

Pattern

```typescript
const popupPromise =
page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Continue with Google'
}).click();

const googlePage =
await popupPromise;

await googlePage.waitForLoadState();

await googlePage
.getByLabel('Email')
.fill('user@gmail.com');

```

After authentication,

the popup closes,

and the original page updates.

----------

# Example – Microsoft Login

```typescript
const popupPromise =
page.waitForEvent('popup');

await page.getByRole('button', {
    name: 'Microsoft'
}).click();

const microsoft =
await popupPromise;

await microsoft.waitForLoadState();

await microsoft
.getByLabel('Email')
.fill('admin@test.com');

```

----------

# Example – Payment Gateway

Suppose

```text
Checkout

↓

Razorpay

↓

Payment Success

↓

Return to Application

```

```typescript
const paymentPromise =
page.waitForEvent('popup');

await page.click('#pay');

const payment =
await paymentPromise;

await payment.waitForLoadState();

await payment.click('#success');

```

----------

# Waiting for Popup to Close

```typescript
await popup.waitForEvent('close');

```

Useful after OAuth authentication.

----------

# Closing a Popup

```typescript
await popup.close();

```

Original page remains open.

----------

# Bringing a Page to Front

When several pages exist

```typescript
await reportPage.bringToFront();

```

Useful when visually debugging headed tests.

----------

# Getting Active Pages

```typescript
const pages =
context.pages();

for (const page of pages) {

    console.log(await page.title());

}

```

Useful for browser automation utilities.

----------

# Real-World Example – Documentation Link

```typescript
const docsPromise =
context.waitForEvent('page');

await page.getByText('Documentation')
.click();

const docs =
await docsPromise;

await docs.waitForLoadState();

await expect(docs)
.toHaveTitle(/Documentation/);

await docs.close();

await page.bringToFront();

```

----------

# Multi-User vs Multi-Page

Many engineers confuse these.

### Multiple Pages

```text
Context

↓

Page A

↓

Page B

```

Shared session.

----------

### Multiple Users

```text
Context A

↓

Page

------------

Context B

↓

Page

```

Different sessions.

Always use **different BrowserContexts** for independent users.

----------

# Common Multi-Page Patterns

## Pattern 1 – New Tab

```typescript
const pagePromise =
context.waitForEvent('page');

await page.click(...);

const newPage =
await pagePromise;

```

----------

## Pattern 2 – Popup

```typescript
const popupPromise =
page.waitForEvent('popup');

await page.click(...);

const popup =
await popupPromise;

```

----------

## Pattern 3 – OAuth

```typescript
Application

↓

Popup

↓

Login

↓

Popup Closes

↓

Application Updates

```

----------

# Best Practices

-   Register `waitForEvent()` **before** triggering the action.
    
-   Call `waitForLoadState()` before interacting with the new page.
    
-   Keep separate variables for each page to avoid confusion.
    
-   Close temporary pages when they're no longer needed.
    
-   Use separate BrowserContexts—not separate Pages—for multiple users.
    

----------

# Common Mistakes

### ❌ Waiting after clicking

```typescript
await page.click('#report');

await context.waitForEvent('page');

```

The event may already have occurred.

----------

### ❌ Treating pages like Selenium windows

Playwright doesn't require switching by window handle.

You already have a reference to the Page object.

----------

### ❌ Using multiple pages for different users

Bad

```text
Page A

↓

User A

--------

Page B

↓

User B

```

Cookies are shared.

----------

Better

```text
Context A

↓

User A

--------

Context B

↓

User B

```

----------

### ❌ Interacting before the page finishes loading

Always wait for the page to reach an appropriate load state or assert on a meaningful UI element before continuing.

----------

# Interview Questions

### Q1. What is the difference between `context.waitForEvent('page')` and `page.waitForEvent('popup')`?

-   `context.waitForEvent('page')` waits for a new page created within the BrowserContext.
    
-   `page.waitForEvent('popup')` waits for a popup opened by a specific page, typically through `window.open()`.
    

----------

### Q2. Why should `waitForEvent()` be registered before clicking?

Because the new page or popup may open immediately. Registering the wait first ensures the event isn't missed.

----------

### Q3. How do you switch between tabs in Playwright?

There is no window-handle API. Simply interact with the desired `Page` object. Use `bringToFront()` only if you need to make a tab active in a headed browser.

----------

### Q4. Can multiple pages share the same login session?

Yes. Pages within the same BrowserContext share cookies, storage, and authentication state.

----------

### Q5. How do you simulate two independent users?

Create two separate BrowserContexts:

```typescript
const adminContext = await browser.newContext();
const customerContext = await browser.newContext();

const adminPage = await adminContext.newPage();
const customerPage = await customerContext.newPage();

```

Each context has isolated cookies and storage.

----------

# Summary

Playwright treats every browser tab or window as a `Page` object, making multi-page automation much simpler than traditional window-handle management. By registering page or popup events before triggering user actions and understanding when to use `context.waitForEvent('page')` versus `page.waitForEvent('popup')`, you can automate complex workflows such as OAuth authentication, payment gateways, documentation links, and report generation with confidence.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMTU5MTk4XX0=
-->
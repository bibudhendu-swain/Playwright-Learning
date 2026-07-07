This is the **final chapter of the Auto Waiting section** and one of the most practical chapters in the entire handbook. A common misconception is that _"Playwright auto-waits, so I never need explicit waits."_ That's not true.

Auto-waiting handles **element readiness**, but there are legitimate scenarios where you should explicitly wait for navigation, network requests, events, or custom application conditions.

----------

# Chapter: Auto Waiting (Part 3) – Advanced Waiting APIs

----------

# Introduction

Playwright's auto-waiting eliminates most synchronization issues, but some situations require explicit waiting.

Examples include:

-   Waiting for navigation
    
-   Waiting for API requests
    
-   Waiting for a download
    
-   Waiting for a popup
    
-   Waiting for a WebSocket message
    
-   Waiting for a custom JavaScript condition
    

For these scenarios, Playwright provides dedicated waiting APIs.

----------

# When Should You Use Explicit Waits?

Use explicit waits when you're waiting for **application behavior**, not **element readiness**.

Auto Waiting

Explicit Waiting

Button becomes clickable

Wait for an API response

Input becomes editable

Wait for navigation

Checkbox becomes enabled

Wait for a popup

Element becomes visible

Wait for a download

----------

# waitForSelector()

## Purpose

Waits until a selector reaches a specific state.

Although this API is available, **locator assertions are generally preferred in modern Playwright code** because they are more expressive and integrate naturally with auto-retrying assertions.

----------

## Syntax

```typescript
await page.waitForSelector(selector);

```

----------

## Example

```typescript
await page.waitForSelector('.results');

await page.locator('.results').click();

```

----------

## Waiting for Different States

### Visible

```typescript
await page.waitForSelector('.message', {
    state: 'visible'
});

```

----------

### Hidden

```typescript
await page.waitForSelector('.spinner', {
    state: 'hidden'
});

```

----------

### Attached

```typescript
await page.waitForSelector('.card', {
    state: 'attached'
});

```

----------

### Detached

```typescript
await page.waitForSelector('.notification', {
    state: 'detached'
});

```

----------

## Recommended Alternative

Instead of

```typescript
await page.waitForSelector('.success');

```

Prefer

```typescript
await expect(page.locator('.success'))
    .toBeVisible();

```

The assertion is clearer and communicates the expected outcome.

----------

# waitForLoadState()

## Purpose

Waits for the page to reach a particular loading state.

----------

## Available States

State

Meaning

`load`

All page resources have loaded

`domcontentloaded`

HTML has been parsed

`networkidle`

No network activity for a short period (generally not recommended for testing)

----------

## Example

```typescript
await page.goto('/dashboard');

await page.waitForLoadState('load');

```

----------

## DOM Content Loaded

```typescript
await page.waitForLoadState(
    'domcontentloaded'
);

```

Useful when JavaScript files can continue loading in the background but the DOM is ready.

----------

## Network Idle

```typescript
await page.waitForLoadState(
    'networkidle'
);

```

> **Recommendation:** The Playwright team generally advises against relying on `networkidle` for test synchronization. Modern applications often make background requests, so it's usually better to wait for a specific UI change or network request.

----------

# waitForURL()

## Purpose

Waits until the page navigates to a specific URL.

----------

## Example

```typescript
await page.getByRole('button', {
    name: 'Login'
}).click();

await page.waitForURL(/dashboard/);

```

----------

## Exact URL

```typescript
await page.waitForURL(
    'https://example.com/dashboard'
);

```

----------

## Wildcards

```typescript
await page.waitForURL('**/dashboard');

```

----------

## Better Alternative

```typescript
await expect(page)
    .toHaveURL(/dashboard/);

```

This automatically retries and provides better assertion reporting.

----------

# waitForResponse()

## Purpose

Waits until a network response matching a condition is received.

----------

## Example

```typescript
const responsePromise = page.waitForResponse(
    response =>
        response.url().includes('/users') &&
        response.status() === 200
);

await page.getByRole('button', {
    name: 'Load Users'
}).click();

const response = await responsePromise;

```

----------

## Why Capture the Promise First?

Notice the sequence:

```typescript
const responsePromise = page.waitForResponse(...);

await page.click(...);

await responsePromise;

```

If you wait **after** the click, the request may already have completed and you could miss it.

----------

## Real-World Example

```typescript
const responsePromise = page.waitForResponse(
    '**/api/orders'
);

await page.getByRole('button', {
    name: 'Orders'
}).click();

const response = await responsePromise;

expect(response.status()).toBe(200);

```

----------

# waitForRequest()

## Purpose

Waits until a matching network request is sent.

----------

## Example

```typescript
const requestPromise = page.waitForRequest(
    '**/login'
);

await page.getByRole('button', {
    name: 'Login'
}).click();

const request = await requestPromise;

```

----------

## Use Cases

-   Verify payloads
    
-   Validate headers
    
-   Authentication testing
    
-   API monitoring
    

----------

# waitForFunction()

## Purpose

Waits until a JavaScript expression evaluates to `true`.

----------

## Example

```typescript
await page.waitForFunction(() => {
    return window.localStorage.getItem('token') !== null;
});

```

----------

## Another Example

```typescript
await page.waitForFunction(() => {
    return document.querySelectorAll('.product').length === 10;
});

```

----------

## Real-World Example

Waiting for a global application flag:

```typescript
await page.waitForFunction(() => {
    return window.appReady === true;
});

```

----------

# waitForEvent()

## Purpose

Waits for a Playwright event.

----------

## Popup Example

```typescript
const popupPromise = page.waitForEvent('popup');

await page.getByRole('link', {
    name: 'Open Report'
}).click();

const popup = await popupPromise;

```

----------

## Download Example

```typescript
const downloadPromise =
    page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Download'
}).click();

const download = await downloadPromise;

```

----------

## File Chooser Example

```typescript
const chooserPromise =
    page.waitForEvent('filechooser');

await page.getByText('Upload').click();

const chooser = await chooserPromise;

```

----------

# waitForTimeout()

## Purpose

Pauses execution for a fixed duration.

```typescript
await page.waitForTimeout(5000);

```

----------

## Why It Should Usually Be Avoided

Problems:

-   Always waits the full duration
    
-   Slows down test execution
    
-   Doesn't guarantee the application is ready
    
-   Can still fail if the application takes longer
    

----------

## Acceptable Uses

-   Debugging
    
-   Demonstrations
    
-   Temporary investigation
    

Avoid using it as a synchronization strategy in production tests.

----------

# Choosing the Right Waiting API

Scenario

Recommended API

Wait for an element

Locator assertion (`toBeVisible()`, etc.)

Wait for navigation

`expect(page).toHaveURL()` or `waitForURL()`

Wait for page loading

`waitForLoadState()`

Wait for API response

`waitForResponse()`

Wait for request

`waitForRequest()`

Wait for popup

`waitForEvent('popup')`

Wait for download

`waitForEvent('download')`

Wait for custom JavaScript condition

`waitForFunction()`

Pause for debugging

`waitForTimeout()`

----------

# Real-World Example – Login

```typescript
const loginResponse =
    page.waitForResponse('**/api/login');

await page.getByRole('button', {
    name: 'Login'
}).click();

await loginResponse;

await expect(page)
    .toHaveURL(/dashboard/);

await expect(
    page.getByRole('heading', {
        name: 'Dashboard'
    })
).toBeVisible();

```

This combines:

-   Network synchronization
    
-   Navigation verification
    
-   UI validation
    

----------

# Real-World Example – File Download

```typescript
const downloadPromise =
    page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Export'
}).click();

const download = await downloadPromise;

expect(await download.suggestedFilename())
    .toContain('.csv');

```

----------

# Common Synchronization Patterns

## Pattern 1 – Wait Before the Trigger

```typescript
const popupPromise =
    page.waitForEvent('popup');

await page.click('#open');

const popup = await popupPromise;

```

Always register the wait **before** performing the action.

----------

## Pattern 2 – UI-Driven Synchronization

```typescript
await page.click('#save');

await expect(
    page.getByText('Saved Successfully')
).toBeVisible();

```

Preferred over fixed delays.

----------

## Pattern 3 – Network + UI

```typescript
const response =
    page.waitForResponse('**/orders');

await page.click('#loadOrders');

await response;

await expect(
    page.locator('.order')
).toHaveCount(10);

```

This verifies both the backend and frontend behavior.

----------

# Best Practices

-   Prefer locator assertions over `waitForSelector()`.
    
-   Register `waitForResponse()`, `waitForRequest()`, and `waitForEvent()` **before** triggering the action.
    
-   Use `expect(page).toHaveURL()` when validating navigation.
    
-   Wait for business conditions rather than arbitrary time.
    
-   Reserve `waitForTimeout()` for debugging.
    

----------

# Common Mistakes

### ❌ Waiting after the action

```typescript
await page.click('#download');

await page.waitForEvent('download');

```

The download may already have started.

Correct:

```typescript
const downloadPromise =
    page.waitForEvent('download');

await page.click('#download');

await downloadPromise;

```

----------

### ❌ Using `networkidle` for everything

Modern SPAs frequently make background requests, so `networkidle` may never be reached or may not represent the business state you're interested in.

----------

### ❌ Waiting for selectors instead of business outcomes

```typescript
await page.waitForSelector('.message');

```

Better:

```typescript
await expect(
    page.getByText('Saved Successfully')
).toBeVisible();

```

----------

# Interview Questions

### Q1. Does Playwright eliminate the need for explicit waits?

No. Auto-waiting covers element readiness, but explicit waits are still appropriate for navigation, network requests, downloads, popups, and custom application conditions.

----------

### Q2. Why should `waitForResponse()` be registered before clicking?

Because the request may complete very quickly. Registering the wait first ensures the event isn't missed.

----------

### Q3. Why is `waitForTimeout()` discouraged?

It introduces fixed delays, slows tests, and doesn't guarantee that the application has reached the expected state.

----------

### Q4. When should `waitForFunction()` be used?

When waiting for a custom JavaScript condition that Playwright cannot infer automatically, such as a global application flag or a value in local storage.

----------

### Q5. What is the preferred way to wait for navigation?

When you're validating navigation as part of a test outcome, prefer:

```typescript
await expect(page).toHaveURL(/dashboard/);

```

Use `waitForURL()` when you need to synchronize with navigation but are not necessarily making an assertion about the final application state.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUyOTI2MDczNF19
-->
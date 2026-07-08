This chapter moves beyond the assertions most engineers use every day and into the features that make Playwright's assertion system powerful for enterprise test suites.

----------

# Chapter: Assertions (Part 3) – Advanced Assertions

----------

# Introduction

So far, we've learned how to verify UI elements using Playwright's built-in assertions.

However, enterprise projects often require more sophisticated validation strategies, such as:

-   Ignoring some failures while continuing the test
    
-   Verifying that something **does not** happen
    
-   Waiting for APIs or databases to reach an expected state
    
-   Creating reusable custom assertions
    
-   Adding meaningful failure messages
    

Playwright provides a rich assertion API to address these scenarios.

In this chapter, we'll cover:

-   Negating Matchers (`.not`)
    
-   Asymmetric Matchers
    
-   Soft Assertions
    
-   Custom Expect Messages
    
-   `expect.configure()`
    
-   `expect.poll()`
    
-   `expect.toPass()`
    
-   Custom Matchers (`expect.extend()`)
    
-   Combining Matchers
    
-   Compatibility with the Jest `expect` library
    

----------

# Negating Matchers (`.not`)

## What is `.not`?

The `.not` modifier verifies that a condition **is not true**.

Instead of checking:

> "Element is visible"

You can verify:

> "Element is **not** visible"

----------

## Syntax

```typescript
await expect(locator).not.toBeVisible();

```

----------

## Example – Hidden Success Message

```typescript
await expect(
    page.getByText('Order Successful')
).not.toBeVisible();

```

Playwright waits until the element is **not visible**.

----------

## Example – Disabled Button

```typescript
await expect(
    page.getByRole('button', {
        name: 'Submit'
    })
).not.toBeEnabled();

```

Equivalent to:

```typescript
await expect(
    page.getByRole('button', {
        name: 'Submit'
    })
).toBeDisabled();

```

Prefer the more expressive matcher (`toBeDisabled()`) when available.

----------

## Example – URL Validation

```typescript
await expect(page)
    .not.toHaveURL(/login/);

```

Verifies that the application has navigated away from the login page.

----------

## Best Practices

Use `.not` only when there isn't a more expressive positive matcher.

Good:

```typescript
await expect(page).not.toHaveURL(/login/);

```

Less expressive:

```typescript
await expect(locator).not.toBeEnabled();

```

Better:

```typescript
await expect(locator).toBeDisabled();

```

----------

# Asymmetric Matchers

## What Are Asymmetric Matchers?

Sometimes you don't want to compare an entire object or array.

Instead, you want to verify only specific parts.

Playwright supports the same asymmetric matchers available through the underlying `expect` implementation.

----------

## expect.any()

Matches any value of a particular type.

Example:

```typescript
expect({
    id: 10,
    name: 'John'
}).toEqual({
    id: expect.any(Number),
    name: expect.any(String)
});

```

Useful when IDs are generated dynamically.

----------

## expect.anything()

Matches any value except `null` and `undefined`.

```typescript
expect({
    token: 'abc123'
}).toEqual({
    token: expect.anything()
});

```

Useful for session IDs, tokens, timestamps, etc.

----------

## expect.objectContaining()

Matches part of an object.

```typescript
const user = {
    id: 100,
    name: 'Alice',
    email: 'alice@example.com',
    role: 'Admin'
};

expect(user).toEqual(
    expect.objectContaining({
        name: 'Alice',
        role: 'Admin'
    })
);

```

The other properties are ignored.

----------

## expect.arrayContaining()

Matches part of an array.

```typescript
expect([
    'Java',
    'TypeScript',
    'Playwright'
]).toEqual(
    expect.arrayContaining([
        'TypeScript'
    ])
);

```

The array may contain additional values.

----------

## Real-World API Example

```typescript
const response = await request.get('/users');
const body = await response.json();

expect(body).toEqual(
    expect.objectContaining({
        status: 'SUCCESS',
        userId: expect.any(Number)
    })
);

```

Perfect for API testing where only a subset of fields matters.

----------

# Soft Assertions

## What Are Soft Assertions?

Normally, when an assertion fails:

```typescript
await expect(locator).toBeVisible();

```

the test stops immediately.

A **soft assertion** records the failure but allows the test to continue.

----------

## Syntax

```typescript
await expect.soft(locator).toBeVisible();

```

----------

## Example

```typescript
await expect.soft(page.getByText('Welcome'))
    .toBeVisible();

await expect.soft(page.getByText('Dashboard'))
    .toBeVisible();

await expect.soft(page.getByText('Profile'))
    .toBeVisible();

```

If the first assertion fails, Playwright still evaluates the remaining assertions.

At the end of the test, all soft assertion failures are reported together.

----------

## Real-World Use Case

Dashboard validation:

```typescript
await expect.soft(page.getByText('Revenue')).toBeVisible();
await expect.soft(page.getByText('Orders')).toBeVisible();
await expect.soft(page.getByText('Customers')).toBeVisible();
await expect.soft(page.getByText('Profit')).toBeVisible();

```

You receive a complete picture of what failed instead of stopping at the first missing widget.

----------

## When to Use

Good for:

-   Dashboard verification
    
-   Report validation
    
-   UI smoke testing
    
-   Accessibility audits
    

Avoid using soft assertions for critical workflow steps such as login or payment confirmation.

----------

# Custom Expect Messages

Assertion failures are easier to understand when they include business context.

----------

## Syntax

```typescript
await expect(
    page.getByRole('button', { name: 'Login' }),
    'Login button should be visible before user authentication'
).toBeVisible();

```

If the assertion fails, the custom message appears in the test report.

----------

## Real-World Example

```typescript
await expect(
    page.getByText('Payment Successful'),
    'Payment confirmation should appear after successful checkout'
).toBeVisible();

```

These messages make debugging much easier, especially in large test suites.

----------

# expect.configure()

## Purpose

Creates a customized `expect` instance with predefined options.

----------

## Example

```typescript
const slowExpect = expect.configure({
    timeout: 15000
});

```

Now:

```typescript
await slowExpect(locator).toBeVisible();

```

always waits up to 15 seconds.

----------

## Another Example

```typescript
const softExpect = expect.configure({
    soft: true
});

```

```typescript
await softExpect(page.getByText('Dashboard'))
    .toBeVisible();

```

Equivalent to:

```typescript
await expect.soft(
    page.getByText('Dashboard')
).toBeVisible();

```

----------

## Why Use It?

Very useful when an entire test or helper function requires consistent assertion behavior.

----------

# expect.poll()

## What Is `expect.poll()`?

Sometimes the value you're testing isn't a locator.

For example:

-   API status
    
-   Database state
    
-   Queue length
    
-   Cache value
    
-   File contents
    

`expect.poll()` repeatedly executes a function until the expectation succeeds or times out.

----------

## Example

```typescript
await expect.poll(async () => {
    const response = await request.get('/status');
    return response.status();
}).toBe(200);

```

Playwright continues polling until the status becomes `200`.

----------

## Real-World Example

Wait for background processing:

```typescript
await expect.poll(async () => {
    const response = await request.get('/job/123');
    const body = await response.json();
    return body.status;
}).toBe('Completed');

```

No manual retry loop required.

----------

# expect.toPass()

## Purpose

Retries an entire block of code until it passes.

Unlike `expect.poll()`, which retries a single function result, `expect.toPass()` retries everything inside the callback.

----------

## Example

```typescript
await expect(async () => {
    await expect(page.getByText('Completed'))
        .toBeVisible();

    await expect(page.getByText('Invoice Generated'))
        .toBeVisible();
}).toPass();

```

If either assertion fails, the entire callback is retried until both succeed or the timeout expires.

----------

## Real-World Example

After clicking a button, several UI components update asynchronously.

```typescript
await page.getByRole('button', {
    name: 'Refresh'
}).click();

await expect(async () => {
    await expect(page.locator('.status'))
        .toHaveText('Ready');

    await expect(page.locator('.progress'))
        .toHaveText('100%');
}).toPass();

```

----------

# Custom Matchers with `expect.extend()`

Large projects often repeat the same assertions.

Instead of duplicating code, create reusable matchers.

----------

## Example

```typescript
import { expect } from '@playwright/test';

expect.extend({
    async toBeSuccessful(response) {
        const pass = response.status() >= 200 &&
                     response.status() < 300;

        return {
            pass,
            message: () =>
                `Expected successful response`
        };
    }
});

```

Usage:

```typescript
await expect(response).toBeSuccessful();

```

----------

## Another Example

```typescript
await expect(page).toHaveSuccessfulLogin();

```

The matcher hides the implementation details and makes tests more readable.

----------

# Compatibility with the Jest `expect` Library

Playwright's `expect` supports many familiar matchers from the Jest ecosystem.

Examples include:

```typescript
expect(value).toBe(10);

expect(array).toContain('Playwright');

expect(object).toEqual({
    name: 'Alice'
});

expect(value).toBeTruthy();

```

In addition, Playwright extends `expect` with browser-aware assertions such as:

```typescript
await expect(locator).toBeVisible();

await expect(page).toHaveURL(/dashboard/);

await expect(locator).toHaveText('Success');

```

----------

# Combining Custom Matchers from Multiple Modules

Large organizations often organize custom matchers by domain.

Example:

```
matchers/
    apiMatchers.ts
    uiMatchers.ts
    databaseMatchers.ts

```

Each module exports its own `expect.extend()` implementation.

Your test setup imports them so that all custom matchers are available across the project.

This keeps assertion logic reusable and easy to maintain.

----------

# Best Practices

-   Use `.not` only when it clearly expresses the intent.
    
-   Use soft assertions for independent validations, not critical workflow steps.
    
-   Add custom messages for assertions that represent business rules.
    
-   Use `expect.poll()` for external systems such as APIs or databases.
    
-   Use `expect.toPass()` when multiple assertions must eventually succeed together.
    
-   Encapsulate repeated validation logic in custom matchers.
    

----------

# Common Mistakes

### ❌ Using `waitForTimeout()` before polling

```typescript
await page.waitForTimeout(5000);

expect(status).toBe('Completed');

```

Better:

```typescript
await expect.poll(async () => status()).toBe('Completed');

```

----------

### ❌ Using soft assertions for critical flows

```typescript
await expect.soft(page).toHaveURL(/dashboard/);

```

If login fails, continuing the test usually produces misleading failures. Use regular assertions for critical checkpoints.

----------

### ❌ Creating overly specific custom matchers

Avoid matchers that are tightly coupled to a single page or test. Keep them generic enough to be reused.

----------

# Interview Questions

### Q1. What is the difference between `expect.poll()` and `expect.toPass()`?

-   `expect.poll()` repeatedly evaluates the return value of a function until it matches the expectation.
    
-   `expect.toPass()` retries an entire block of code, including multiple assertions or actions.
    

----------

### Q2. When should soft assertions be used?

When multiple independent validations should all run, even if some fail—for example, validating every widget on a dashboard.

----------

### Q3. Why use `expect.configure()`?

To create customized `expect` instances with predefined settings such as longer timeouts or soft assertion behavior.

----------

### Q4. What are asymmetric matchers useful for?

They allow partial matching of objects and arrays and validation of values with dynamic types, making API tests more flexible.

----------

### Q5. Why create custom matchers?

Custom matchers encapsulate repeated assertion logic, improve readability, and promote consistency across large test suites.

----------

# Summary

Advanced assertions make Playwright suitable for complex enterprise automation. Features like **soft assertions**, **custom expect messages**, **`expect.poll()`**, **`expect.toPass()`**, and **custom matchers** help you write tests that are expressive, maintainable, and resilient to asynchronous behavior. Rather than relying on manual retry loops or duplicated validation logic, these APIs let you model business expectations directly in your tests.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTUwMTI3MzE3N119
-->
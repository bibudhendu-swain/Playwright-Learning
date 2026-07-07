Assertions are one of the **core pillars of Playwright**. A test without assertions only performs actions—it doesn't verify whether the application behaves correctly.

----------

# Chapter: Assertions (Part 1) – Assertion Fundamentals

----------

# Introduction

An **assertion** verifies that the application is in the expected state.

Consider the following test:

```typescript
await page.getByRole('button', { name: 'Login' }).click();

```

The click succeeds, but did the login actually work?

Without an assertion, the test will pass even if:

-   The user remains on the login page.
    
-   An error message appears.
    
-   The dashboard never loads.
    

Adding an assertion verifies the expected outcome.

```typescript
await page.getByRole('button', { name: 'Login' }).click();

await expect(page).toHaveURL(/dashboard/);

```

Now the test checks that the user actually reached the dashboard.

----------

# Why Are Assertions Important?

Assertions answer the question:

> **"Did the application behave as expected?"**

Without assertions, automation only performs user actions.

With assertions, automation validates business functionality.

Example:

```typescript
await page.getByRole('button', {
    name: 'Add to Cart'
}).click();

await expect(page.getByText('Item Added'))
    .toBeVisible();

```

The click alone doesn't prove success.

The assertion does.

----------

# How Playwright Assertions Differ from Traditional Assertions

One of Playwright's biggest advantages is **auto-retrying assertions**.

Traditional assertions (common in many testing frameworks):

```typescript
expect(value).toBe(expected);

```

This evaluates immediately.

If the value isn't ready yet, the assertion fails.

Playwright locator assertions behave differently.

Example:

```typescript
await expect(page.getByText('Welcome'))
    .toBeVisible();

```

If "Welcome" appears after 3 seconds, Playwright waits automatically.

This significantly reduces flaky tests.

----------

# The `expect()` Function

All Playwright assertions begin with:

```typescript
expect(...)

```

General syntax:

```typescript
await expect(locator).matcher();

```

or

```typescript
expect(value).matcher();

```

Examples:

```typescript
await expect(page).toHaveTitle('Home');

await expect(locator).toBeVisible();

expect(total).toBe(5);

```

----------

# Types of Assertions

Playwright supports three broad categories:

Type

Auto Retry

Example

Locator Assertions

✅

`toBeVisible()`

Page Assertions

✅

`toHaveURL()`

Generic Value Assertions

❌

`toBe()`

Understanding this difference is extremely important.

----------

# Auto-Retrying Assertions

These assertions automatically retry until:

-   The assertion passes
    
-   The timeout is reached
    

Example:

```typescript
await expect(
    page.getByText('Order Successful')
).toBeVisible();

```

Internally:

```
Try Assertion
      ↓
Fail
      ↓
Wait
      ↓
Retry
      ↓
Pass

```

This continues until the timeout expires.

----------

## Example

Suppose a spinner disappears after an API call.

```typescript
await page.getByRole('button', {
    name: 'Submit'
}).click();

await expect(page.getByText('Success'))
    .toBeVisible();

```

No explicit wait is needed.

----------

## Another Example

```typescript
await expect(page.locator('.price'))
    .toContainText('$100');

```

If the UI updates after 2 seconds, Playwright waits automatically.

----------

# Locator Assertions

These are the most commonly used assertions.

Examples include:

```typescript
await expect(locator).toBeVisible();

await expect(locator).toBeHidden();

await expect(locator).toBeEnabled();

await expect(locator).toBeDisabled();

await expect(locator).toHaveText('Success');

await expect(locator).toContainText('Order');

await expect(locator).toHaveValue('Admin');

await expect(locator).toHaveCount(5);

```

These assertions automatically retry.

We'll explore each matcher in detail in the next chapter.

----------

# Page Assertions

Playwright also supports assertions on the page itself.

Examples:

```typescript
await expect(page)
    .toHaveTitle('Dashboard');

```

```typescript
await expect(page)
    .toHaveURL(/dashboard/);

```

Example:

```typescript
await page.getByRole('button', {
    name: 'Login'
}).click();

await expect(page)
    .toHaveURL(/dashboard/);

```

----------

# Generic Value Assertions

These work like standard testing framework assertions.

Example:

```typescript
const total = 5;

expect(total).toBe(5);

```

String example:

```typescript
expect('Playwright')
    .toContain('wright');

```

Array example:

```typescript
expect(['Java', 'TypeScript'])
    .toContain('Java');

```

These assertions **do not retry** because they operate on values already available in memory.

----------

# Locator Assertions vs Generic Assertions

Suppose you want to verify an input value.

❌ Less appropriate:

```typescript
const value = await page.locator('#username').inputValue();

expect(value).toBe('admin');

```

This checks the value only once.

✔ Better:

```typescript
await expect(page.locator('#username'))
    .toHaveValue('admin');

```

Playwright waits until the input reaches the expected value.

----------

# Assertion Timeout

Assertions have their own timeout.

Example:

```typescript
await expect(locator)
    .toBeVisible({
        timeout: 10000
    });

```

Playwright waits up to 10 seconds.

If the assertion still fails:

```
Timeout Error

```

----------

# Common Assertion Flow

Typical UI flow:

```typescript
await page.goto('/login');

await page.getByLabel('Username')
    .fill('admin');

await page.getByLabel('Password')
    .fill('password');

await page.getByRole('button', {
    name: 'Login'
}).click();

await expect(page)
    .toHaveURL(/dashboard/);

await expect(page.getByRole('heading'))
    .toContainText('Dashboard');

```

Notice there are **no manual waits**.

Assertions handle synchronization.

----------

# Assertions in API Testing

Assertions aren't limited to UI tests.

Example:

```typescript
const response = await request.get('/users');

expect(response.status()).toBe(200);

```

Response body:

```typescript
const body = await response.json();

expect(body.name).toBe('John');

```

These are generic assertions because the data is already available.

----------

# Assertions in Collections

```typescript
await expect(page.locator('.product'))
    .toHaveCount(10);

```

Playwright waits until exactly ten products are displayed.

----------

# Assertions with Tables

Example:

```typescript
await expect(
    page.locator('table tr')
)
.toHaveCount(6);

```

This validates:

-   Rows loaded
    
-   Table rendered correctly
    

----------

# Assertions with Dynamic Content

Example:

```typescript
await page.getByRole('button', {
    name: 'Load Results'
}).click();

await expect(
    page.locator('.result')
)
.toHaveCount(20);

```

Playwright retries until all results appear.

----------

# Assertions vs Explicit Waits

❌ Unnecessary:

```typescript
await page.waitForTimeout(5000);

expect(await page.locator('.message').textContent())
    .toBe('Saved');

```

✔ Better:

```typescript
await expect(page.locator('.message'))
    .toHaveText('Saved');

```

The second approach is faster and more reliable.

----------

# Real-World Example – Login Validation

```typescript
test('Successful Login', async ({ page }) => {

    await page.goto('/login');

    await page.getByLabel('Username')
        .fill('admin');

    await page.getByLabel('Password')
        .fill('password');

    await page.getByRole('button', {
        name: 'Login'
    }).click();

    await expect(page)
        .toHaveURL(/dashboard/);

    await expect(
        page.getByRole('heading', {
            name: 'Dashboard'
        })
    ).toBeVisible();

});

```

This test verifies both navigation and page content, making it much more robust than checking only one condition.

----------

# Best Practices

-   Prefer Playwright locator assertions over generic value assertions for UI verification.
    
-   Let Playwright handle synchronization through auto-retrying assertions.
    
-   Use page assertions (`toHaveURL()`, `toHaveTitle()`) to verify navigation.
    
-   Use meaningful assertions that reflect business outcomes, not just technical implementation.
    
-   Combine actions with assertions to clearly express the expected behavior.
    
-   Override assertion timeouts only when a particular operation is known to take longer than the default.
    

----------

# Common Mistakes

### ❌ Using generic assertions for UI elements

```typescript
const text = await page.locator('.status').textContent();

expect(text).toBe('Completed');

```

Better:

```typescript
await expect(page.locator('.status'))
    .toHaveText('Completed');

```

----------

### ❌ Using `waitForTimeout()`

```typescript
await page.waitForTimeout(3000);

await expect(locator).toBeVisible();

```

The explicit wait is usually unnecessary because the assertion already waits.

----------

### ❌ Verifying only actions

```typescript
await page.getByRole('button', {
    name: 'Save'
}).click();

```

Always verify the expected outcome.

```typescript
await expect(page.getByText('Saved'))
    .toBeVisible();

```

----------

# Interview Questions

### Q1. What is the difference between Playwright assertions and traditional assertions?

Playwright locator and page assertions automatically retry until they succeed or the timeout expires. Traditional assertions evaluate only once.

----------

### Q2. Why should `toHaveText()` be preferred over `textContent()` followed by `expect()`?

`toHaveText()` automatically waits for the expected text, while reading `textContent()` retrieves the current value immediately and may lead to flaky tests.

----------

### Q3. Which assertions auto-retry?

Assertions on **locators** and **pages**, such as `toBeVisible()`, `toHaveText()`, `toHaveCount()`, `toHaveURL()`, and `toHaveTitle()`.

----------

### Q4. Do generic assertions retry?

No. Assertions on plain JavaScript values, arrays, strings, numbers, or objects (`expect(value).toBe(...)`) execute immediately.

----------

### Q5. Why are Playwright assertions more reliable than explicit waits?

Because they wait only as long as necessary for the expected condition. Explicit waits always delay execution for a fixed time, even if the application is already ready.

----------

# Summary

Assertions are what transform an automated interaction into a meaningful test. Playwright's auto-retrying assertions are designed specifically for modern, asynchronous web applications, allowing tests to remain stable without relying on arbitrary delays. By choosing locator and page assertions whenever possible, you can write tests that are both more readable and significantly less flaky.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE5OTYzNDMyOF19
-->
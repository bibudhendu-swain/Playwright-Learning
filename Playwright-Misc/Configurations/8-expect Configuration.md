# Part 8 – `expect` Configuration & Assertions (Complete Guide)

The `expect` API is one of the most important parts of Playwright. Unlike many testing frameworks, Playwright's assertions are **auto-retrying**. Instead of failing immediately, Playwright waits for the expected condition to become true within the configured timeout.

This is one of the key reasons Playwright tests require far fewer explicit waits.

----------

# How `expect()` Works

When you write:

```ts
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();

```

Playwright does **not** simply check visibility once.

Instead, it repeatedly evaluates the locator until one of the following happens:

1.  The condition becomes true → ✅ Test passes.
    
2.  The timeout is reached → ❌ Test fails.
    

----------

## Internal Retry Loop

```text
Locator Found?

↓

No

↓

Retry

↓

No

↓

Retry

↓

Yes

↓

Assertion Passed

```

This automatic retrying is why assertions are much more reliable than manual waits.

----------

# Default Expect Timeout

By default:

```text
5 seconds

```

Playwright keeps retrying for up to **5 seconds**.

----------

# Configure Global Expect Timeout

```ts
export default defineConfig({

  expect: {
    timeout: 10000
  }

});

```

Now every assertion retries for:

```text
10 seconds

```

----------

# Override Per Assertion

```ts
await expect(
  page.getByText('Order Successful')
).toBeVisible({
  timeout: 20000
});

```

Only this assertion uses 20 seconds.

----------

# Why Auto-Retry Matters

Suppose a success message appears after **3 seconds**.

Without auto-retry:

```ts
expect(await page.locator('.toast').isVisible()).toBe(true);

```

This checks only once and may fail immediately.

With Playwright:

```ts
await expect(page.locator('.toast')).toBeVisible();

```

Playwright keeps checking until the toast appears or the timeout expires.

----------

# Locator Assertions

These assertions work with `Locator` objects and benefit from auto-retrying.

## Visibility

```ts
await expect(page.locator('#login')).toBeVisible();

```

Checks that the element is visible.

----------

## Hidden

```ts
await expect(page.locator('#loader')).toBeHidden();

```

Waits until the loader disappears.

----------

## Attached

```ts
await expect(page.locator('#user')).toBeAttached();

```

The element exists in the DOM.

----------

## Detached

```ts
await expect(page.locator('#loader')).toBeDetached();

```

The element is removed from the DOM.

----------

## Enabled

```ts
await expect(page.locator('#submit')).toBeEnabled();

```

----------

## Disabled

```ts
await expect(page.locator('#submit')).toBeDisabled();

```

----------

## Editable

```ts
await expect(page.locator('#email')).toBeEditable();

```

----------

## Empty

```ts
await expect(page.locator('#cart')).toBeEmpty();

```

----------

## Focused

```ts
await expect(page.locator('#username')).toBeFocused();

```

----------

# Text Assertions

## Exact Text

```ts
await expect(page.locator('h1'))
  .toHaveText('Welcome');

```

----------

## Partial Text

```ts
await expect(page.locator('h1'))
  .toContainText('Welcome');

```

----------

## Multiple Elements

```ts
await expect(page.locator('li'))
  .toHaveText([
    'Apple',
    'Orange',
    'Banana'
  ]);

```

Useful for validating lists.

----------

# Attribute Assertions

```ts
await expect(page.locator('#login'))
  .toHaveAttribute('type', 'submit');

```

----------

# CSS Assertions

```ts
await expect(page.locator('.button'))
  .toHaveCSS('color', 'rgb(255, 0, 0)');

```

----------

# Class Assertions

```ts
await expect(page.locator('.card'))
  .toHaveClass('active');

```

Regular expression:

```ts
await expect(page.locator('.card'))
  .toHaveClass(/active/);

```

----------

# Value Assertions

```ts
await expect(page.locator('#username'))
  .toHaveValue('john');

```

----------

# Count Assertions

```ts
await expect(page.locator('.product'))
  .toHaveCount(10);

```

Playwright waits until there are exactly ten matching elements.

----------

# URL Assertions

```ts
await expect(page)
  .toHaveURL(/dashboard/);

```

----------

Exact URL

```ts
await expect(page)
  .toHaveURL('https://example.com/dashboard');

```

----------

# Title Assertions

```ts
await expect(page)
  .toHaveTitle('Dashboard');

```

Regular expression:

```ts
await expect(page)
  .toHaveTitle(/Dashboard/);

```

----------

# Soft Assertions

Normally:

```ts
await expect(page.locator('#user'))
  .toBeVisible();

await expect(page.locator('#password'))
  .toBeVisible();

```

If the first assertion fails:

```text
Second assertion never runs.

```

----------

Soft assertion:

```ts
await expect.soft(page.locator('#user'))
  .toBeVisible();

await expect.soft(page.locator('#password'))
  .toBeVisible();

```

Now both assertions execute, and Playwright reports all soft assertion failures at the end of the test.

----------

# When to Use Soft Assertions

Useful when validating:

-   dashboards
    
-   forms
    
-   reports
    
-   multiple widgets
    

Example:

```text
Verify:

✓ Header

✓ Menu

✓ Footer

✓ Logo

✓ Search

✓ Notification

```

You receive a complete list of failures instead of stopping at the first one.

----------

# Custom Assertion Messages

```ts
await expect(
  page.locator('#login'),
  'Login button should be visible before clicking'
).toBeVisible();

```

Failure output becomes much more descriptive.

----------

# Polling Assertions

Sometimes you need to verify a value that changes over time.

```ts
await expect.poll(async () => {

  const response = await page.locator('#status').textContent();

  return response;

}).toBe('Completed');

```

Playwright repeatedly calls the function until it returns `"Completed"` or times out.

----------

# Custom Polling Interval

```ts
await expect.poll(

  async () => getStatus(),

  {

    intervals: [1000, 2000, 5000],

    timeout: 30000

  }

).toBe('Completed');

```

This is useful for asynchronous workflows such as background jobs or eventual consistency.

----------

# `expect.configure()`

You can create a customized `expect` instance.

```ts
const slowExpect = expect.configure({

  timeout: 20000

});

```

Usage:

```ts
await slowExpect(
  page.locator('.spinner')
).toBeHidden();

```

This avoids repeating timeout options across multiple assertions.

----------

# Negation

```ts
await expect(page.locator('#error'))
  .not.toBeVisible();

```

Another example:

```ts
await expect(page)
  .not.toHaveURL(/error/);

```

----------

# Snapshot Assertions

Playwright supports snapshot testing.

```ts
await expect(page)
  .toHaveScreenshot();

```

On the first run:

```text
Snapshot Created

```

Subsequent runs compare the current page against the baseline image.

----------

Custom name:

```ts
await expect(page)
  .toHaveScreenshot('homepage.png');

```

----------

# ARIA Snapshot Assertions

Playwright also supports accessibility snapshots.

```ts
await expect(page.locator('main'))
  .toMatchAriaSnapshot();

```

This validates the accessibility tree rather than visual appearance.

----------

# Combining Assertions

Example:

```ts
await expect(page).toHaveURL(/dashboard/);

await expect(page.locator('h1'))
  .toHaveText('Dashboard');

await expect(page.locator('.user'))
  .toContainText('John');

await expect(page.locator('#logout'))
  .toBeVisible();

```

----------

# Enterprise Example

```ts
test('Checkout', async ({ page }) => {

  await page.goto('/checkout');

  await expect(page)
    .toHaveURL(/checkout/);

  await expect(page.locator('h1'))
    .toHaveText('Checkout');

  await expect(page.locator('#place-order'))
    .toBeEnabled();

  await expect(page.locator('.cart-item'))
    .toHaveCount(3);

});

```

Notice there are **no manual waits**.

----------

# Common Mistakes

## ❌ Using `isVisible()`

```ts
expect(await locator.isVisible()).toBe(true);

```

This checks only once.

Prefer:

```ts
await expect(locator).toBeVisible();

```

----------

## ❌ Using `waitForTimeout()`

```ts
await page.waitForTimeout(5000);

```

Then:

```ts
expect(...)

```

Instead:

```ts
await expect(locator).toBeVisible();

```

Let Playwright handle the waiting.

----------

## ❌ Using Soft Assertions Everywhere

Soft assertions are valuable for collecting multiple failures, but overusing them can allow tests to continue after a critical prerequisite has failed.

----------

## ❌ Increasing Timeout Excessively

```ts
expect: {

timeout: 60000

}

```

A 60-second assertion timeout often hides performance issues and slows down feedback.

----------

# Interview Questions

### Q1. What is the default `expect` timeout?

```text
5 seconds

```

----------

### Q2. Why is `expect(locator).toBeVisible()` preferred over `locator.isVisible()`?

Because `expect()` automatically retries until the condition becomes true or the timeout expires, while `isVisible()` checks only once.

----------

### Q3. What is a soft assertion?

A soft assertion records failures but allows the test to continue executing subsequent assertions.

----------

### Q4. What does `expect.poll()` do?

It repeatedly executes a function until the expected value is returned or the timeout is reached.

----------

### Q5. Can you override the timeout for a single assertion?

Yes.

```ts
await expect(locator).toBeVisible({
  timeout: 10000
});

```

----------

### Q6. What is `expect.configure()` used for?

It creates a customized `expect` instance with predefined settings such as a different timeout.

----------

### Q7. Which assertions automatically retry?

Assertions built on Playwright's `expect()` with `Locator` and `Page` APIs (e.g., `toBeVisible`, `toHaveText`, `toHaveURL`, `toHaveTitle`, etc.) automatically retry until the condition is met or the timeout expires.

----------

# Best Practices

-   Prefer `expect(locator)` over methods like `isVisible()` or `textContent()` when verifying UI state.
    
-   Keep the global `expect.timeout` reasonable (5–10 seconds for most applications).
    
-   Use soft assertions when validating multiple independent UI elements.
    
-   Add custom assertion messages for clearer failure reports.
    
-   Use `expect.poll()` for asynchronous workflows such as background jobs or delayed status updates.
    
-   Avoid manual waits (`waitForTimeout`) unless debugging.
    
-   Use snapshot assertions judiciously—reserve them for stable UI components to minimize maintenance.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg5NTk4MDAyOF19
-->
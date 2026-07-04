# Part 3 – Timeouts in Playwright (`timeout`, `globalTimeout`, `actionTimeout`, `navigationTimeout`, `expect.timeout`)

Timeouts are one of the most misunderstood parts of Playwright. A common misconception is that there's only a single timeout. In reality, Playwright has **multiple independent timeouts**, each serving a different purpose.

Understanding how they interact helps you write reliable tests and avoid unnecessary failures.

----------

# Timeout Hierarchy

When a test runs, Playwright applies different timeouts depending on the operation.

```text
Global Timeout
    │
    ├── Test Timeout
    │      │
    │      ├── Action Timeout
    │      ├── Navigation Timeout
    │      └── Expect Timeout
    │
    └── Hook Timeout (beforeAll, afterAll, etc.)

```

Each timeout controls a different aspect of execution.

----------

# 1. `timeout` (Test Timeout)

## Purpose

The maximum time allowed for a **single test**, including:

-   test body
    
-   fixtures
    
-   `beforeEach`
    
-   `afterEach`
    

If the test exceeds this limit, Playwright stops it.

----------

## Default

```text
30 seconds

```

----------

## Configuration

```ts
export default defineConfig({
    timeout: 30 * 1000
});

```

Equivalent:

```ts
timeout: 30000

```

----------

## Example

```ts
test('Long test', async ({ page }) => {

    await page.goto('/');

    await page.waitForTimeout(35000);

});

```

Output:

```text
Test timeout of 30000ms exceeded.

```

----------

## Increasing Timeout

```ts
export default defineConfig({
    timeout: 120000
});

```

Now every test can run for two minutes.

----------

# Override Per Test

Sometimes only one test is slow.

```ts
test('Import huge dataset', async ({ page }) => {

    test.setTimeout(180000);

    // long execution

});

```

Only this test gets a three-minute timeout.

----------

# Override Inside `describe`

```ts
test.describe('Report Generation', () => {

    test.setTimeout(120000);

});

```

All tests inside the suite inherit the timeout.

----------

# 2. `globalTimeout`

## Purpose

Limits the **entire test run**, not an individual test.

Useful in CI to prevent endless executions.

----------

## Example

```ts
export default defineConfig({

    globalTimeout: 60 * 60 * 1000

});

```

One hour maximum for the full suite.

----------

Suppose:

```text
500 tests

Each takes 20 seconds

```

Without `globalTimeout`, execution may continue indefinitely if workers are stalled.

With:

```ts
globalTimeout: 3600000

```

Playwright terminates the run after one hour.

----------

# Difference Between `timeout` and `globalTimeout`

Test Timeout

Global Timeout

Per test

Entire execution

Default: 30 seconds

No default

Stops one test

Stops all workers

Configurable with `test.setTimeout()`

Cannot be overridden by a test

----------

# 3. `actionTimeout`

## Purpose

Maximum time for a Playwright action.

Examples:

```ts
click()

fill()

hover()

check()

selectOption()

dragTo()

```

----------

Default:

```text
0 (disabled)

```

Meaning:

Actions use Playwright's auto-wait mechanism until the enclosing test timeout is reached.

----------

Configuration:

```ts
use: {

    actionTimeout: 10000

}

```

----------

Example

```ts
await page.getByRole('button').click();

```

If the button never becomes clickable:

```text
Timeout 10000ms exceeded while waiting for locator

```

----------

Why use it?

Instead of waiting:

```text
30 seconds

```

for every failed click,

Playwright fails after:

```text
10 seconds

```

making failures easier to diagnose.

----------

# Override Per Action

```ts
await page.locator('#submit').click({

    timeout: 5000

});

```

----------

# 4. `navigationTimeout`

## Purpose

Controls navigation-related methods.

Examples:

```ts
goto()

reload()

goBack()

goForward()

waitForURL()

```

----------

Configuration

```ts
use: {

    navigationTimeout: 45000

}

```

----------

Example

```ts
await page.goto('/dashboard');

```

If loading takes more than:

```text
45 seconds

```

Playwright throws:

```text
Navigation timeout exceeded

```

----------

Override Per Navigation

```ts
await page.goto('/dashboard', {

    timeout: 10000

});

```

----------

# Difference Between Action and Navigation Timeout

Action Timeout

Navigation Timeout

click

goto

fill

reload

hover

waitForURL

press

goBack

----------

# 5. `expect.timeout`

## Purpose

Controls how long Playwright retries an assertion.

----------

Default

```text
5 seconds

```

----------

Example

```ts
await expect(

    page.getByText('Order placed')

).toBeVisible();

```

Playwright retries automatically for five seconds.

----------

Configuration

```ts
expect: {

    timeout: 10000

}

```

----------

Override Per Assertion

```ts
await expect(

    page.locator('.success')

).toBeVisible({

    timeout: 15000

});

```

----------

# Why `expect.timeout` Exists

Suppose:

```ts
await expect(page.locator('.toast')).toBeVisible();

```

Toast appears after:

```text
3 seconds

```

Playwright keeps retrying until it appears.

No manual wait is required.

----------

# How Timeouts Work Together

Configuration

```ts
export default defineConfig({

    timeout: 30000,

    use: {

        actionTimeout: 5000,

        navigationTimeout: 15000

    },

    expect: {

        timeout: 8000

    }

});

```

Execution

```ts
await page.goto('/');

await page.locator('#login').click();

await expect(

page.getByText('Welcome')

).toBeVisible();

```

Timeouts applied:

```text
goto()            → 15 seconds

click()           → 5 seconds

expect()          → 8 seconds

Entire test       → 30 seconds

```

----------

# Which Timeout Wins?

Example

```text
Test Timeout = 20 sec

Navigation Timeout = 60 sec

```

Navigation starts.

After:

```text
20 seconds

```

The test fails.

The enclosing **Test Timeout** always wins because the test cannot exceed its overall limit.

----------

# Common Mistakes

## Mistake 1

Increasing only:

```ts
expect.timeout

```

when the failure comes from:

```text
Test Timeout

```

----------

## Mistake 2

Using:

```ts
page.waitForTimeout(10000)

```

instead of:

```ts
expect(locator).toBeVisible()

```

Fixed waits slow tests and are generally discouraged.

----------

## Mistake 3

Setting:

```ts
timeout: 10 minutes

```

for every test.

This can hide performance issues and make failures take much longer to surface.

----------

# Enterprise Configuration Example

```ts
export default defineConfig({

    timeout: 90000,

    globalTimeout: 4 * 60 * 60 * 1000,

    expect: {

        timeout: 10000

    },

    use: {

        actionTimeout: 10000,

        navigationTimeout: 30000

    }

});

```

This setup works well for medium to large enterprise UI suites.

----------

# Interview Questions

### Q1. What is the default test timeout?

**30 seconds.**

----------

### Q2. What is the default `expect` timeout?

**5 seconds.**

----------

### Q3. What is the default `actionTimeout`?

**0**, which means no separate limit is enforced; actions are governed by Playwright's auto-wait and the enclosing test timeout.

----------

### Q4. Can `actionTimeout` be greater than `test timeout`?

Yes, but it has little practical effect. The **test timeout** is the upper bound for the entire test and will terminate the test first if reached.

----------

### Q5. Can you override a timeout for a single test?

Yes.

```ts
test('Slow test', async ({ page }) => {

    test.setTimeout(120000);

});

```

----------

### Q6. Which timeout controls assertions?

`expect.timeout`

----------

### Q7. Which timeout controls `page.goto()`?

`navigationTimeout`

----------

### Q8. Which timeout controls `locator.click()`?

`actionTimeout`

----------

# Best Practices

-   Keep the global **test timeout** realistic (e.g., 30–90 seconds) rather than excessively large.
    
-   Use `test.setTimeout()` only for genuinely slow tests instead of increasing the timeout for the entire suite.
    
-   Prefer increasing `expect.timeout` for slow-loading UI elements rather than inserting hard waits.
    
-   Set `actionTimeout` to catch stuck UI interactions quickly, especially in CI.
    
-   Use `globalTimeout` in CI pipelines to prevent stalled test runs from consuming build agents indefinitely.
    
-   Avoid `page.waitForTimeout()` except for debugging or very specific timing scenarios; rely on Playwright's auto-waiting and retrying assertions whenever possible.
    

----------

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU5NTM4Nzk5MF19
-->
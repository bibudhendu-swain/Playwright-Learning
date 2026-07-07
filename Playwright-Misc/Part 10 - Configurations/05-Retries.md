# Part 5 – Retries & CI Configuration (`retries`, `forbidOnly`, `maxFailures`, `failOnFlakyTests`)

In real-world automation, tests may occasionally fail due to external factors such as network latency, backend response times, or temporary infrastructure issues. Playwright provides built-in features to handle these situations while also helping teams identify and eliminate flaky tests.

This section covers:

-   `retries`
    
-   `forbidOnly`
    
-   `maxFailures`
    
-   `failOnFlakyTests`
    
-   How retries work internally
    
-   CI best practices
    

----------

# How Playwright Classifies Test Results

Unlike many frameworks, Playwright doesn't just classify tests as **Passed** or **Failed**.

It categorizes them into three states:

Status

Meaning

**Passed**

Passed on the first attempt

**Flaky**

Failed initially but passed on a retry

**Failed**

Failed even after all retries

Example:

```text
Attempt 1 ❌

Attempt 2 ✅

```

Result:

```text
Flaky

```

----------

# 1. `retries`

## Purpose

Retries automatically rerun a failed test before marking it as failed.

----------

## Default

```ts
retries: 0

```

No retries.

----------

## Configuration

```ts
export default defineConfig({

    retries: 2

});

```

Execution:

```text
Attempt 1 ❌

↓

Attempt 2 ❌

↓

Attempt 3 ✅

```

Result:

```text
Flaky

```

----------

# Retry Timeline

```text
Test Starts

↓

Attempt 1

↓

Fail

↓

Retry

↓

Attempt 2

↓

Pass

↓

Marked as Flaky

```

----------

# Retry Count

```ts
retries: 3

```

Means:

```text
Original Run

+

Retry 1

+

Retry 2

+

Retry 3

```

Total:

```text
4 executions

```

----------

# Retry Only in CI

This is one of the most common enterprise configurations.

```ts
export default defineConfig({

    retries: process.env.CI ? 2 : 0

});

```

Local:

```text
Retries = 0

```

CI:

```text
Retries = 2

```

Developers get fast feedback locally, while CI is more resilient to transient issues.

----------

# Override from CLI

```bash
npx playwright test --retries=3

```

Overrides the configuration file.

----------

# What Happens During a Retry?

This is an important interview topic.

When a test is retried:

-   A **new browser context** is created.
    
-   Fixtures are reinitialized (depending on their scope).
    
-   The test starts from the beginning.
    
-   Previous page state is discarded.
    

Example:

```ts
test('Login', async ({ page }) => {

    await page.goto('/');

    await page.fill('#username', 'user');

});

```

On retry:

```text
New browser

↓

New page

↓

goto()

↓

fill()

```

Nothing is reused from the previous attempt.

----------

# Worker Restart After Failure

Suppose:

```text
Worker 1

Test A ❌

Test B

Test C

```

If Test A fails:

Playwright starts a **new worker process** before running the retry or subsequent tests. This helps isolate failures caused by corrupted worker state.

----------

# 2. `forbidOnly`

## Purpose

Prevents accidentally committing focused tests.

----------

Suppose someone writes:

```ts
test.only('Login', async () => {

});

```

Only this test executes.

If committed:

The CI pipeline may silently skip the rest of the suite.

----------

Configuration:

```ts
export default defineConfig({

    forbidOnly: !!process.env.CI

});

```

In CI:

```text
Error:

test.only is not allowed

```

Execution stops immediately.

----------

# Example

```ts
test.only('Checkout', async () => {

});

```

Local:

```text
Runs successfully

```

CI:

```text
Build fails

```

----------

# Why `!!process.env.CI`?

`process.env.CI` is usually a string such as `"true"`.

Using:

```ts
!!process.env.CI

```

converts it to a boolean:

```text
undefined → false

"true" → true

```

----------

# 3. `maxFailures`

## Purpose

Stops the test run after a specified number of failures.

----------

Configuration:

```ts
export default defineConfig({

    maxFailures: 5

});

```

Execution:

```text
Test 1 ❌

Failure Count = 1


Test 2 ❌

Failure Count = 2


...

Test 5 ❌

↓

Execution stops

```

Remaining tests are skipped.

----------

# Why Use It?

Imagine:

```text
5000 tests

Backend is down

```

Without:

```ts
maxFailures

```

All 5000 tests fail.

With:

```ts
maxFailures: 20

```

The suite stops after 20 failures, saving time and CI resources.

----------

# CLI Override

```bash
npx playwright test --max-failures=10

```

----------

# Recommended Usage

CI:

```ts
maxFailures: 20

```

Local:

```ts
maxFailures: undefined

```

----------

# 4. `failOnFlakyTests`

## Purpose

Treat flaky tests as build failures.

----------

Configuration

```ts
export default defineConfig({

    failOnFlakyTests: true

});

```

Scenario:

```text
Attempt 1 ❌

Attempt 2 ✅

```

Normally:

```text
Flaky

Build passes

```

With:

```ts
failOnFlakyTests: true

```

Result:

```text
Build fails

```

This encourages teams to fix flaky tests instead of ignoring them.

----------

# Local vs CI Example

```ts
export default defineConfig({

    failOnFlakyTests: !!process.env.CI

});

```

Local:

```text
Flaky allowed

```

CI:

```text
Flaky build fails

```

----------

# Combining Retry Features

```ts
export default defineConfig({

    retries: 2,

    forbidOnly: !!process.env.CI,

    maxFailures: 20,

    failOnFlakyTests: true

});

```

Behavior:

-   Retry failed tests up to two times.
    
-   Prevent `test.only` in CI.
    
-   Stop after 20 failures.
    
-   Fail the build if any test is flaky.
    

----------

# Enterprise CI Configuration

```ts
export default defineConfig({

    retries: process.env.CI ? 2 : 0,

    workers: process.env.CI ? 4 : undefined,

    forbidOnly: !!process.env.CI,

    maxFailures: process.env.CI ? 20 : undefined,

    failOnFlakyTests: !!process.env.CI

});

```

This is a common starting point for medium to large Playwright projects.

----------

# Retry Flow Example

Suppose:

```text
Test A

Attempt 1 ❌

↓

Retry

↓

Attempt 2 ❌

↓

Retry

↓

Attempt 3 ❌

```

With:

```ts
retries: 2

```

Result:

```text
Failed

```

----------

Another example:

```text
Attempt 1 ❌

↓

Retry

↓

Attempt 2 ✅

```

Result:

```text
Flaky

```

----------

# Common Mistakes

## Mistake 1

Using:

```ts
retries: 5

```

to hide unstable tests.

Retries should help mitigate temporary issues, not mask defects. A consistently flaky test should be investigated and fixed.

----------

## Mistake 2

Not enabling:

```ts
forbidOnly

```

A committed `test.only()` can unintentionally prevent most of the suite from running in CI.

----------

## Mistake 3

Not configuring:

```ts
maxFailures

```

A widespread outage (e.g., authentication service down) can waste significant CI time if thousands of tests continue to execute.

----------

## Mistake 4

Ignoring flaky tests.

Treat flaky tests as technical debt. Over time, they reduce confidence in the automation suite.

----------

# Interview Questions

### Q1. What happens when a Playwright test is retried?

A fresh browser context is created, fixtures are reinitialized according to their scope, and the test starts from the beginning.

----------

### Q2. What is the default retry count?

```text
0

```

----------

### Q3. If `retries: 2`, how many total attempts are made?

```text
3

1 original run

2 retries

```

----------

### Q4. What does `forbidOnly` do?

It prevents `test.only()` from being executed, typically in CI, helping ensure the full suite runs.

----------

### Q5. What does `maxFailures` do?

It stops the entire test run after the configured number of test failures.

----------

### Q6. What is a flaky test?

A test that fails on the initial run but passes on a retry.

----------

### Q7. What does `failOnFlakyTests` do?

It marks the test run as failed if any tests are classified as flaky.

----------

# Best Practices

-   Use `retries: 0` for local development to expose issues quickly.
    
-   Use `retries: 1` or `2` in CI to reduce failures caused by transient infrastructure issues.
    
-   Always enable `forbidOnly` in CI.
    
-   Configure `maxFailures` to avoid wasting CI resources during widespread failures.
    
-   Monitor flaky tests regularly and fix the root causes instead of increasing retry counts.
    
-   Consider enabling `failOnFlakyTests` in mature projects where test stability is a priority.
    

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIzMzIzNjI4MF19
-->
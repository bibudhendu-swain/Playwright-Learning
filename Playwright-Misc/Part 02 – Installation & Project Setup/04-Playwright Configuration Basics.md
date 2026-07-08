Almost every enterprise Playwright project customizes `playwright.config.ts`. In fact, it is the **heart of the Playwright framework**.

----------

# Part 2 – Installation & Project Setup

# Chapter 4 – Playwright Configuration Basics

----------

# Introduction

A Playwright project is driven by a single configuration file:

```text
playwright.config.ts

```

This file defines how Playwright executes tests.

Instead of configuring each test individually, common settings are centralized in one place, making the framework easier to maintain, scale, and customize.

Think of this file as the **control center** of your Playwright project.

----------

# Why Do We Need Configuration?

Imagine a project with 500 test files.

Without centralized configuration, every test would need to specify:

-   Browser
    
-   Timeout
    
-   Base URL
    
-   Retry count
    
-   Reporter
    
-   Screenshots
    
-   Video recording
    

That would result in massive duplication.

Instead:

```text
Tests

↓

playwright.config.ts

↓

Execution

```

Every test inherits the same configuration unless it explicitly overrides it.

----------

# The Role of `playwright.config.ts`

The configuration file controls:

-   Test discovery
    
-   Browser selection
    
-   Projects
    
-   Timeouts
    
-   Retries
    
-   Parallel execution
    
-   Reporters
    
-   Screenshots
    
-   Videos
    
-   Tracing
    
-   Base URL
    
-   Environment settings
    

It acts as a blueprint for the test runner.

----------

# Typical Configuration File

A basic configuration looks like this:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    headless: true,
    baseURL: 'https://example.com'
  }
});

```

Although small, this file controls the behavior of every test.

----------

# Why `defineConfig()`?

Playwright recommends wrapping the configuration with:

```typescript
defineConfig()

```

Benefits:

-   Type safety
    
-   IntelliSense
    
-   Auto-completion
    
-   Better validation
    
-   Cleaner code
    

Instead of exporting a plain object, `defineConfig()` helps the IDE catch mistakes while you're writing the configuration.

----------

# Configuration Loading Process

When you run:

```bash
npx playwright test

```

Playwright performs the following sequence:

```text
Read Config

↓

Validate Config

↓

Discover Tests

↓

Create Workers

↓

Execute Tests

↓

Generate Reports

```

The configuration is loaded once before test execution begins.

----------

# Understanding `testDir`

Example:

```typescript
testDir: './tests'

```

This tells Playwright where to look for test files.

Example:

```text
project/

├── tests/
│   ├── login.spec.ts
│   ├── search.spec.ts
│   └── checkout.spec.ts

```

If your tests are stored elsewhere, update `testDir` accordingly.

----------

# The `use` Section

One of the most frequently used sections is:

```typescript
use: {
    ...
}

```

Everything inside `use` becomes the default configuration for every test.

Example:

```typescript
use: {

    headless: true,

    baseURL: 'https://example.com'

}

```

Think of it as the default browser settings.

----------

# Base URL

Instead of:

```typescript
await page.goto(
'https://example.com/login'
);

```

you can write:

```typescript
await page.goto('/login');

```

because:

```typescript
baseURL:
'https://example.com'

```

has already been configured.

Benefits:

-   Cleaner code
    
-   Easy environment switching
    
-   Less duplication
    

----------

# Headless Mode

Example:

```typescript
headless: true

```

Browser runs invisibly.

```text
Playwright

↓

Browser

↓

No UI

```

Ideal for:

-   CI/CD
    
-   Faster execution
    
-   Regression suites
    

----------

# Headed Mode

Example:

```typescript
headless: false

```

Browser opens visibly.

Useful during:

-   Development
    
-   Debugging
    
-   Demo sessions
    

----------

# Choosing Between Headless and Headed

Headless

Headed

Faster

Easier debugging

CI/CD

Local development

Lower resource usage

Visual validation

Most teams develop in headed mode and execute CI pipelines in headless mode.

----------

# Timeouts

Timeouts prevent tests from waiting forever.

Example:

```typescript
timeout: 30000

```

means:

```text
30 Seconds

↓

Fail Test

```

if the test exceeds the configured duration.

----------

# Expect Timeout

Assertions have a separate timeout.

Example:

```typescript
expect: {

    timeout: 5000

}

```

This controls how long Playwright retries assertions before failing.

----------

# Retries

Example:

```typescript
retries: 2

```

Execution:

```text
Attempt 1

↓

Fail

↓

Attempt 2

↓

Fail

↓

Attempt 3

```

Retries help reduce failures caused by temporary environmental issues, but they should not be used to hide unstable tests.

----------

# Workers

Example:

```typescript
workers: 4

```

Execution:

```text
Worker 1

Worker 2

Worker 3

Worker 4

```

Multiple workers execute tests in parallel.

We'll study workers in detail later.

----------

# Fully Parallel

Example:

```typescript
fullyParallel: true

```

Allows Playwright to execute tests within the same file in parallel, provided they are independent.

Use this setting only when tests are designed to avoid shared state.

----------

# Reporter

Example:

```typescript
reporter: 'html'

```

Output:

```text
Execution

↓

HTML Report

```

Playwright supports multiple reporters, including HTML, List, JSON, JUnit, and custom reporters.

----------

# Screenshots

Example:

```typescript
screenshot:
'only-on-failure'

```

Useful for debugging failed tests.

----------

# Video Recording

Example:

```typescript
video:
'retain-on-failure'

```

Videos are recorded only when needed.

This saves storage while preserving useful debugging artifacts.

----------

# Trace Collection

Example:

```typescript
trace:
'on-first-retry'

```

Recommended for most projects.

Workflow:

```text
Test Fails

↓

Retry

↓

Capture Trace

```

This balances diagnostics with storage usage.

----------

# Browser Selection

Example:

```typescript
projects: [

{
name: 'Chromium'
}

]

```

Projects allow the same test suite to execute against different browsers or configurations.

----------

# Environment Variables

Instead of hardcoding values:

```typescript
baseURL:
'https://test.example.com'

```

prefer:

```typescript
baseURL:
process.env.BASE_URL

```

Benefits:

-   Supports multiple environments
    
-   Keeps secrets out of source code
    
-   Simplifies CI/CD configuration
    

----------

# Configuration Hierarchy

Playwright applies configuration in this order:

```text
Global Config

↓

Project Config

↓

Test Config

↓

Individual API Options

```

More specific settings override broader ones.

----------

# Configuration Best Practices

Keep configuration:

-   Centralized
    
-   Environment-specific where appropriate
    
-   Free from sensitive credentials
    
-   Well documented
    

Avoid duplicating settings across test files.

----------

# Sample Enterprise Configuration

```typescript
export default defineConfig({

    testDir: './tests',

    timeout: 30000,

    retries: 2,

    workers: 4,

    reporter: 'html',

    use: {

        baseURL:
        process.env.BASE_URL,

        headless: true,

        screenshot:
        'only-on-failure',

        trace:
        'on-first-retry',

        video:
        'retain-on-failure'

    }

});

```

This is a solid starting point for many enterprise projects.

----------

# Common Mistakes

### ❌ Hardcoding URLs

Prefer environment variables over fixed URLs.

----------

### ❌ Setting Very Large Timeouts

Large timeouts often hide performance issues.

----------

### ❌ Enabling Videos for Every Test

Recording every execution consumes significant disk space.

----------

### ❌ Using Retries to Hide Flaky Tests

Retries should help with temporary issues, not replace proper test stabilization.

----------

### ❌ Scattering Configuration Across Test Files

Centralize shared settings in `playwright.config.ts`.

----------

# Enterprise Best Practices

-   Use `defineConfig()` for type-safe configuration.
    
-   Configure a `baseURL` to simplify navigation.
    
-   Store environment-specific values outside the codebase.
    
-   Use HTML reports and traces in CI.
    
-   Capture screenshots and videos only when needed.
    
-   Keep timeouts realistic and monitor slow tests.
    
-   Document configuration choices so the team understands why they exist.
    

----------

# Interview Questions

### Q1. What is the purpose of `playwright.config.ts`?

It is the central configuration file that defines how Playwright discovers, executes, and reports tests. It controls settings such as browsers, timeouts, retries, reporters, projects, and default browser options.

----------

### Q2. Why should `defineConfig()` be used?

`defineConfig()` provides type safety, IntelliSense, and compile-time validation, making configuration easier to write and maintain.

----------

### Q3. What is the advantage of configuring a `baseURL`?

A `baseURL` allows tests to navigate using relative paths, reducing duplication and making it easier to switch between environments.

----------

### Q4. What is the difference between `timeout` and `expect.timeout`?

`timeout` limits the total execution time of a test, while `expect.timeout` controls how long Playwright retries an assertion before failing.

----------

### Q5. Why is `trace: 'on-first-retry'` a common recommendation?

It captures detailed diagnostic information only when a test fails and is retried, providing useful debugging artifacts without generating traces for every successful execution.

----------

# Summary

The `playwright.config.ts` file is the central control point of every Playwright project. It defines how tests are discovered, how browsers are launched, how long tests wait, how reports are generated, and how execution behaves across environments. A well-designed configuration simplifies maintenance, supports scalable automation, and provides a consistent execution experience for developers, CI/CD pipelines, and enterprise teams alike.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY1MjIwMTQ0Nl19
-->
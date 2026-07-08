# Part 2 – Installation & Project Setup

# Chapter 9 – Running and Debugging Playwright Tests

----------

# Introduction

Writing a Playwright test is only the beginning.

A typical automation engineer spends a significant amount of time:

-   Running tests
    
-   Debugging failures
    
-   Investigating flaky behavior
    
-   Inspecting traces
    
-   Reviewing reports
    
-   Re-running failed tests
    

Playwright provides a rich set of tools to support every stage of this workflow.

By the end of this chapter, you'll understand how to execute tests efficiently and use Playwright's built-in debugging capabilities to diagnose problems quickly.

----------

# The Test Execution Lifecycle

Every execution follows the same general workflow.

```text
Write Test

↓

Run Test

↓

Observe Results

↓

Debug Failure

↓

Fix Test

↓

Run Again

```

Playwright provides tools for each stage.

----------

# Running All Tests

The simplest command is:

```bash
npx playwright test

```

Playwright will:

```text
Read Configuration

↓

Discover Tests

↓

Create Workers

↓

Launch Browsers

↓

Execute Tests

↓

Generate Report

```

Every discovered test matching the configured patterns will run.

----------

# Running a Single Test File

Instead of executing the entire suite:

```bash
npx playwright test tests/login.spec.ts

```

Useful when working on one feature.

----------

# Running a Single Test

Suppose your file contains:

```typescript
test('User Login', async () => { });

test('User Logout', async () => { });

```

Execute only one test:

```bash
npx playwright test --grep "User Login"

```

Only matching test names execute.

----------

# Running Tests by Pattern

Example

```bash
npx playwright test --grep "Checkout"

```

Workflow

```text
Discover Tests

↓

Filter

↓

Execute Matches

```

This is useful for large regression suites.

----------

# Excluding Tests

Example

```bash
npx playwright test --grep-invert "Smoke"

```

Runs every test except those matching "Smoke."

----------

# Running Tests in Headed Mode

By default,

tests typically execute in headless mode.

To see the browser:

```bash
npx playwright test --headed

```

Useful for:

-   Development
    
-   Debugging
    
-   Demonstrations
    

----------

# Running in Debug Mode

One of Playwright's most powerful features.

```bash
npx playwright test --debug

```

Debug mode automatically:

-   Opens the browser
    
-   Opens Playwright Inspector
    
-   Slows execution
    
-   Pauses at breakpoints
    

----------

# What Happens in Debug Mode?

```text
Run Test

↓

Pause

↓

Inspector

↓

Step

↓

Continue

```

This allows you to observe exactly what the test is doing.

----------

# Using Playwright Inspector

The Inspector provides:

-   Step-by-step execution
    
-   Locator highlighting
    
-   Console output
    
-   Action log
    
-   Resume and pause controls
    

Think of it as a debugger designed specifically for browser automation.

----------

# Using UI Mode

Launch UI Mode:

```bash
npx playwright test --ui

```

UI Mode provides:

-   Test Explorer
    
-   Run buttons
    
-   Debug buttons
    
-   Filters
    
-   Trace integration
    
-   Live results
    

Architecture

```text
UI Mode

↓

Select Test

↓

Execute

↓

View Results

```

----------

# Watching Test Progress

During execution,

the console displays:

```text
Running 12 tests

✓ Login

✓ Search

✗ Checkout

✓ Logout

```

This provides immediate feedback without opening reports.

----------

# Running Tests in Parallel

Example

```bash
npx playwright test --workers=4

```

Execution

```text
Worker 1

Worker 2

Worker 3

Worker 4

```

Parallel execution significantly reduces runtime for large suites.

----------

# Running Sequentially

Sometimes debugging is easier with a single worker.

```bash
npx playwright test --workers=1

```

Tests execute one after another.

----------

# Running Only Failed Tests

After a previous execution,

run:

```bash
npx playwright test --last-failed

```

Only previously failed tests execute.

Very useful during bug fixing.

----------

# Running Tests by Project

If your configuration defines:

```text
Chromium

Firefox

WebKit

```

Run only Chromium:

```bash
npx playwright test --project=Chromium

```

----------

# Running Tests by Tag

Example

```typescript
test('@smoke Login Test', async () => {});

```

Execute:

```bash
npx playwright test --grep "@smoke"

```

Tags allow flexible grouping without creating separate test suites.

----------

# Viewing the HTML Report

After execution:

```bash
npx playwright show-report

```

The report contains:

-   Passed tests
    
-   Failed tests
    
-   Duration
    
-   Attachments
    
-   Errors
    
-   Screenshots
    
-   Videos
    
-   Traces
    

----------

# Understanding the HTML Report

```text
Execution

↓

HTML Report

↓

Open Failed Test

↓

Inspect Details

```

The report is often the first place engineers investigate failures.

----------

# Capturing Screenshots

When configured,

failed tests automatically include screenshots.

Example configuration:

```typescript
use: {
    screenshot: 'only-on-failure'
}

```

Screenshots provide a snapshot of the application state at the point of failure.

----------

# Capturing Videos

Example

```typescript
use: {
    video: 'retain-on-failure'
}

```

Videos help understand dynamic failures that are difficult to reproduce.

----------

# Using Trace Viewer

Recommended configuration:

```typescript
use: {
    trace: 'on-first-retry'
}

```

Open trace:

```bash
npx playwright show-trace trace.zip

```

Trace Viewer includes:

-   DOM snapshots
    
-   Action timeline
    
-   Network requests
    
-   Console logs
    
-   Screenshots
    

----------

# Debugging with Breakpoints

Using VS Code,

click beside a line number.

```text
↓

Breakpoint

```

Run in Debug mode.

Execution pauses before the selected line.

----------

# Using `page.pause()`

Example

```typescript
await page.pause();

```

Execution stops,

allowing interactive inspection through Playwright Inspector.

Useful while developing new tests.

----------

# Slow Motion

Launch the browser with slow motion.

Example:

```typescript
browserType.launch({
    slowMo: 500
});

```

Every action pauses for 500 milliseconds.

Useful for demonstrations or observing automation behavior.

----------

# Debugging Failed Assertions

Example

```typescript
await expect(page)
.toHaveTitle("Dashboard");

```

If the assertion fails,

Playwright displays:

-   Expected value
    
-   Actual value
    
-   Timeout information
    
-   Stack trace
    

This diagnostic information helps identify the cause quickly.

----------

# Reading Stack Traces

Example

```text
LoginPage.ts:45

↓

click()

↓

Timeout

```

Start investigating from the first relevant application file rather than external library code.

----------

# Common Debugging Workflow

```text
Failure

↓

HTML Report

↓

Trace Viewer

↓

Inspector

↓

Fix

↓

Run Again

```

This sequence resolves most automation issues efficiently.

----------

# Common Mistakes

### ❌ Running the Entire Suite During Development

Execute only the tests you're actively modifying.

----------

### ❌ Ignoring the HTML Report

The report often contains enough information to diagnose failures without rerunning the test.

----------

### ❌ Adding `waitForTimeout()` for Debugging

Prefer:

-   Breakpoints
    
-   `page.pause()`
    
-   Inspector
    
-   Trace Viewer
    

----------

### ❌ Debugging in Headless Mode

When investigating failures,

headed mode is usually more informative.

----------

### ❌ Ignoring Trace Files

Trace Viewer is one of Playwright's strongest debugging tools. Learn to use it effectively.

----------

# Enterprise Best Practices

-   Run individual tests during development and the full suite in CI.
    
-   Use `--grep` and tags to execute targeted subsets of tests.
    
-   Enable traces on retries to balance diagnostics with storage usage.
    
-   Configure screenshots and videos for failed tests only.
    
-   Use UI Mode and Inspector for interactive debugging.
    
-   Review HTML reports before attempting to reproduce failures.
    
-   Avoid masking issues with unnecessary retries or hard waits.
    

----------

# Interview Questions

### Q1. How do you run a single Playwright test?

Use the `--grep` option with the test name, for example:

```bash
npx playwright test --grep "User Login"

```

Alternatively, run it directly from VS Code's Test Explorer.

----------

### Q2. What is the purpose of `--debug`?

`--debug` launches the browser in headed mode, opens Playwright Inspector, slows execution, and enables interactive debugging.

----------

### Q3. What is the difference between UI Mode and Playwright Inspector?

UI Mode is an interactive interface for managing, filtering, and running tests. Inspector is focused on step-by-step debugging of a specific test execution.

----------

### Q4. What information does Trace Viewer provide?

Trace Viewer includes the action timeline, DOM snapshots, screenshots, network activity, console logs, and timing information, allowing engineers to replay test execution.

----------

### Q5. Why is `page.pause()` useful?

`page.pause()` temporarily halts test execution and opens Playwright Inspector, making it easier to inspect the current page state, locators, and application behavior during test development.

----------

# Summary

Running and debugging tests effectively is just as important as writing them. Playwright provides a comprehensive set of execution and debugging tools, including targeted test execution, headed mode, debug mode, UI Mode, Inspector, HTML reports, Trace Viewer, screenshots, videos, and VS Code integration. Mastering these tools enables engineers to diagnose failures quickly, reduce debugging time, and maintain reliable automation suites as projects grow.

----------
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExOTMwMzgzMDVdfQ==
-->